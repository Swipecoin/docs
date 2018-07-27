---
title: Tarifas
---

A rede Stellar requer pequenas [tarifas por transação](#tarifa-de-transação) e [saldos mínimos nas contas](#minimum-account-balance) para prevenir que pessoas sobrecarreguem a rede e para ajudar com priorização.

Há dois valores especiais usados para calcular tarifas:

1. A **base fee** (tarifa base), atualmente de 100 stroops, é usada nas tarifas de transação.
2. A **base reserve** (reserva base), atualmente de 0.5 XLM, é usada nos saldos mínimos das contas.


## Tarifa de transação

A tarifa por uma transação é o número de operações contidas pela transação, multiplicado pela **base fee**, que é de **100 stroops** (0.00001 XLM).

```math-formula
([# de operações] * [base fee])
```

Por exemplo, uma transação que permite a confiança na trustline de uma conta *(operação 1)* e envia um pagamento para aquela conta *(operação 2)* teria uma tarifa de $$2 * [base fee] = 200 stroops$$.

Stellar subtrai toda a tarifa da [conta fonte](./transactions.md#conta-fonte) da transação, independente de que contas estão envolvidas em cada operação ou quem assinou a transação.


### Transaction Limits

Each Stellar node usually limits the number of transactions that it will propose to the network when a ledger closes. If too many transactions are submitted, nodes propose the transactions with the highest fees for the ledger’s transaction set. Transactions that aren’t included are held for a future ledger, when fewer transactions are waiting.

See [transaction life cycle](./transactions.md#life-cycle) for more information.

## Fee Pool

The fee pool is the lot of lumens collected from [transaction fees](./fees.md#transaction-fee).

Stellar does not retain these lumens. They are distributed in the weekly process of [inflation voting](./inflation.md).

If there are any unallocated lumens after the vote, those lumens return to the fee pool for dispersal in the next round.

## Minimum Account Balance

All Stellar accounts must maintain a minimum balance of lumens. Any transaction that would reduce an account's balance to less than the minimum will be rejected with an `INSUFFICIENT_BALANCE` error.

The minimum balance is calculated using the **base reserve,** which is **0.5 XLM**:

```math-formula
(2 + [# of entries]) * [base reserve]
```

The minimum balance for a basic account is $$2 * [base reserve]$$. Each additional entry costs the base reserve. Entries include:

- Trustlines
- Offers
- Signers
- Data entries

For example, an account with 1 trustline and 2 offers would have a minimum balance of $$(2 + 3) * [base reserve] = 2.5 XLM$$.


## Fee Changes

The **base reserve** and **base fee** can change, but should not do so more than once every several years. For the most part, you can think of them as fixed values. When they are changed, the change works by the same consensus process as any transaction. For details, see [versioning](https://www.stellar.org/developers/guides/concepts/versioning.html).

You can look up the current fees by [checking the details of the latest ledger](../../horizon/reference/endpoints/ledgers-single.md).
