---
title: Stellar Smart Contracts
---

Stellar pode ser usado para construir smart contracts sofisticados. Smart contracts são programas de computador que podem executar automaticamente um acordo baseado em lógica de programação.

O conceito de integrar tecnologia e contratos jurídicos data desde os anos 1950, quando acadêmicos construiram métodos computacionais que poderiam executar regras jurídicas sem envolver processos jurídicos tradicionais. Foram definidos formalmente por Nick Szabo em 1997:

> Smart contracts combinam protocolos com interfaces de usuário para formalizar e proteger relações por meio de redes informáticas. Os objetivos e princípios do design desses sistemas são derivados de princípios jurídicos, teoria econômica e teorias de protocolos confiáveis e seguros.

Nos últimos anos, a tecnologia blockchain tem tornado possível uma nova geração de smart contracts com armazenamento imutável de termos do acordo, autorização criptográfica e transferências de valor integradas.

Para a Rede Stellar, smart contracts se manifestam como Stellar Smart Contracts. Um **Stellar Smart Contract** (SSC) é expresso como composições de transações que são conectadas e executadas usando vários condicionamentos. A seguir estão exemplos de condicionamentos que podem ser considerados e implementados ao criar SSCs:

- *Multisignature* - Que chaves são necessárias para autorizar certa operação? Quais partes devem concordar em dada circunstância para executar os passos?

Multisignature ou multiassinaturas é o conceito de exigir assinaturas de múltiplas partes para assinar transações que originadas de uma conta. Por meio de pesos das assinaturas e limiares, cria-se uma representação de poder nas assinaturas.

- *Batching/Atomicidade* - Que operações devem ocorrer todas ao mesmo tempo ou falhar? O que deve acontecer para forçar isso a falhar ou passar?

Batching é o conceito de incluir múltiplas operações em uma transação. Atomicidade é a garantia que, dada uma série de operações submetidas à rede, se uma operação falhar, todas as operações na transação falham.

- *Sequência* - Em que ordem uma série de transações deverá ser processada? Quais são as limitações e relações de dependência?

O conceito de sequência é representado na Rede Stellar por meio do número sequencial. Utilzando números sequenciais ao manipular transações, pode-se garantir que transações específicas não terão sucesso se uma transação alternativa for submetida.

- *Limites de tempo* - Quando uma transação pode ser processada?

Limites de tempo ou time bounds são restrições ao período de tempo durante o qual uma transação é válida. Usar os limites de tempo permite que períodos de tempo sejam representados em um SSC.

