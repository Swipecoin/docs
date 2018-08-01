---
title: Adicione Stellar a sua Exchange
---

Este guia irá indicar os passos de integração para adicionar Stellar a sua exchange. Este exemplo usa Node.js e o [SDK de Stellar para JS](https://github.com/stellar/js-stellar-sdk), mas deve ser fácil adaptá-los a outras linguagens.

Há várias maneiras de arquitetar uma exchange. Este guia usa o seguinte design:
 - `issuing account`: Conta emissora. Uma conta Stellar que detém offline a maioria dos depósitos dos clientes.
 - `base account`: Conta base. Uma conta Stellar que detém online uma pequena quantidade de depósitos dos clientes e é usada para custear pedidos de saque.
 - `customerID`: ID do cliente. Cada usuário possui um customerID, usado para correlacionar envios de depósito e a conta de um usuário específico na exchange.

Os dois pontos principais de integração ao Stellar para uma exchange são:<br>
1) Escutar transações de depósito a partir da rede Stellar<br>
2) Submeter transações de saque à rede Stellar

## Configuração

### Operacional
* *(opcional)* Configurar [Stellar Core](https://www.stellar.org/developers/stellar-core/software/admin.html)
* *(opcional)* Configurar [Horizon](https://www.stellar.org/developers/horizon/reference/index.html)

Se sua exchange não tiver que lidar com muito volume, não é preciso montar suas próprias instâncias de Stellar Core e Horizon. Ao invés disso, use um dos servidores públicos de Horizon do Stellar.org.
```
  test net: {hostname:'horizon-testnet.stellar.org', secure:true, port:443};
  live: {hostname:'horizon.stellar.org', secure:true, port:443};
```

### Conta emissora
Uma conta emissora é tipicamente usada para guardar com segurança a maior parte dos fundos dos clientes. Uma conta emissora é uma conta Stellar cujas chaves secretas não estão em nenhum dispositivo que tenha contato com a Internet. transações são iniciadas manualmente por um humano e assinadas localmente na máquina offline — uma instalação local de `js-stellar-sdk` cria uma `tx_blob` que contém a transação assinada. Esta `tx_blob` pode ser transportada para uma máquina conectada à Internet por métodos offline (ex.: USB ou manualmente). Esta operação torna a chave secreta da conta emissora muito mais difícil de ser comprometida.

### Conta base
Uma conta base contém um número mais limitado de fundos que uma conta emissora. Uma conta base é uma conta Stellar usada em uma máquina que está conectada à Internet. Ela realiza operações cotidianas de envio e recebimento de lumens. O número limitado de fundos numa conta base evita maiores perdas em caso de uma quebra de segurança.

### Database
- Need to create a table for pending withdrawals, `StellarTransactions`.
- Need to create a table to hold the latest cursor position of the deposit stream, `StellarCursor`.
- Need to add a row to your users table that creates a unique `customerID` for each user.
- Need to populate the customerID row.

```
CREATE TABLE StellarTransactions (UserID INT, Destination varchar(56), XLMAmount INT, state varchar(8));
CREATE TABLE StellarCursor (id INT, cursor varchar(50)); // id - AUTO_INCREMENT field
```

Possible values for `StellarTransactions.state` are "pending", "done", "error".


### Code

Here is a code framework you can use to integrate Stellar into your exchange. The following sections describe each step.

For this guide, we use placeholder functions for reading/writing to the exchange database. Each database library connects differently, so we abstract away those details.

```js
// Config your server
var config = {};
config.baseAccount = "your base account address";
config.baseAccountSecret = "your base account secret key";

// You can use Stellar.org's instance of Horizon or your own
config.horizon = 'https://horizon-testnet.stellar.org';

// Include the JS Stellar SDK
// It provides a client-side interface to Horizon
var StellarSdk = require('stellar-sdk');
// uncomment for live network:
// StellarSdk.Network.usePublicNetwork();

// Initialize the Stellar SDK with the Horizon instance
// You want to connect to
var server = new StellarSdk.Server(config.horizon);

// Get the latest cursor position
var lastToken = latestFromDB("StellarCursor");

// Listen for payments from where you last stopped
// GET https://horizon-testnet.stellar.org/accounts/{config.baseAccount}/payments?cursor={last_token}
let callBuilder = server.payments().forAccount(config.baseAccount);

// If no cursor has been saved yet, don't add cursor parameter
if (lastToken) {
  callBuilder.cursor(lastToken);
}

callBuilder.stream({onmessage: handlePaymentResponse});

// Load the account sequence number from Horizon and return the account
// GET https://horizon-testnet.stellar.org/accounts/{config.baseAccount}
server.loadAccount(config.baseAccount)
  .then(function (account) {
    submitPendingTransactions(account);
  })
```

## Listening for deposits
When a user wants to deposit lumens in your exchange, instruct them to send XLM to your base account address with the customerID in the memo field of the transaction.

You must listen for payments to the base account and credit any user that sends XLM there. Here's code that listens for these payments:

```js
// Start listening for payments from where you last stopped
var lastToken = latestFromDB("StellarCursor");

// GET https://horizon-testnet.stellar.org/accounts/{config.baseAccount}/payments?cursor={last_token}
let callBuilder = server.payments().forAccount(config.baseAccount);

// If no cursor has been saved yet, don't add cursor parameter
if (lastToken) {
  callBuilder.cursor(lastToken);
}

callBuilder.stream({onmessage: handlePaymentResponse});
```


For every payment received by the base account, you must:<br>
-check the memo field to determine which user sent the deposit.<br>
-record the cursor in the `StellarCursor` table so you can resume payment processing where you left off.<br>
-credit the user's account in the DB with the number of XLM they sent to deposit.

So, you pass this function as the `onmessage` option when you stream payments:

```js
function handlePaymentResponse(record) {

  // GET https://horizon-testnet.stellar.org/transaction/{id of transaction this payment is part of}
  record.transaction()
    .then(function(txn) {
      var customer = txn.memo;

      // If this isn't a payment to the baseAccount, skip
      if (record.to != config.baseAccount) {
        return;
      }
      if (record.asset_type != 'native') {
         // If you are a XLM exchange and the customer sends
         // you a non-native asset, some options for handling it are
         // 1. Trade the asset to native and credit that amount
         // 2. Send it back to the customer  
      } else {
        // Credit the customer in the memo field
        if (checkExists(customer, "ExchangeUsers")) {
          // Update in an atomic transaction
          db.transaction(function() {
            // Store the amount the customer has paid you in your database
            store([record.amount, customer], "StellarDeposits");
            // Store the cursor in your database
            store(record.paging_token, "StellarCursor");
          });
        } else {
          // If customer cannot be found, you can raise an error,
          // add them to your customers list and credit them,
          // or do anything else appropriate to your needs
          console.log(customer);
        }
      }
    })
    .catch(function(err) {
      // Process error
    });
}
```


## Submitting withdrawals
When a user requests a lumen withdrawal from your exchange, you must generate a Stellar transaction to send them the lumens. Here is additional documentation about [Building Transactions](https://www.stellar.org/developers/js-stellar-base/learn/building-transactions.html).

The function `handleRequestWithdrawal` will queue up a transaction in the exchange's `StellarTransactions` table whenever a withdrawal is requested.

```js
function handleRequestWithdrawal(userID,amountLumens,destinationAddress) {
  // Update in an atomic transaction
  db.transaction(function() {
    // Read the user's balance from the exchange's database
    var userBalance = getBalance('userID');

    // Check that user has the required lumens
    if (amountLumens <= userBalance) {
      // Debit the user's internal lumen balance by the amount of lumens they are withdrawing
      store([userID, userBalance - amountLumens], "UserBalances");
      // Save the transaction information in the StellarTransactions table
      store([userID, destinationAddress, amountLumens, "pending"], "StellarTransactions");
    } else {
      // If the user doesn't have enough XLM, you can alert them
    }
  });
}
```

Then, you should run `submitPendingTransactions`, which will check `StellarTransactions` for pending transactions and submit them.

```js
StellarSdk.Network.useTestNetwork();
// This is the function that handles submitting a single transaction

function submitTransaction(exchangeAccount, destinationAddress, amountLumens) {
  // Update transaction state to sending so it won't be
  // resubmitted in case of the failure.
  updateRecord('sending', "StellarTransactions");

  // Check to see if the destination address exists
  // GET https://horizon-testnet.stellar.org/accounts/{destinationAddress}
  server.loadAccount(destinationAddress)
    // If so, continue by submitting a transaction to the destination
    .then(function(account) {
      var transaction = new StellarSdk.TransactionBuilder(exchangeAccount)
        .addOperation(StellarSdk.Operation.payment({
          destination: destinationAddress,
          asset: StellarSdk.Asset.native(),
          amount: amountLumens
        }))
        // Sign the transaction
        .build();

      transaction.sign(StellarSdk.Keypair.fromSecret(config.baseAccountSecret));

      // POST https://horizon-testnet.stellar.org/transactions
      return server.submitTransaction(transaction);
    })
    //But if the destination doesn't exist...
    .catch(StellarSdk.NotFoundError, function(err) {
      // create the account and fund it
      var transaction = new StellarSdk.TransactionBuilder(exchangeAccount)
        .addOperation(StellarSdk.Operation.createAccount({
          destination: destinationAddress,
          // Creating an account requires funding it with XLM
          startingBalance: amountLumens
        }))
        .build();

      transaction.sign(StellarSdk.Keypair.fromSecret(config.baseAccountSecret));

      // POST https://horizon-testnet.stellar.org/transactions
      return server.submitTransaction(transaction);
    })
    // Submit the transaction created in either case
    .then(function(transactionResult) {
      updateRecord('done', "StellarTransactions");
    })
    .catch(function(err) {
      // Catch errors, most likely with the network or your transaction
      updateRecord('error', "StellarTransactions");
    });
}

// This function handles submitting all pending transactions, and calls the previous one
// This function should be run in the background continuously

function submitPendingTransactions(exchangeAccount) {
  // See what transactions in the db are still pending
  // Update in an atomic transaction
  db.transaction(function() {
    var pendingTransactions = querySQL("SELECT * FROM StellarTransactions WHERE state =`pending`");

    while (pendingTransactions.length > 0) {
      var txn = pendingTransactions.pop();

      // This function is async so it won't block. For simplicity we're using
      // ES7 `await` keyword but you should create a "promise waterfall" so
      // `setTimeout` line below is executed after all transactions are submitted.
      // If you won't do it will be possible to send a transaction twice or more.
      await submitTransaction(exchangeAccount, tx.destinationAddress, tx.amountLumens);
    }

    // Wait 30 seconds and process next batch of transactions.
    setTimeout(function() {
      submitPendingTransactions(sourceAccount);
    }, 30*1000);
  });
}
```

## Going further...
### Federation
The federation protocol allows you to give your users easy addresses—e.g., bob*yourexchange.com—rather than cumbersome raw addresses such as: GCEZWKCA5VLDNRLN3RPRJMRZOX3Z6G5CHCGSNFHEYVXM3XOJMDS674JZ?19327

For more information, check out the [federation guide](./concepts/federation.md).

### Anchor
If you're an exchange, it's easy to become a Stellar anchor as well. The integration points are very similar, with the same level of difficulty. Becoming a anchor could potentially expand your business.

To learn more about what it means to be an anchor, see the [anchor guide](./anchor/index.html).

### Accepting non-native assets
First, open a [trustline](https://www.stellar.org/developers/guides/concepts/assets.html#trustlines) with the issuing account of the non-native asset -- without this you cannot begin to accept this asset.

```js
var someAsset = new StellarSdk.Asset('ASSET_CODE', issuingKeys.publicKey());

transaction.addOperation(StellarSdk.Operation.changeTrust({
        asset: someAsset
}))
```
If the asset issuer has `authorization_required` set to true, you will need to wait for the trustline to be authorized before you can begin accepting this asset. Read more about [trustline authorization here](https://www.stellar.org/developers/guides/concepts/assets.html#controlling-asset-holders).

Then, make a few small changes to the example code above:
* In the `handlePaymentResponse` function, we dealt with the case of incoming non-native assets. Since we are now accepting non-native assets, you will need to change this condition; if the user sends us lumens  we will either:
	1. Trade lumens for the desired non-native asset
	2. Send the lumens back to the sender

*Note*: the user cannot send us non-native assets whose issuing account we have not explicitly opened a trustline with.

* In the `withdraw` function, when we add an operation to the transaction, we must specify the details of the asset we are sending. For example:
```js
var someAsset = new StellarSdk.Asset('ASSET_CODE', issuingKeys.publicKey());

transaction.addOperation(StellarSdk.Operation.payment({
        destination: receivingKeys.publicKey(),
        asset: someAsset,
        amount: '10'
      }))
```
* Also in the `withdraw` function, note that your customer must have opened a trustline with the issuing account of the asset they are withdrawing. So you must take the following into consideration:
	* Confirm the user you are sending the asset to has a trustline
	* Parse the [Horizon error](https://www.stellar.org/developers/guides/concepts/list-of-operations.html#payment) that will occur after sending an asset to an account without a trustline


For more information about assets check out the [general asset guide](https://www.stellar.org/developers/guides/concepts/assets.html) and the [issuing asset guide](https://www.stellar.org/developers/guides/issuing-assets.html).
