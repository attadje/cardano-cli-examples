# How to create an address

Examples on how to create different kinds of addresses on the cardano blockchain.


**Table of contents**
- [Create a payment address with no staking right](#Create-a-payment-address-with-no-staking-right)
- [Create a payment address with staking right](#Create-a-payment-address-with-staking-right)


### Create the wallet directory 

```bash
walletName="Drake"
walletDir=/users/$(whoami)/testnet/priv/wallet/$walletName
mkdir -p $walletDir
```

# 1) Create a payment address with no staking right

An addresse with no staking right have only a payment part, so it mean that the owner of the address have the control of the funds but not have no staking rights of funds.


### Create the address 

```bash
# Create the verification and signing key files for the payment address 
cardano-cli address key-gen \
--verification-key-file $walletDir/drake.addr.vkey \
--signing-key-file $walletDir/drake.addr.skey \

# Build the payment address
cardano-cli address build \
--payment-verification-key-file $walletDir/drake.addr.vkey \
--testnet-magic 1 \
--out-file $walletDir/drake.addr

# Bech32 encoded address
echo $(cat $walletDir/drake.addr) 
# Bech32 decoded address (first number indicate the type of address (type6))
echo $(bech32 <<< $(cat $walletDir/drake.addr))
```

```bash
addr_test1vpyy5wscvu466yavkjjzdxh5nuk4xk5uz32e9jsqzu9n7dslhpwut
60484a3a18672bad13acb4a4269af49f2d535a9c145592ca00170b3f36
```

### Send and get the UTXOs from the address

Use [cardano faucette](https://docs.cardano.org/cardano-testnet/tools/faucet) to send tADA to the address.

```bash
cardano-cli query utxo --address $walletDir/drake.addr --testnet-magic 1
```

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
9088035211336c207022fad02f9b938d08ce4e0f2d2ca969c310ec2edcaaf714     0        10000000000 lovelace + TxOutDatumNone
```

# 2) Create a payment address with staking right

The owner of this address will have the control of the funds and the staking right.


### Create the address

```bash
# Create the verification and signing key files for the payment address 
cardano-cli address key-gen \
--verification-key-file $walletDir/drake-payment.addr.vkey \
--signing-key-file $walletDir/drake-payment.addr.skey \

# Create the verification and signing key files for the stake address  
cardano-cli stake-address key-gen \
--verification-key-file $walletDir/drake-stake.addr.vkey \
--signing-key-file $walletDir/drake-stake.addr.skey \

# Build the payment address
cardano-cli address build \
--payment-verification-key-file $walletDir/drake-payment.addr.vkey \
--stake-verification-key-file $walletDir/drake-stake.addr.vkey \
--testnet-magic 1 \
--out-file $walletDir/drake.addr2

# Bech32 encoded address
echo $(cat $walletDir/drake.addr2) 
# Bech32 decoded address (first number indicate the type of address (type 0))
echo $(bech32 <<< $(cat $walletDir/drake.addr2))
```

```bash
addr_test1qz83jdf92qc3ztjhsa8qxcwsccsj27gltd9x5gry7gswx74d28swvcn7t7yysu0gdeavmrfge4ktqva7eyewzde0av3spdpec5
008f1935255031112e57874e0361d0c62125791f5b4a6a2064f220e37aad51e0e6627e5f884871e86e7acd8d28cd6cb033bec932e1372feb23
```

### Get the UTXOs from the address

```bash
cardano-cli query utxo --address $walletDir/drake.addr2 --testnet-magic 1
```

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
a9ed1f9dd03fb6226f7ceab1c17e8dfb61ae0fadef7a51c0e53b672914676ca6     0        10000000000 lovelace + TxOutDatumNone
```
