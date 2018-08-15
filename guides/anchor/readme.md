---
title: Arquitetura
sequence:
  next: 2-bridge-server.md
---

Âncoras são entidades em que pessoas confiam para manter seus depósitos e [emitir crédito](../issuing-assets.md) na rede Stellar por esses depósitos. Todas as transações monetárias na rede Stellar (exceto lumens) ocorrem na forma de crédito emitido por âncoras, tal que as âncoras agem como uma ponte entre moedas preexistentes e a rede Stellar. A maioria das âncoras são organizações como bancos, instituições de poupança, cooperativas de fazendas, bancos centrais e empresas de remessas.

Antes de continuar, recomendamos que você se familizarize com:

- [Emissão de ativos](../issuing-assets.md), a atividade mais básica de uma âncora.
- [Federation](../concepts/federation.md), que permite que uma única conta Stellar represente várias pessoas.
- [Compliance](../compliance-protocol.md), se você estiver sujeito a qualquer tipo de regulamentação financeira.


## Estrutura das Contas

Como uma âncora, você deve manter pelo menos duas contas:

- Uma **conta emissora** ou issuing account, usada apenas para emitir e destruir ativos.
- Uma **conta base** ou base account, usada para transacionar com outras contas Stellar. Ela detém um saldo dos ativos emitidos pela *conta emissora*.

Crie-as na rede de testes usando o [laboratório](https://stellar.org/laboratory/) ou os passos do [guia “para começar”](../get-started/create-account.md).

Neste guia usaremos as seguintes chaves:

<dl>
  <dt>ID da Conta Emissora</dt>
  <dd><code>GAIUIQNMSXTTR4TGZETSQCGBTIF32G2L5P4AML4LFTMTHKM44UHIN6XQ</code></dd>
  <dt>Seed da Emissora</dt>
  <dd><code>SBILUHQVXKTLPYXHHBL4IQ7ISJ3AKDTI2ZC56VQ6C2BDMNF463EON65U</code></dd>
  <dt>ID da Conta Base</dt>
  <dd><code>GAIGZHHWK3REZQPLQX5DNUN4A32CSEONTU6CMDBO7GDWLPSXZDSYA4BU</code></dd>
  <dt>Seed da Base</dt>
  <dd><code>SAV75E2NK7Q5JZZLBBBNUPCIAKABN64HNHMDLD62SZWM6EBJ4R7CUNTZ</code></dd>
</dl>



### Contas de Clientes

Há duas maneiras simples de contabilizar os fundos dos seus clientes:

1. Manter uma conta Stellar para cada cliente. Quando um cliente depositar dinheiro com a sua instituição, você deve pagar uma quantia equivalente de seu ativo na conta Stellar do cliente a partir da sua *conta base*. Quando um cliente precisar obter moeda física de você, deduza a quantia equivalente de seu ativo da conta Stellar do cliente.

    Este método simplifica a contabilidade usando a rede Stellar em vez de seus próprios sistemas internos. Assim, também é possível permitir a seus clientes um pouco mais de controle sobre como as contas funcionam no Stellar.

2. Usar [federation](../concepts/federation.md) e o campo [`memo`](../concepts/transactions.md#memo) em transações para enviar e receber pagamentos em nome de seus clientes. Neste método, transações feitas para seus clientes são todas realizadas usando sua *conta base*. O campo `memo` da transação é usado para identificar o cliente a que se destina o pagamento.

    Usando apenas uma conta requer que você faça um esforço de contabilidade adicional, mas significa que você tem menos chaves para gerenciar e mais controle sobre as contas. Se você já possuir sistemas bancários, esta é a forma mais simples de integrar ao Stellar com eles.

Você pode também criar suas próprias variações dos métodos acima. **Para este guia, seguiremos o método 2 — usar uma única conta Stellar para transacionar em nome de seus clientes..**


## Fluxo de Dados

Para agir como uma âncora, sua infraestrutura terá que:

- Fazer pagamentos.
- Monitorar uma conta Stellar e atualizar as contas de clientes ao receber pagamentos.
- Consultar e responder a requests por endereços federados.
- Cumprir regulações Anti-Money Laundering (AML).

Stellar oferece um [servidor federation](https://github.com/stellar/go/tree/master/services/federation) e um [servidor compliance regulatório](https://github.com/stellar/bridge-server/blob/master/readme_compliance.md) pré-construídos, desenhados para você instalar e integrar com sua infraestrutura já existente. O [servidor bridge](https://github.com/stellar/bridge-server/blob/master/readme_bridge.md) os coordena e simplifica a interação com a rede Stellar. Este guia demonstra como integrá-los com sua infraestrutura, mas você também pode escrever suas próprias versões personalizadas.

### Fazer Pagamentos

When using the above services, a complex payment using federation and compliance works as follows:

![Diagram of sending a payment](assets/anchor-send-payment-compliance.png)

1. A customer using your organization’s app or web site sends a payment using your services.
2. Your internal services send a payment using the bridge server.
3. The bridge server determines whether compliance checks are needed and forwards transaction information to the compliance server.
4. The compliance server determines the receiving account ID by looking up the federation address.
5. The compliance server contacts your internal services to get information about the customer sending the payment in order to provide it to the receiving organization’s compliance systems.
6. If the result is successful, the bridge server creates a transaction, signs it, and sends it to the Stellar network.
7. Once the transaction is confirmed on the network, the bridge server returns the result to your services, which should update your customer’s account.


### Receiving Payments:

When someone is sending a transaction to you, the flow is slightly different:

![Diagram of receiving a payment](assets/anchor-receive-payment-compliance.png)

1. The sender looks up the Stellar account ID to send the payment to based on your customer’s federated address from your federation server.
2. The sender contacts your compliance server with information about the person sending the payment.
3. Your compliance server contacts three services you implement:
    1. A sanctions callback to determine whether the sender is permitted to pay your customer.
    2. If the sender wants to check your customer’s information, a callback is used to determine whether you are willing to share your customer’s information.
    3. The same callback used when sending a payment (above) is used to actually get your customer’s information.
4. The sender submits the transaction to the Stellar network.
5. The bridge server monitors the Stellar network for the transaction and sends it to your compliance server to verify that it was the same transaction you approved in step 3.1.
6. The bridge server contacts a service you implement to notify you about the transaction. You can use this step to update your customer’s account balances.

**While these steps can seem complicated, Stellar’s bridge, federation, and compliance services do most of the work.** You only need to implement four callbacks and create a [stellar.toml](../concepts/stellar-toml.html) file where others can find the URL of your services.

In the rest of this guide, we’ll walk through setting up each part of this infrastructure step by step.

<nav class="sequence-navigation">
  <a rel="next" href="2-bridge-server.md">Next: Bridge Server</a>
</nav>
