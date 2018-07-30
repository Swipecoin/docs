---
title: Ledger
---

Um **ledger** representa o estado do universo Stellar em dado momento. Um ledger contém a lista de todas as contas e saldos, todas as ordens na exchange distribuída, e qualquer outro dado que persista.

O primeiro ledger na história da rede é chamado de ledger gênesis.

A cada rodada do [Protocolo de Consenso Stellar (SCP)](https://www.stellar.org/developers/learn/concepts/scp.html), a rede chega a um consenso sobre que [conjunto de transações](./transactions.md#conjuntos-de-transações) aplicar ao último ledger fechado; quando um novo conjunto é aplicado, é definido um novo "último ledger fechado".

Cada ledger está criptograficamente conectado a um ledger anterior único, criando uma corrente histórica de ledgers que chegam até o ledger gênesis.

Definimos o número sequencial de um ledger recursivamente:
* Ledger gênesis possui número sequencial 1
* O ledger que segue diretamente um ledger com número sequencial n possui número sequencial igual a n+1

## Ledger header
Every ledger has a **ledger header**. This header has references to the actual data within the ledger as well as a reference to the previous ledger. References here are cryptographic hashes of the content being referenced--the hashes behave like pointers in typical data structures but with added security guarantees.

You can think of the historical ledger chain as a linked list of ledger headers:

[Genesis] <---- [LedgerHeader_1] <----- ... <---- [LedgerHeader_n]

See the protocol file for the object definitions.
[`src/xdr/Stellar-ledger.x`](https://github.com/stellar/stellar-core/blob/master/src/xdr/Stellar-ledger.x)

Every ledger header has the following fields:

- **Version**: Protocol version of this ledger.

- **Previous Ledger Hash**: Hash of the previous ledger. Forms a chain of ledgers stretching back to the genesis ledger.

- **SCP value**: During consensus all the nodes in the network run SCP and come to agreement about a particular value. This value is stored here and in the next three fields (transaction set hash, close time, and upgrades).

  - **Transaction set hash**: Hash of the transaction set that was applied to the previous ledger.

  - **Close time**: When the network closed this ledger. UNIX timestamp.

  - **Upgrades**: How the network adjusts the [base fee](./fees.md) and moves to a new protocol version. This field is usually empty. For more info, see [versioning](./versioning.md).

- **Transaction set result hash**: Hash of the results of applying the transaction set. This data is not, strictly speaking, necessary for validating the results of the transactions. However, this data makes it easier for entities to validate the result of a given transaction without having to apply the transaction set to the previous ledger.

- **Bucket list hash**: Hash of all the objects in this ledger. The data structure that contains all the objects is called the [bucket list](https://github.com/stellar/stellar-core/tree/master/src/bucket).

- **Ledger sequence**: The sequence number of this ledger.

- **Total coins**: Total number of lumens in existence.

- **Fee pool**: Number of lumens that have been paid in fees. This number will be added to the inflation pool and reset to 0 the next time inflation runs. Note this is denominated in lumens, even though a transaction’s [`fee`](./transactions.md#fee) field is in stroops.

- **Inflation sequence**: Number of times inflation has been run.

- **ID pool**: The last used global ID. These IDs are used for generating objects.

- **Maximum Number of Transactions**: The maximum number of [transactions](./transactions.md) the validators have agreed to process in a given ledger. If more transactions are submitted than this number, the validators will include those with the highest fees.

- **Base fee**: The [fee](./fees.md#transaction-fee) the network charges per [operation](./operations.md) in a [transaction](./transactions.md). This field is in stroops, which are 1/10,000,000th of a lumen.

- **Base reserve**: The [reserve](./fees.md#minimum-account-balance) the network uses when calculating an account's minimum balance.

- **Skip list**: Hashes of ledgers in the past. Allows you to jump back in time in the ledger chain without walking back ledger by ledger. There are 4 ledger hashes stored in the skip list. Each slot contains the oldest ledger that is mod of either 50  5000  50000 or 500000 depending on index skipList[0] mod(50), skipList[1] mod(5000), etc.



# Ledger Entries

The ledger is a collection of **entries**. Currently there are 4 types of ledger entries. They're specified in
[`src/xdr/Stellar-ledger-entries.x`](https://github.com/stellar/stellar-core/blob/master/src/xdr/Stellar-ledger-entries.x).

## Account entry
This entry represents an [account](./accounts.md). In Stellar, everything is built around accounts: transactions are performed by accounts, and accounts control the access rights to balances.

Other entries are add-ons, owned by a main account entry. With every new entry
attached to the account, the minimum balance in XLM goes up for the
account. For details, see [fees and minimum balance](./fees.md#minimum-account-balance).

## Trustline entry
[Trustlines](./assets.md) are lines of credit the account has given a particular issuer in a specific currency.

Trustline entries define the rules around the use of this currency. Rules can be defined by the user--e.g., setting a balance limit to limit risk--or by the issuer--e.g., an authorized flag.

## Offer entry
Offers are entries that an account creates in the orderbook. They are a way to automate simple trading inside the Stellar network. For more on offers, refer to the [distributed exchange documentation](exchange.md).

## Data entry
Data entries are key value pairs attached to an account. They allow account controllers to attach arbitrary data to their account. It provides a flexible extension point to add application specific data into the ledger.
