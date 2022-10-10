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

An addresse with no stakink right have only a payment part, so it mean that the owner of the address have the control of the funds but not on the staking rights.


### Create the address 

```bash
walletDir=/users/$(whoami)/testnet/priv/wallet/drake

# Create the verification and signing key files for the payment address 
cardano-cli address key-gen \
--verification-key-file $walletDir/djessy.addr.vkey \
--signing-key-file $walletDir/djessy.addr.skey \

# Build the payment address
cardano-cli address build \
--payment-verification-key-file $walletDir/djessy.addr.vkey \
--testnet-magic 1 \
--out-file $walletDir/djessy.addr

# Bech32 encoded address
echo $(cat $walletDir/djessy.addr) 
# Bech32 decoded address (first number indicate the type of address (type6))
echo $(bech32 <<< $(cat $walletDir/djessy.addr))
```

### Send and get the UTXOs from the address

Use [cardano faucette](https://docs.cardano.org/cardano-testnet/tools/faucet) to send tADA to the address.

```bash
walletDir=$(cat /users/$(whoami)/testnet/priv/wallet/drake/djessy.addr)
cardano-cli query utxo --address $walletDir --testnet-magic 1
```

# 2) Create a payment address with staking right

The owner of this address will have the control of the funds and the staking right.


### Create the address

```bash
walletDir=/users/$(whoami)/testnet/priv/wallet/drake

# Create the verification and signing key files for the payment address 
cardano-cli address key-gen \
--verification-key-file $walletDir/djessy-payment.addr.vkey \
--signing-key-file $walletDir/djessy-payment.addr.skey \

# Create the verification and signing key files for the stake address  
cardano-cli stake-address key-gen \
--verification-key-file $walletDir/djessy-stake.addr.vkey \
--signing-key-file $walletDir/djessy-stake.addr.skey \

# Build the payment address
cardano-cli address build \
--payment-verification-key-file $walletDir/djessy-payment.addr.vkey \
--stake-verification-key-file $walletDir/djessy-stake.addr.vkey \
--testnet-magic 1 \
--out-file $walletDir/djessy.addr2

# Bech32 encoded address
echo $(cat $walletDir/djessy.addr2) 
# Bech32 decoded address (first number indicate the type of address (type 0))
echo $(bech32 <<< $(cat $walletDir/djessy.addr2))
```

### Get the UTXOs from the address

```bash
walletDir=$(cat /users/$(whoami)/testnet/priv/wallet/drake/djessy.addr2)
cardano-cli query utxo --address $walletDir --testnet-magic 1
```