Esta visão geral apresenta dois padrões de design comuns que podem ser usados para criar SSCs na Rede Stellar. As transações podem ser traduzidas em API requests ou podem ser executadas usando o [Laboratório Stellar](https://www.stellar.org/laboratory/).


## Conta Escrow Multisignature para Duas Partes com Bloqueio Temporal e Recuperação
### Exemplo de Caso de Uso
Ben Bitdiddle vende 50 tokens CODE para Alyssa P. Hacker, com a condição de que Alyssa não poderá revender os tokens até ter se passado um ano. Ben não confia completamente em Alyssa, então Ben sugere que ele detenha os tokens para Alyssa durante o ano.

Alyssa protesta. Como ela irá saber se o Ben ainda terá os tokens depois de um ano? Como ela pode confiar que ele irá realmente entregá-los?

Além disso, às vezes Alyssa é meio esquecida. Existe a possibilidade de que ela não irá lembrar de resgatar seus tokens depois de decorrido um ano. Ben gostaria de embutir um mecanismo de recuperação para, caso Alyssa não resgate seus tokens, eles podem ser recuperados. Assim, os tokens não serão perdidos para sempre.

### Implementação
Um acordo escrow é criado entre duas entidades: a origem – a entidade que financia o acordo, e o alvo – a entidade que recebe os fundos no final do contrato.

Três contas são necessárias para executar um contrato escrow entre duas partes: uma conta fonte, uma conta de destino e uma conta escrow. A conta fonte é a conta da origem que está iniciando e fundando o acordo escrow. A conta de destino é a conta do alvo que por fim irá ganhar controle dos fundos. A conta escrow é criada pela origem e detém os fundos durante o período de travamento.

Dois períodos de tempo devem ser estabelecidos e acordados para este acordo escrow: um período de trancamento, durante o qual nenhuma parte pode manipular (transferir, utilizar) os ativos, e um período de recuperação, depois do qual a origem consegue recuperar os fundos da conta escrow.

Cinco transações são usadas para criar um contrato escrow, explicadas abaixo em ordem de criação. As variáveis a seguir serão usadas na explicação:
-  **N**, **M** – número sequencial da conta escrow e da conta fonte, respectivamente; N+1 representa próximo número sequencial da transação, e de assim em diante
- **T** – o período de trancamento
- **D** – a data a partir da qual começar o período de trancamento
- **R** – o período de recuperação

Para o padrão de design descrito abaio, o ativo sendo trocado é o ativo nativo. A ordem de envio das transações à rede Stellar é diferente da ordem de criação. A imagem a seguir mostra a ordem de envio, no que diz respeito ao tempo:

![Diagram Transaction Submission Order for Escrow Agreements](assets/ssc-escrow.png)

#### Transação 1: Criar uma Conta Escrow
**Conta**: conta fonte  
**Número Sequencial**: M  
**Operações**:
- [Create Account](../concepts/list-of-operations.md#create-account): criar conta escrow no sistema
	 - saldo inicial: [saldo mínimo](../concepts/fees.md#saldo-mínimo-da-conta) + [tarifa de transação](../concepts/fees.md#tarifa-de-transação)

**Signatários**: source account

A Transação 1 é submetida à rede pela origem por meio da conta fonte. Cria-se a conta escrow, financiada com a reserva mínima atual, e dá-se à origem acesso às chaves pública e privada da conta escrow. A conta escrow é financiada com o saldo mínimo, sendo assim uma conta válida na rede. Ela recebe dinheiro adicional para custear a tarifa que incorre ao transferir os ativos no fim do acordo escrow. Ao criar novas contas, recomenda-se financiar a conta com um saldo maior que o saldo inicial calculado.


#### Transação 2: Habilitar Multi-sig
**Conta**: conta escrow   
**Número Sequencial**: N  
**Operações**:
- [Set Option - Signer](../concepts/list-of-operations.md#set-options): Adicionar a conta de destino como um signatário com peso em transações para a conta escrow
	 - weight: 1
- [Set Option - Thresholds & Weights](../concepts/list-of-operations.md#set-options): definir o peso da chave mestra e alterar os limiares para exigirem todas as assinaturas (2 de 2 signers)
	 - master weight: 1
	 - low threshold: 2
	 - medium threshold: 2
	 - high threshold: 2

**Signatários**: conta escrow

A Transação 2 é criada e submetida à rede. É feita pela origem usando a conta escrow, já que a origin tem controle sobre a conta escrow nesse momento. A primeira operação adiciona à conta escrow a conta de destino como um segundo signatário com um peso igual a 1.

Por padrão, as limiares são desiguais. A segunda operação define o peso da chave mestra como igual a 1, nivelando seu peso com o peso da conta de destino. Na mesma operação, os limiares são definidos como iguais a 2. Isso faz que qualquer tipo de transação que origine da conta escrow exija que todas as assinaturas tenham um peso total de 2. Nesse momento, o peso de assinar com tanto a conta escrow e a conta de destino são somados, resultando em um total igual a 2. Isso garante que, desse ponto em diante, ambas a conta escrow e a conta de destino (a origem e o alvo) precisem assinar todas as transações referentes à conta escrow. Isso dá controle parcial da conta escrow ao alvo.

#### Transação 3: Destravar  
**Conta**: conta escrow  
**Número Sequencial**: N+1  
**Operações**:
- [Set Option - Thresholds & Weights](../concepts/list-of-operations.md#set-options): definir peso da chave mestra e alterar limiares para exigirem apenas 1 assinatura
	 - master weight: 0
	 - low threshold: 1
	 - medium threshold: 1
	 - high threshold: 1

**Limites de Tempo**:
- tempo mínimo: data do destravamento
- tempo máximo: 0  

**Signatário Imediato**: conta escrow  
**Signatário Eventual**: conta de destino


#### Transação 4: Recuperação
**Conta**: conta escrow  
**Número Sequencial**: N+1  
**Operações**:
- [Set Option - Signer](../concepts/list-of-operations.md#set-options): remover a conta de destino como signatário
	 - weight: 0  
 - [Set Option - Thresholds & Weights](../concepts/list-of-operations.md#set-options): definir o peso da chave mestra e alterar limiares para exigirem apenas 1 assinatura
	 - low threshold: 1
	 - medium threshold: 1
	 - high threshold: 1  

**Limites de Tempo**:
- tempo mínimo: data da recuperação
- tempo máximo: 0

**Signatário Imediato**: conta escrow  
**Signatário Eventual**: conta de destino  

A Transação 3 e a Transação 4 são criadas e assinadas pela conta escrow pela origem. A origem então dá as transações 3 e 4, em [forma XDR](https://www.stellar.org/developers/horizon/reference/xdr.html), ao alvo para que ele assine usando a conta de destino. O alvo então as publica para a origem [verificá-las](https://www.stellar.org/laboratory/#xdr-viewer?type=TransactionEnvelope&network=test) e salvá-las em um lugar seguro. Uma vez assinadas por ambas as partes, essas transações não podem ser modificadas. Tanto a origem como o alvo devem reter uma cópia dessas transações assinadas em sua forma XDR, e as transações podem ser armazenadas em um lugar publicamente acessível sem receio de alterações maliciosas.

A Transação 3 e a Transação 4 são criadas e assinadas antes da conta escrow ser financiada, e possuem o mesmo número de transação. Isso é feito para garantir que ambas as partes estejam de acordo. Se as circunstâncias mudarem antes antes de uma dessas duas transações forem submetidas, ambos a origem e o alvo devem autorizar transações utilizando a conta escrow. Isso é representado pela exigência por assinaturas de tanto a conta de destino como a conta escrow.

A Transação 3 remove a conta escrow como um signatário para transações geradas a partir de si mesma. Esta transação transfere o controle total da conta escrow ao alvo. Depois do fim do período de trancamento, a única conta que é necessária para assinar transações a partir da conta escrow é a conta de destino. A data de destravamento (D+T) é a primeira data em que a transação de destravamento pode ser submetida. Se a Transação 3 for submetida antes dessa data, a transação não será válida. O tempo máximo é definido como igual a 0, para denotar que a transação não possui uma data de expiração.

A Transação 4 existe para a recuperação da conta, caso o alvo não submeta a transação de destravamento. Ela remove a conta de destino como signatário, e reestabelece os pesos para apenas exigir sua própria assinatura. Isso retorna o controle total da conta escrow à origem. A Transação 4 pode apenas ser submetida após a data de recuperação (D+T+R), e não possui data de expiração.

A Transação 3 pode ser submetida a qualquer momento durante o período de recuperação (R). Se o alvo não submeter a Transação 3 para permitir acesso aos fundos na conta escrow, a origem pode submeter a Transação 4 após a data de recuperação. A origem pode retomar os ativos trancados caso deseje, já que a Transação 4 faz que o alvo não seja mais necessário para assinar transações da conta escrow. Após a data de recuperação, ambas Transações 3 e 4 são válidas e possíveis de serem enviadas à rede, mas apenas uma transação será aceita pela rede. Isso é garantido pelo recurso de que ambas as transações têm o mesmo número sequencial.

Para resumir: se a Transação 3 não for submetida pelo alvo, a Transação 4 será submetida pela origem depois do período de recuperação.

#### Transação 5: Funding  
**Conta**: source account  
**Número Sequencial**: M+1  
**Operações**:
- [Payment](../concepts/list-of-operations.md#payment): Pay the escrow account the appropriate asset amount  

**Signer**: source account

Transação 5 is the transaction that deposits the appropriate amount of assets into the escrow account from the source account. It should be submitted once Transação 3 and Transação 4 have been signed by the destination account and published back to the source account.

## Joint-Entity Crowdfunding
### Use Case Scenario
Alyssa P. Hacker needs to raise money to pay for a service from a company, Coding Tutorials For Dogs, but she wants to source the funding from the public via crowdfunding. If enough people donate, she will be able to pay for the service directly to the company. If not, she will have a mechanism to return the donations. To guarantee her trustworthiness and reliability to the donors, she decides to asks Ben Bitdiddle if he’s willing to help her with getting people to commit to the crowdfunding. He will also vouch for Alyssa’s trustworthiness to his friends as a way to get them to donate to the crowdfunding efforts.

### Pattern Implementation
In the simplest example, a crowdfunding smart contract requires at least three parties: two of which (from here out called party A and party B) agree to sponsor the crowdfunding, and a third to which the final funds will be given (called the target). A token must be created as the mechanism to execute the crowdfunding. The participation token utilized, as well as a holding account, must be created by one of two parties. A holding account issues participation tokens that can be priced at any value per token. The holding account collects the funding, and, after the end of the crowdfunding period, will return contributors funds if the value goal isn't met.

Five transactions are used to create a crowdfunding contract. The following variables are used in explaining the formulation of the contract:
- **N**, **M** - sequence number of party A's account and the holding account, respectively; N+1 means the next sequential transaction number, and so on
- **V** - total value the crowdfunding campaign is looking to raise
- **X** - value at which the tokens will be sold

There are four accounts used for creating a basic crowdfunding schema. First is the holding account, which is the account that deals with collecting and interacting with the donors. It requires the signature of both party A and party B in order to carry out any transactions. The second is the goal account, the account owned by the target to which the crowdfunded value is delivered to on success. The two remaining accounts are respectively owned by party A and party B, who are running the crowdfunding.

The transactions that create this design pattern can be created and submitted by any party sponsoring the crowdfunding campaign. The transactions are presented in order of creation. The order of submission to the Stellar Network is conditional, and depends on the success of the crowdfunding campaign.

![Diagram Transaction Submission Order for Crowdfunding Campaigns](assets/ssc-crowdfunding.png)


#### Transação 1: Create the Holding Account
**Conta**: party A  
**Número Sequencial**: M  
**Operações**:
- [Create Account](../concepts/list-of-operations.md#create-account): create holding account in system
	- [starting balance](../concepts/fees.md#minimum-account-balance): minimum balance

**Signatários**: party A

#### Transação 2: Add signers
**Conta**: holding account  
**Número Sequencial**: N  
**Operações**:
 - [Set Option - Signer](../concepts/list-of-operations.md#set-options): Add party A as a signer with weight on transactions for the escrow account
	- weight: 1
 - [Set Option - Signer](../concepts/list-of-operations.md#set-options): Add party B as a signer with weight on transactions for the escrow account
	- weight: 1
 - [Set Option - Thresholds & Weights](../concepts/list-of-operations.md#set-options): remove master keys and change thresholds weights to require all other signatures (2 of 2 signers)
	- master weight: 0
	- low threshold: 2
	- medium threshold: 2
	- high threshold: 2

**Signatários**: holding account


Transação 1 and 2 are created and submitted by one of the two parties sponsoring the crowdfunding campaign. Transação 1 creates the holding account. The holding account is funded with a starting balance in order to make it valid on the network. It is recommended that when creating new accounts to fund the account with a balance larger than the calculated starting balance. Transação 2 removes the holding account as a signer for its own transactions, and adds party A and party B as signers. From this point on, all parties involved must agree and sign all transactions coming from the holding account. This trust mechanism is in place to protect donors from one party carrying malicious actions.  

After Transação 2, the holding account should be funded with the tokens to be used for the crowdfunding campaign, as well as with enough lumens to cover the transaction fees for all of the following transactions.

#### Transação 3: Begin Crowdfunding
**Conta**: holding account  
**Número Sequencial**: N+1  
**Operações**:
- [Manage Offer - Sell](../concepts/list-of-operations.md#manage-offer): sell participation tokens at a rate of X per token

**Signer**: party A’s account, party B’s account

Transação 3 is created and submitted to the network to begin the crowdfunding campaign. It creates an offer on the network that sells the participation tokens at a rate of X per token. Given a limited amount of tokens are created for the crowdfunding campaign, the tokens are priced in a manner that enables a total of V to be raised through sales.

#### Transação 4: Crowdfunding Succeeds  
**Conta**: holding account  
**Número Sequencial**: N+2    
**Operações**:
- [Payment](../concepts/list-of-operations.md#payment): send V from the holding account to the goal account


**Time Bounds**:
- minimum time: end of crowdfunding period
- maximum time: 0

**Signatários**: party A’s account, party B’s account

#### Transação 5: Crowdfunding Fails
**Conta**: holding account    
**Número Sequencial**: N+3      
**Operações**:
- [Manage Offer - Cancel](../concepts/list-of-operations.md#manage-offer): cancel pre-existing offer to sell tokens
 - [Manage Offer - Buy](../concepts/list-of-operations.md#manage-offer): holding account buys participation tokens at a rate of X per token

**Time Bounds**:
- minimum time: end of crowdfunding period
- maximum time: 0

**Signatários**: party A’s account, party B’s account  

Transação 4 and Transação 5 are pre-signed, unsubmitted transactions that are published. Both transactions have a minimum time of the end of the crowdfunding period to prevent them from being submitted earlier than agreed upon by the sponsoring parties. They can be submitted by anyone upon the end of the crowdfunding. Transação 4 transfers the raised amount to the goal account. Transação 5 prevents all remaining tokens from being sold by canceling the offer and enables donors to create offers to sell back tokens to the holding account.

Security is provided through sequence numbers. As noted, the sequence number for Transação 4 is *N+2* and the sequence number for Transação 5 is *N+3*. These sequential sequence numbers demand that both Transação 4 and Transação 5 are submitted to the network in the appropriate order.  

The crowdfunding was a failure when not enough funds were raised by the expected date. This is the equivalent to not selling all of the participation tokens. Transação 4 is submitted to the network, but it will fail. The holding account will have enough lumens to pay the transaction fee, so the transaction will be considered in consensus and a sequence number will get consumed. An error will occur, though, because there will not be enough funds in the account to cover the actual requested amount of the payment. Transação 5 is then submitted to the network, enabling contributors to sell back their tokens. Additionally, Transação 5 cancels the holding account’s ability to sell participation tokens, halting the status of the crowdfunding event.  

The crowdfunding is a success if V was raised by the appropriate time. Raising enough funds is equivalent to having all participation tokens being purchased from the holding account. Transação 4 is submitted to the network and will succeed because there are enough funds present in the account to fulfill the payment operation, as well as cover the transaction fee. Transação 5 will then be submitted to the network, but will fail. The holding account will have enough lumens to pay the transaction fee, so the transaction will be considered in consensus and a sequence number will get consumed. The transaction will succeed, but because the holding account will not have the funds to buy back the tokens, participants will not be able to make attempts to recover their funds.

#### Bonus: Crowdfunding Contributors
The following steps are carried out in order to become a contributor to the crowdfunding:
1. [Create a trustline](../concepts/list-of-operations.md#change-trust) to the holding account for participation tokens
	- The trustline creates trust between the contributor and the holding accounts, enabling transactions involving participation tokens to be valid
2. [Create an offer](../concepts/list-of-operations.md#manage-offer) to buy participation tokens
	- The contributor account will receive participation tokens and the holding account will receive the value
3. If the crowdfunding:
	- succeeds - do nothing
	- fails - create an offer to sell participation tokens, enabling the contributor to get back their value invested

## SSC Best Practices
When it comes to designing a smart contract, parties must come together and clearly outline the purpose of the contract, the cooperation between parties, and the desired outcomes. In this outline, clear conditions and their outcomes should be agreed upon. After establishing the conditions and their outcomes, the contract can then be translated to a series of operations and transactions. As a reminder, smart contracts are created using code. Code can contain bugs or may not perform as intended. Be sure to analyze and agree upon all possible edge cases when coming up with the conditions and outcomes of the smart contract.


## Resources

- [Jurimetrics - The Next Steps Forward](http://heinonline.org/HOL/LandingPage?handle=hein.journals/mnlr33&div=28&id=&page) - Lee Loevinger
- [Formalizing and Securing Relationships on Public Networks](http://firstmonday.org/article/view/548/469) - Nick Szabo
- [Smart Contracts: 12 Use Cases for Business and Beyond](https://bloq.com/assets/smart-contracts-white-paper.pdf) - Chamber of Digital Commerce
- [Concept: Transactions](https://www.stellar.org/developers/guides/concepts/transactions.html) - Stellar<span>.org
- [Concept: Multisignature](https://www.stellar.org/developers/guides/concepts/multi-sig.html) - Stellar<span>.org
- [Concept: Time Bounds](https://www.stellar.org/developers/guides/concepts/transactions.html#time-bounds) - Stellar<span>.org
- [Concept: Trustlines](https://www.stellar.org/developers/guides/concepts/assets.html#trustlines) - Stellar<span>.org
