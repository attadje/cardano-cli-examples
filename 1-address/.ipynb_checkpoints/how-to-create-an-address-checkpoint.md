---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.14.1
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

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
walletDir=/users/$(whoami)/testnet/priv/wallet/drake

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

### Send and get the UTXOs from the address

Use [cardano faucette](https://docs.cardano.org/cardano-testnet/tools/faucet) to send tADA to the address.

```bash
walletDir=$(cat /users/$(whoami)/testnet/priv/wallet/drake/drake.addr)
cardano-cli query utxo --address $walletDir --testnet-magic 1
```

# 2) Create a payment address with staking right

The owner of this address will have the control of the funds and the staking right.


### Create the address

```bash
walletDir=/users/$(whoami)/testnet/priv/wallet/drake

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

### Get the UTXOs from the address

```bash
walletDir=$(cat /users/$(whoami)/testnet/priv/wallet/drake/drake.addr2)
cardano-cli query utxo --address $walletDir --testnet-magic 1
```

```python
!jupytext --to markdown how-to-create-an-address.ipynb
```

```python

```
