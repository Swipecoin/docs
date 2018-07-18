---
title: Enviar e Receber Dinheiro
---

Agora que você tem uma conta, você pode enviar e receber fundos por meio da rede Stellar. Se não tiver criado uma conta ainda, leia o [passo 2 do guia Para Começar](./create-account.md).

Na maioria das vezes, você estará enviando dinheiro para outra pessoa que tem sua própria conta. Porém, para este guia interativo, é melhor você fazer uma segunda conta com a qual transacionar, pelo mesmo método que usou para fazer sua primeira conta.

## Enviar Pagamentos

Ações que alteram coisas no Stellar, como enviar pagamentos, mudar sua conta, ou fazer ofertas de troca entre vários tipos de moedas, são chamadas **operações**, ou operations.[^1] Para realizar de fato uma operação, cria-se uma **transação** (transaction), que é apenas um grupo de operações acompanhado de algumas informações adicionais, como que conta está fazendo a transação e uma assinatura criptográfica para verificar que a transação é autêntica.[^2]

Se qualquer operação na transação falhar, todas falham. Por exemplo, digamos que você tem 100 lumens e faz duas operações de pagamento de 60 lumens cada. Se fizer duas transações (cada uma com uma operação), a primeira terá sucesso e a segunda irá falhar, pois você não terá lumens suficientes. Restarão 40 lumens. No entanto, se agrupar os dois pagamentos em apenas uma transação, ambos irão falhar e todos os 100 lumens permanecerão na sua conta.

Por último, toda transação custa uma pequena tarifa. Assim como o saldo mínimo nas contas, a tarifa ajuda a impedir que pessoas sobrecarreguem o sistema com um monte de transações. Conhecida como **tarifa base** (base fee), é uma tarifa bem pequena — 100 stroops por operação (igual a 0.00001 XLM; é mais fácil falar em stroops do que em frações de lumen tão minúsculas). Uma transação com duas operações custaria 200 stroops.[^3]

### Construir uma Transação

Stellar armazena e comunica os dados das transações em um formato binário chamado XDR.[^4] Por sorte, os SDKs de Stellar dão ferramentas que cuidam de tudo isso. Aqui está como você poderia enviar 10 lumens para outra conta:

<code-example name="Submeter uma Transação">

```js
var StellarSdk = require('stellar-sdk');
StellarSdk.Network.useTestNetwork();
var server = new StellarSdk.Server('https://horizon-testnet.stellar.org');
var sourceKeys = StellarSdk.Keypair
  .fromSecret('SCZANGBA5YHTNYVVV4C3U252E2B6P6F5T3U6MM63WBSBZATAQI3EBTQ4');
var destinationId = 'GA2C5RFPE6GCKMY3US5PAB6UZLKIGSPIUKSLRB6Q723BM2OARMDUYEJ5';
// A variável transaction irá guardar uma transação pré-construída caso o resultado seja desconhecido.
var transaction;

// Primeiro, checar para ter certeza de que a conta de destino existe.
// Você pode pular isso, mas se a conta não existir, será cobrada
// a tarifa de transação quando a transação falhar.
server.loadAccount(destinationId)
  // Se a conta não for encontrada, subir uma mensagem de erro mais agradável.
  .catch(StellarSdk.NotFoundError, function (error) {
    throw new Error('A conta de destino não existe!');
  })
  // Se não houver erro, carregar informações atualizadas sobre a sua conta.
  .then(function() {
    return server.loadAccount(sourceKeys.publicKey());
  })
  .then(function(sourceAccount) {
    // Começar a construir a transação.
    transaction = new StellarSdk.TransactionBuilder(sourceAccount)
      .addOperation(StellarSdk.Operation.payment({
        destination: destinationId,
        // Como o Stellar permite transações em várias moedas, é preciso
        // especificar o tipo do ativo (asset). O ativo especial "native" representa Lumens.
        asset: StellarSdk.Asset.native(),
        amount: "10"
      }))
      // Um memo permite adicionar seus próprios metadados a uma transação.
      // É opcional e não afeta como o Stellar trata a transação.
      .addMemo(StellarSdk.Memo.text('Transação teste'))
      .build();
    // Assinar a transação para provar que você é realmente a pessoa que está enviando.
    transaction.sign(sourceKeys);
    // E, finalmente, enviar para o Stellar!
    return server.submitTransaction(transaction);
  })
  .then(function(result) {
    console.log('Successo! Resultados:', result);
  })
  .catch(function(error) {
    console.error('Algo deu errado!', error);
    // Se o resultado for desconhecido (sem body da resposta, tempo esgotado, etc.),
    // simplesmente reenviamos a transação já construída:
    // server.submitTransaction(transaction);
  });
```

