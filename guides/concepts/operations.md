---
title: Operações
---

[Transações](./transactions.md) são compostas de uma [lista de operações](./list-of-operations.md). Cada
operação (operation) é um comando individual que altera (mutate) o ledger.

Aqui estão os tipos possíves de operações:
- [Create Account](./list-of-operations.md#create-account)
- [Payment](./list-of-operations.md#payment)
- [Path Payment](./list-of-operations.md#path-payment)
- [Manage Offer](./list-of-operations.md#manage-offer)
- [Create Passive Offer](./list-of-operations.md#create-passive-offer)
- [Set Options](./list-of-operations.md#set-options)
- [Change Trust](./list-of-operations.md#change-trust)
- [Allow Trust](./list-of-operations.md#allow-trust)
- [Account Merge](./list-of-operations.md#account-merge)
- [Inflation](./list-of-operations.md#inflation)
- [Manage Data](./list-of-operations.md#manage-data)
- [Bump Sequence](./list-of-operations.md#bump-sequence)

Operações são executadas em nome da conta fonte especificada na
transação, a não ser que haja algo definido na operação que ignore isso.

## Thresholds

Cada operação pertence a uma categoria de threshold (limiar) específica: baixa, média ou alta.
Thresholds definem o nível de privilégio que uma operação precisa para obter sucesso.

* Low Security (baixa segurança):
  * AllowTrustTx
    * Usado para permitir que outros signatários (signers) autorizem pessoas a deter crédito desta conta mas não emitir crédito.
  * BumpSequence
* Medium Security (segurança média):
  * Todo o resto
* High Security (alta segurança):
  * AccountMerge
    * Fundir uma conta com outra.
  * SetOptions para Signer e threshold
    * Usado para alterar o Set de signers e os thresholds.


## Validade de uma operação

Há dois lugares no [ciclo de vida de uma transação](./transactions.md#life-cycle) onde operações podem falhar. O primeiro é quando uma transação é submetida à rede. O nó para onde a transação é enviada verifica a validade da operação: no **validity check** (verificação de validade), o nó realiza algumas verificações superficiais para conferir que a transação está formada adequadamente antes de incluí-la em seu set de transações e enviá-la ao resto da rede.

A verificação de validade somente olha para o estado da conta fonte, assegurando que:

1) A transação saindo tem assinaturas suficientes para a conta fonte da operação atingir o threshold daquela operação.
2) Sejam aprovadas as verificações de validade específicas a operações. Essas verificações são aquelas que permaneceriam verdadeiras indepentendemente do estado do ledger — por exemplo, os parâmetros estão dentro dos limites esperados? Verificações que dependem do estado do ledger não acontecem até o momento da aplicação — por exemplo, uma operação de envio não irá verificar se há saldo suficiente para enviar até o momento da aplicação.

Depois de uma transação passar essa primeira validação, ela é propagada à rede e incluída em um set de transações em algum momento. Fazendo parte de um set de transações, a transação é aplicada ao ledger. Nesse momento uma tarifa é retirada da conta fonte independentemente de sucesso/falha. Depois, a transação é processada: número sequencial e assinaturas são verificados antes da tentativa de efetuar as operações na ordem que elas ocorrem na transação. Se alguma operação falhar, toda a transação falha e os efeitos das operações anteriores são anulados.


## Result

For each operation, there is a matching result type. In the case of success, this result allows users to gather information about the effects of the operation. In the case of failure, it allows users to learn more about the error.

Stellar Core queues results in the txhistory table for other components to derive data from. This txhistory table is used by the history module in Stellar Core for uploading the history into long-term storage. It can also be used by external processes such as Horizon to gather the network history they need.

## Transactions involving multiple accounts

Typically transactions only involve operations on a single account. For example, if account A wanted to send lumens to account B, only account A needs to authorize the transaction.

It's possible, however, to compose a transaction that includes operations on multiple accounts. In this case, to authorize the operations, the transaction envelope must include signatures of every account in question. For example, you can make a transaction where accounts A and B both send to account C. This transaction would need authorization from both account A and B before it's submitted to the network.


## Examples
### 1. Exchange without third party

  Anush wants to send Bridget some XLM (Operation 1) in exchange for BTC (Operation 2).

  A transaction is constructed:
  * source = `Anush_account`
  * Operation 1
    * source = _null_
    * Payment send XLM --> `Bridget_account`
  * Operation 2
    * source = _`Bridget_account`
    * Payment send BTC --> `Anush_account`

   Signatures required:
  * Operation 1: requires signatures from `Anush_account` (the operation inherits
    the source account from the transaction) to meet medium threshold
  * Operation 2: requires signatures for `Bridget_account` to meet medium threshold
  * The transaction requires signatures for `Anush_account` to meet low threshold since `Anush_account` is the
    source for the entire transaction.

Therefore, if both `Anush_account` and `Bridget_account` sign the transaction, it will be validated.  
Other, more complex ways of submitting this transaction are possible, but signing with those two accounts is sufficient.

### 2. Workers

   An anchor wants to divide the processing of their online ("base") account between machines. That way, each machine will submit transactions from its local account and keep track of its own sequence number. For more on transaction sequence numbers, please refer to [the transactions doc](./transactions.md).

   * Each machine gets a private/key pair associated with it. Let's say there are only 3 machines: Machine_1, Machine_2, and Machine_3. (In practice, there can be as many machines as the anchor wants.)
   * All three machines are added as Signers to the anchor's base account "baseAccount", with
     a weight that gives them medium rights. The worker machines can then sign on behalf of the base account. (For more on signing, please refer to the [multisig documentation](multi-sig.md).)
   * When a machine (say Machine_2) wants to submit a transaction to the network, it constructs the transaction:
      * source=_public key for Machine_2_
      * sequence number=_sequence number of Machine_2's account_
      * Operation
        * source=_baseAccount_
        * Payment send an asset --> destination account
   * sign it with the private key of Machine_2.

   The benefit of this scheme is that each machine can increment its sequence number and submit a transaction without invalidating any transactions submitted by the other machines.  Recall from the [transactions doc](transactions.md) that all transactions from a source account have their own specific sequence number.  Using worker machines, each with an account, allows this anchor to submit as many transactions as possible without sequence number collisions.

### 3. Long-lived transactions

Transactions that require multiple parties to sign, such as the exchange transaction between Anush and Bridget from example #1, can take an arbitrarily long time. Because all transactions are constructed with specific sequence numbers, waiting on the signatures can block Anush's account. To avoid this situation, a scheme similar to Example #2 can be used.

  Anush would create a temporary account `Anush_temp`, fund `Anush_temp` with XLM, and add the `Anush_account` public key as signer to `Anush_temp` with a weight crossing at least the low threshold.

  A transaction is then constructed:
  * source=_Anush_temp_
  * sequence number=_Anush_temp seq num_
  * Operation 1
    * source=_Anush_account_
    * Payment send XLM -> Bridget_account
  * Operation 2
    * source=_Bridget_account_
    * Payment send BTC -> Anush_account

  The transaction would have to be signed by both Anush_account and Bridget_account, but the sequence
  number consumed will be from account Anush_temp.

  If `Anush_account` wants to recover the XLM balance from `Anush_temp`, an additional operation "Operation 3" can be included in the transaction. If you want to do this, `Anush_temp` must add `Anush_account` as a signer with a weight that crosses the high threshold:
  * Operation 3
    * source=_null_
    * Account Merge -> "Anush_account"
