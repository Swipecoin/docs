---
title: Configurar um Servidor Bifrost
---

Bifrost é um serviço que permite que usuários movam BTC/ETH à rede Stellar. Pode ser usado tanto para representar BTC ou ETH na rede como para trocá-los por outro token personalizado. Isso é particularmente útil para ICOs (Initial Coin Offerings).
Este guia se foca em como configurar o servidor Bifrost para mover ETH à rede Stellar.

## O que você vai precisar

- Base de Dados Postgresql
- Nó de Bitcoin/Ethereum
- Servidor Bifrost

## Configurar Postgresql

Isso não será abordado aqui, sendo que já há boas documentações sobre como preparar isso online, dependendo do seu OS.

## Configurar um nó de Ethereum

- Baixe o [geth versão 1.7.1 ou acima](https://geth.ethereum.org/downloads/).
- Extraia os conteúdos do arquivo baixado
- Inicie o listener na rede de testes

```bash  
./geth --testnet --rpc
```

- Leia mais sobre [como administrar o geth](https://github.com/ethereum/go-ethereum)

## Criar uma Ordem de Venda para seu Ativo

O Bifrost vai trocar automaticamente os BTC ou ETH recebidos pelo seu token personalizado. Para isso acontecer, deve haver uma ordem de venda para os pares de ativos CUSTOM-TOKEN/BTC ou CUSTOM-TOKEN/ETH na exchange distribuída do Stellar.

Por exemplo, digamos que a taxa de câmbio seja de 1 `TOKE` por 0.2 `ETH`. Você pode usar o [Laboratório Stellar](https://www.stellar.org/laboratory/) para criar e submeter uma operação manage offer:

- Vá até a aba "Transaction Builder"
- Repare no botão no canto superior direito da página com "test/public". Tome cuidado para deixá-lo como "public" para transações reais e "test" para transações na testnet
- Preencha o formulário na página:
  - Coloque a conta fonte na Source account (emissora do ativo ou conta distribuidora)
  - Clique no botão "Fetch next sequence number"
  - Desça e selecione um "Operation Type" como "Manage Offer"
  - Para "Selling": selecione Alphanumeric 4
  - Insira o Asset Code `TOKE`
  - Insira o Issuer Account ID: ID da conta emissora
  - Para buying: selecione Alphanumeric 4
  - Insira o Asset Code `ETH`
  - Insira o Account ID: Conta Emissora
  - Amount: Insira a quantidade de TOKE que você está vendendo
  - Price: O preço é representado na razão para o ativo a ser comprado, ou seja, `1 selling_asset = X buying_asset`. No nosso caso, já que queremos vender 1TOKE por 0.2ETH, o valor aqui deve ser igual a 0.2
  - Offer ID: Insira "0" para criar uma nova oferta
  - Desça e clique em "Sign transaction in Signer"
  - Insira a chave secreta da emissora do ativo ou conta distribuidora ou assine uma transação usando um dispositivo Ledger
  - Clique em "Submit to Post transaction"
  - Clique em "Submit".

Os passos acima irão criar uma ordem de venda para seu ativo na exchange distribuída.

## Setting up Bifrost

- Download [the latest version](https://github.com/stellar/go/releases/tag/bifrost-v0.0.2) and extract its component into a folder.
- Rename downloaded file to `bifrost-server` (optional)
- Generate your ethereum master public keys according to [BIP-0032](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).  You can download [this implementation](https://iancoleman.io/bip39/) from GitHub and generate the keys on an offline machine. You can also extract master public key from the Ledger device.
- Create a config file: `bifrost.cfg`, similar to the one below:

<code-example name="bifrost.cfg">

```toml
port = 8002
using_proxy = false
access_control_allow_origin_header = "*"

#uncomment bitcoin parameters if you will be accepting BTC
#[bitcoin]
#master_public_key = "xpub6DxSCdWu6jKqr4isjo7bsPeDD6s3J4YVQV1JSHZg12Eagdqnf7XX4fxqyW2sLhUoFWutL7tAELU2LiGZrEXtjVbvYptvTX5Eoa4Mamdjm9u"
#rpc_server = "localhost:18332"
#rpc_user = "user"
#rpc_pass = "password"
#testnet = true
#minimum_value_btc = "0.0001"
#token_price = "1"

[ethereum]
master_public_key = "xpub68VNckQn96Y23e5GsGh9X7zVmbPT4ho5Vdf6RdgMGG3LyNhH2cLFDCib9zgn8QWgj261xu7MYbmBsX8Fp5VkfDUrecUnpEGWkyCo7qK2gxn"
rpc_server = "localhost:8545"
network_id = "3"
minimum_value_eth = "0.00001"
token_price = "1"

[stellar]
issuer_public_key = "GDNPOP72ZO6AZXZ7LQJ4GKYT7UIH4JEG4X3ZRZBFUCRB467RNV3SFK5D"
distribution_public_key = "GCSSFPPVERDH4ZPWH5BSONEJERHCVS4DPZRWJG3FP3STOA5ZFTD3GMZ5"
signer_secret_key = "SB3WH2NLOFW2K2B5MWN34CWF35ZLQXH33ABZYL7KZFKTVEFP72Q574LM"
token_asset_code = "ZEN"
needs_authorize = false
horizon = "https://horizon-testnet.stellar.org"
network_passphrase = "Test SDF Network ; September 2015"
starting_balance = "4"

[database]
type="postgres"
dsn="postgres://stellar:pass1234@localhost/bifrost?sslmode=disable"
```

</code-example>


- Complete the config file with the values as described [here](https://github.com/stellar/go/tree/master/services/bifrost#config)
- Check that you have the correct master public keys by running:

```bash
./bifrost-server check-keys
```

Output should be similar to:

```bash
MAKE SURE YOU HAVE PRIVATE KEYS TO CORRESPONDING ADDRESSES:
Bitcoin MainNet:
No master key set...
Ethereum:
0 0xAF484B67cC184259d22edfA4aFe874f68275B714
1 0x0163DF805B87A9aB2dd3177f674B275163272630
2 0x42069115ba5802736444Aacba5F0bD4a9a007E69
3 0xA219bCCFeE13B94fcf505120Cb7b8CD090749A4e
4 0x3AB571B247b0CF45E44d111691F9D03eE1bfE705
5 0x1Fe3101B058Aa3b6Fb69B84Cd1cc7766959dcFc2
6 0x1B07c658614F6D4F13225b63d76055EaB07114c9
7 0x3C3459c47388163E56e544F9616ac0E46668420E
8 0x08fb48e4f54f699cDa3B97cd97D9fB6A594354D7
9 0xC5CD4b9E6c5D9c0cd1AAe5A52f6DCA3d20CF08BC
```

## Start the Bifrost server

Once you are done setting up the config file, you can start the server by running:

```bash
./bifrost-server server
```
The Bifrost server will be responsible for generating ethereum addresses, listening for payments on these addresses and transferring the token purchased to the user.

## Using Bifrost JS SDK.

The Bifrost JS SDK provides a way for a client to communicate with the Bifrost server.
Download the [latest version](https://github.com/stellar/bifrost-js-sdk/releases) of the SDK, and include it in your frontend application. See the [example html file](https://github.com/stellar/bifrost-js-sdk/blob/master/example.html) in the [bifrost-js-sdk repo](https://github.com/stellar/bifrost-js-sdk) for an example on how this can be implemented.