```java
Network.useTestNetwork();
Server server = new Server("https://horizon-testnet.stellar.org");

KeyPair source = KeyPair.fromSecretSeed("SCZANGBA5YHTNYVVV4C3U252E2B6P6F5T3U6MM63WBSBZATAQI3EBTQ4");
KeyPair destination = KeyPair.fromAccountId("GA2C5RFPE6GCKMY3US5PAB6UZLKIGSPIUKSLRB6Q723BM2OARMDUYEJ5");

// Primeiro, checar para ter certeza de que a conta de destino existe.
// Você pode pular isso, mas se a conta não existir, será cobrada
// a tarifa de transação quando a transação falhar.
// Será lançada uma HttpResponseException se a conta não existir ou se tiver tido outro erro.
server.accounts().account(destination);

// Se não houver erro, carregar informações atualizadas sobre a sua conta.
AccountResponse sourceAccount = server.accounts().account(source);

// Começar a construir a transação.
Transaction transaction = new Transaction.Builder(sourceAccount)
        .addOperation(new PaymentOperation.Builder(destination, new AssetTypeNative(), "10").build())
        // Um memo permite adicionar seus próprios metadados a uma transação.
        // É opcional e não afeta como o Stellar trata a transação.
        .addMemo(Memo.text("Transação Teste"))
        .build();
// Assinar a transação para provar que você é realmente a pessoa que está enviando.
transaction.sign(source);

// E, finalmente, enviar para o Stellar!
try {
  SubmitTransactionResponse response = server.submitTransaction(transaction);
  System.out.println("Successo!");
  System.out.println(response);
} catch (Exception e) {
  System.out.println("Algo deu errado!");
  System.out.println(e.getMessage());
  // Se o resultado for desconhecido (sem body da resposta, tempo esgotado, etc.),
  // simplesmente reenviamos a transação já construída:
  // SubmitTransactionResponse response = server.submitTransaction(transaction);
}
```

```go
package main

import (
	"github.com/stellar/go/build"
    "github.com/stellar/go/clients/horizon"
    "fmt"
)

func main () {
	source := "SCZANGBA5YHTNYVVV4C3U252E2B6P6F5T3U6MM63WBSBZATAQI3EBTQ4"
	destination := "GA2C5RFPE6GCKMY3US5PAB6UZLKIGSPIUKSLRB6Q723BM2OARMDUYEJ5"

	// Verificar que a conta de destino existe
	if _, err := horizon.DefaultTestNetClient.LoadAccount(destination); err != nil {
		panic(err)
	}

	passphrase := network.TestNetworkPassphrase

	tx, err := build.Transaction(
		build.TestNetwork,
		build.SourceAccount{source},
		build.AutoSequence{horizon.DefaultTestNetClient},
		build.Payment(
			build.Destination{destination},
			build.NativeAmount{"10"},
		),
	)

	if err != nil {
		panic(err)
	}

	// Assinar a transação para provar que você é realmente a pessoa que está enviando.
	txe, err := tx.Sign(source)
	if err != nil {
		panic(err)
	}

	txeB64, err := txe.Base64()
	if err != nil {
		panic(err)
	}

	// E, finalmente, enviar para o Stellar!
	resp, err := horizon.DefaultTestNetClient.SubmitTransaction(txeB64)
	if err != nil {
		panic(err)
	}

	fmt.Println("Transação bem-sucedida:")
	fmt.Println("Ledger:", resp.Ledger)
	fmt.Println("Hash:", resp.Hash)
}
```

</code-example>

O que exatamente aconteceu aí? Vamos ver por partes.

1. Confirmar que o ID da conta para a qual está enviando realmente existe, carregando a partir da rede Stellar os dados associados à conta. Nada vai dar errado se pular essa parte, mas fazer isso dá uma oportunidade de evitar fazer uma transação que você sabe que irá falhar. Você também pode usar esta chamada para realizar qualquer outra verificação que possa querer fazer na conta de destino. Se estiver escrevendo um software bancário, por exemplo, esse é um bom lugar para inserir checagens regulatórias de compliance e verificações <abbr title="Know Your Customer">KYC</abbr>.

    <code-example name="Carregar uma conta">

    ```js
    server.loadAccount(destinationId)
      .then(function(account) { /* validar a conta */ })
    ```

    ```java
    server.accounts().account(destination);
    ```

    ```go
	if _, err := horizon.DefaultTestNetClient.LoadAccount(destination); err != nil {
		panic(err)
	}
    ```

    </code-example>

2. Carregar dados da conta a partir da qual você está enviando. Uma conta pode realizar apenas uma transação por vez[^5] e possui algo chamado [**número sequencial**,](../concepts/accounts.md#sequence-number) (sequence number) que ajuda o Stellar a verificar a ordem das transações. O número sequencial de uma transação precisa bater com o número sequencial da conta, sendo assim necessário puxar da rede o número sequencial da conta.

    <code-example name="Carregar a Conta Fonte">

    ```js
    .then(function() {
    return server.loadAccount(sourceKeys.publicKey());
    })
    ```

    ```java
    AccountResponse sourceAccount = server.accounts().account(source);
    ```

    </code-example>

    O SDK vai automaticamente incrementar o número sequencial da conta quando você construir uma transação, então não será preciso puxar essa informação novamente caso queira realizar uma segunda transação.

3. Começar a construir a transação. Isso requer um objeto account (conta), não só um ID de conta, porque ele irá incrementar o número sequencial da conta.

    <code-example name="Construir uma Transação">

    ```js
    var transaction = new StellarSdk.TransactionBuilder(sourceAccount)
    ```

    ```java
    Transaction transaction = new Transaction.Builder(sourceAccount)
    ```

    ```go
	tx, err := build.Transaction(
	// ...
	)
    ```

    </code-example>

4. Adicionar a operação de pagamento (payment) à conta. Note que é preciso especificar o tipo de ativo que está sendo enviado — a moeda "native" do Stellar é o lumen, mas você pode enviar qualquer tipo de ativo ou moeda que quiser, de dólares a bitcoin ou qualquer outro tipo de ativo que confiar que o emissor pode liquidar [(mais detalhes abaixo)](#transacting-in-other-currencies). Por enquanto, vamos ficar só com lumens, que são chamados de ativos "native" pelo SDK:

    <code-example name="Adicionar uma Operação">

    ```js
    .addOperation(StellarSdk.Operation.payment({
      destination: destinationId,
      asset: StellarSdk.Asset.native(),
      amount: "10"
    }))
    ```

    ```java
    .addOperation(new PaymentOperation.Builder(destination, new AssetTypeNative(), "10").build())
    ```

    ```go
    tx, err := build.Transaction(
		build.Network{passphrase},
		build.SourceAccount{from},
		build.AutoSequence{horizon.DefaultTestNetClient},
		build.MemoText{"Transação Teste"},
		build.Payment(
			build.Destination{to},
			build.NativeAmount{"10"},
		),
	)
    ```

    </code-example>

    Note também que o valor é uma string em vez de um número. Quando se trabalha com frações extremamente pequenas ou valores altos, [cálculos baseados em ponto flutuante podem introduzir pequenas imprecisões](https://pt.wikipedia.org/wiki/V%C3%ADrgula_flutuante#Problemas_com_o_uso_de_ponto_flutuante). Já que nem todos os sistemas têm uma maneira nativa de representar com precisão decimais extremamente pequenos ou grandes, Stellar usa strings como uma maneira confiável de representar o valor exato em qualquer sistema.

5. Opcionalmente, você pode adicionar seus próprios metadados, chamados [**memo,**](../concepts/transactions.md#memo) a uma transação. O Stellar não faz nada com esses dados, mas você pode usá-los para qualquer fim que quiser. Se você é um banco que está recebendo ou enviando pagamentos em nome de outras pessoas, por exemplo, você poderia incluir aqui informações sobre a pessoa à qual se destina o pagamento.

    <code-example name="Adicionar um Memo">

    ```js
    .addMemo(StellarSdk.Memo.text('Transação Teste'))
    ```

    ```java
    .addMemo(Memo.text("Transação Teste"));
    ```

    ```go
    build.MemoText{"Transação Teste"},
    ```

    </code-example>

6. Agora que a transação tem todos os dados que precisa, você tem que assiná-la criptograficamente usando sua seed secreta. Isso prova que os dados realmente vieram de você e não de alguém fingindo ser você.

    <code-example name="Assinar a Transação">

    ```js
    transaction.sign(sourceKeys);
    ```

    ```java
    transaction.sign(source);
    ```

    ```go
    txe, err := tx.Sign(from)
    ```

    </code-example>

7. E finalmente, enviá-la à rede Stellar!

    <code-example name="Submeter a Transação">

    ```js
    server.submitTransaction(transaction);
    ```

    ```java
    server.submitTransaction(transaction);
    ```

    ```go
    resp, err := horizon.DefaultTestNetClient.SubmitTransaction(txeB64)
    ```

    </code-example>

**IMPORTANTE** É possível que você não receba uma resposta do servidor Horizon devido a um bug, condições de conexão, etc. Nessas situações é impossível determinar o status da sua transação. É por isso que se recomenda sempre salvar uma transação construída (ou uma transação codificada em formato XDR) em uma variável ou base de dados e reenviá-la caso não souber seu status. Se a transação já tiver sido aplicada ao ledger com sucesso, o Horizon irá simplesmente retornar o resultado salvo e não irá tentar submeter a transação novamente. Somente em casos em que o status de uma transação é desconhecido (e assim terá uma chance de ser incluída em um ledger) é que ocorrerá um reenvio à rede.

## Receive Payments

You don’t actually need to do anything to receive payments into a Stellar account—if a payer makes a successful transaction to send assets to you, those assets will automatically be added to your account.

However, you’ll want to know that someone has actually paid you. If you are a bank accepting payments on behalf of others, you need to find out what was sent to you so you can disburse funds to the intended recipient. If you are operating a retail business, you need to know that your customer actually paid you before you hand them their merchandise. And if you are an automated rental car with a Stellar account, you’ll probably want to verify that the customer in your front seat actually paid before that person can turn on your engine.

A simple program that watches the network for payments and prints each one might look like:

<code-example name="Receive Payments">

```js
var StellarSdk = require('stellar-sdk');

var server = new StellarSdk.Server('https://horizon-testnet.stellar.org');
var accountId = 'GC2BKLYOOYPDEFJKLKY6FNNRQMGFLVHJKQRGNSSRRGSMPGF32LHCQVGF';

// Create an API call to query payments involving the account.
var payments = server.payments().forAccount(accountId);

// If some payments have already been handled, start the results from the
// last seen payment. (See below in `handlePayment` where it gets saved.)
var lastToken = loadLastPagingToken();
if (lastToken) {
  payments.cursor(lastToken);
}

// `stream` will send each recorded payment, one by one, then keep the
// connection open and continue to send you new payments as they occur.
payments.stream({
  onmessage: function(payment) {
    // Record the paging token so we can start from here next time.
    savePagingToken(payment.paging_token);

    // The payments stream includes both sent and received payments. We only
    // want to process received payments here.
    if (payment.to !== accountId) {
      return;
    }

    // In Stellar’s API, Lumens are referred to as the “native” type. Other
    // asset types have more detailed information.
    var asset;
    if (payment.asset_type === 'native') {
      asset = 'lumens';
    }
    else {
      asset = payment.asset_code + ':' + payment.asset_issuer;
    }

    console.log(payment.amount + ' ' + asset + ' from ' + payment.from);
  },

  onerror: function(error) {
    console.error('Error in payment stream');
  }
});

function savePagingToken(token) {
  // In most cases, you should save this to a local database or file so that
  // you can load it next time you stream new payments.
}

function loadLastPagingToken() {
  // Get the last paging token from a local database or file
}
```

```java
Server server = new Server("https://horizon-testnet.stellar.org");
KeyPair account = KeyPair.fromAccountId("GC2BKLYOOYPDEFJKLKY6FNNRQMGFLVHJKQRGNSSRRGSMPGF32LHCQVGF");

// Create an API call to query payments involving the account.
PaymentsRequestBuilder paymentsRequest = server.payments().forAccount(account);

// If some payments have already been handled, start the results from the
// last seen payment. (See below in `handlePayment` where it gets saved.)
String lastToken = loadLastPagingToken();
if (lastToken != null) {
  paymentsRequest.cursor(lastToken);
}

// `stream` will send each recorded payment, one by one, then keep the
// connection open and continue to send you new payments as they occur.
paymentsRequest.stream(new EventListener<OperationResponse>() {
  @Override
  public void onEvent(OperationResponse payment) {
    // Record the paging token so we can start from here next time.
    savePagingToken(payment.getPagingToken());

    // The payments stream includes both sent and received payments. We only
    // want to process received payments here.
    if (payment instanceof PaymentOperationResponse) {
      if (((PaymentOperationResponse) payment).getTo().equals(account)) {
        return;
      }

      String amount = ((PaymentOperationResponse) payment).getAmount();

      Asset asset = ((PaymentOperationResponse) payment).getAsset();
      String assetName;
      if (asset.equals(new AssetTypeNative())) {
        assetName = "lumens";
      } else {
        StringBuilder assetNameBuilder = new StringBuilder();
        assetNameBuilder.append(((AssetTypeCreditAlphaNum) asset).getCode());
        assetNameBuilder.append(":");
        assetNameBuilder.append(((AssetTypeCreditAlphaNum) asset).getIssuer().getAccountId());
        assetName = assetNameBuilder.toString();
      }

      StringBuilder output = new StringBuilder();
      output.append(amount);
      output.append(" ");
      output.append(assetName);
      output.append(" from ");
      output.append(((PaymentOperationResponse) payment).getFrom().getAccountId());
      System.out.println(output.toString());
    }

  }
});
````

```go
package main

import (
	"context"
	"fmt"
	"github.com/stellar/go/clients/horizon"
)

func main() {
	const address = "GC2BKLYOOYPDEFJKLKY6FNNRQMGFLVHJKQRGNSSRRGSMPGF32LHCQVGF"
	ctx := context.Background()

	cursor := horizon.Cursor("now")

	fmt.Println("Waiting for a payment...")

	err := horizon.DefaultTestNetClient.StreamPayments(ctx, address, &cursor, func(payment horizon.Payment) {
		fmt.Println("Payment type", payment.Type)
		fmt.Println("Payment Paging Token", payment.PagingToken)
		fmt.Println("Payment From", payment.From)
		fmt.Println("Payment To", payment.To)
		fmt.Println("Payment Asset Type", payment.AssetType)
		fmt.Println("Payment Asset Code", payment.AssetCode)
		fmt.Println("Payment Asset Issuer", payment.AssetIssuer)
		fmt.Println("Payment Amount", payment.Amount)
		fmt.Println("Payment Memo Type", payment.Memo.Type)
		fmt.Println("Payment Memo", payment.Memo.Value)
	})

	if err != nil {
		panic(err)
	}

}
```

</code-example>

There are two main parts to this program. First, you create a query for payments involving a given account. Like most queries in Stellar, this could return a huge number of items, so the API returns paging tokens, which you can use later to start your query from the same point where you previously left off. In the example above, the functions to save and load paging tokens are left blank, but in a real application, you’d want to save the paging tokens to a file or database so you can pick up where you left off in case the program crashes or the user closes it.

<code-example name="Create a Payments Query">

```js
var payments = server.payments().forAccount(accountId);
var lastToken = loadLastPagingToken();
if (lastToken) {
  payments.cursor(lastToken);
}
```

```java
PaymentsRequestBuilder paymentsRequest = server.payments().forAccount(account)
String lastToken = loadLastPagingToken();
if (lastToken != null) {
  paymentsRequest.cursor(lastToken);
}
```

</code-example>

Second, the results of the query are streamed. This is the easiest way to watch for payments or other transactions. Each existing payment is sent through the stream, one by one. Once all existing payments have been sent, the stream stays open and new payments are sent as they are made.

Try it out: Run this program, and then, in another window, create and submit a payment. You should see this program log the payment.

<code-example name="Stream Payments">

```js
payments.stream({
  onmessage: function(payment) {
    // handle a payment
  }
});
```

```java
paymentsRequest.stream(new EventListener<OperationResponse>() {
  @Override
  public void onEvent(OperationResponse payment) {
    // Handle a payment
  }
});
```

</code-example>

You can also request payments in groups, or pages. Once you’ve processed each page of payments, you’ll need to request the next one until there are none left.

<code-example name="Paged Payments">

```js
payments.call().then(function handlePage(paymentsPage) {
  paymentsPage.records.forEach(function(payment) {
    // handle a payment
  });
  return paymentsPage.next().then(handlePage);
});
```

```java
Page<OperationResponse> page = payments.execute();

for (OperationResponse operation : page.getRecords()) {
	// handle a payment
}

page = page.getNextPage();
```

</code-example>


## Transacting in Other Currencies

One of the amazing things about the Stellar network is that you can send and receive many types of assets, such as US dollars, Nigerian naira, digital currencies like bitcoin, or even your own new kind of asset.

While Stellar’s native asset, the lumen, is fairly simple, all other assets can be thought of like a credit issued by a particular account. In fact, when you trade US dollars on the Stellar network, you don’t actually trade US dollars—you trade US dollars *from a particular account.* That’s why the assets in the example above had both a `code` and an `issuer`. The `issuer` is the ID of the account that created the asset. Understanding what account issued the asset is important—you need to trust that, if you want to redeem your dollars on the Stellar network for actual dollar bills, the issuer will be able to provide them to you. Because of this, you’ll usually only want to trust major financial institutions for assets that represent national currencies.

Stellar also supports payments sent as one type of asset and received as another. You can send Nigerian naira to a friend in Germany and have them receive euros. These multi-currency transactions are made possible by a built-in market mechanism where people can make offers to buy and sell different types of assets. Stellar will automatically find the best people to exchange currencies with in order to convert your naira to euros. This system is called [distributed exchange](../concepts/exchange.md).

You can read more about the details of assets in the [assets overview](../concepts/assets.md).

## What Next?

Now that you can send and receive payments using Stellar’s API, you’re on your way to writing all kinds of amazing financial software. Experiment with other parts of the API, then read up on more detailed topics:

- [Become an anchor](../anchor/)
- [Security](../security.md)
- [Federation](../concepts/federation.md)
- [Compliance](../compliance-protocol.md)

<div class="sequence-navigation">
  <a class="button button--previous" href="create-account.html">Back to Step 2: Create an Account</a>
</div>


[^1]:  Uma lista de todas as operações possíveis pode ser encontrada na [página de operações](../concepts/operations.md).

[^2]: Os detalhes completos sobre transações podem ser encontrados na [página de transações](../concepts/transactions.md).

[^3]: Os 100 stroops são a **tarifa base** do Stellar. A tarifa base pode ser mudada, mas é improvável que mudanças nas tarifas do Stellar aconteçam mais do que uma vez a cada vários anos. Você pode descobrir as tarifas atuais [checando os detalhes do último ledger](https://www.stellar.org/developers/horizon/reference/endpoints/ledgers-single.html).

[^4]: Embora a maioria das respostas da API REST do Horizon use JSON, a maior parte dos dados no Stellar na verdade é armazenada em um formato chamado XDR, ou External Data Representation. XDR não só é mais compacto do que JSON, como também armazena dados de uma maneira previsível, o que torna mais fácil assinar e verificar uma mensagem codificada em XDR. Pegue mais detalhes na [nossa página sobre XDR](https://www.stellar.org/developers/horizon/reference/xdr.html).

[^5]: Em situações em que é preciso realizar um número alto de transações em um curto período de tempo (por exemplo, um banco pode realizar transações em nome de vários clientes usando uma conta Stellar), você pode criar várias contas Stellar que trabalham simultaneamente. Leia mais sobre isso no [guia para canais](../channels.md).
