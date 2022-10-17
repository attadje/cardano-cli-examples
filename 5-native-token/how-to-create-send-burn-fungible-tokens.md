# How to create fungible tokens

Native tokens on cardano are executed directly on-chain,  
unlike the ERC-20 tokens for example, which are executed by means of smart contracts on the Ethereum blockchain.

Fungible token property:  
  * Numerous quantity  
  * Equivalent to each other  
  * No distinction between each other  


### 1) Defines the token architecture


Token name: `MARS`  
Token supply: `1 000 000 000`   
Token time constraint: `No time constraint`  
Who can mint: `One address`  


### 2) Create a verification key and signing key for mint and burn the token

```bash
cardano-cli address key-gen \
--verification-key-file priv/MARS/marsToken.vkey \
--signing-key-file priv/MARS/marsToken.skey
```

### 3) Create the monetary policy script

```bash
MARS_VKEY=$(cardano-cli address key-hash --payment-verification-key-file priv/MARS/marsToken.vkey)
SCRIPT_PATH=policy/marsToken.script

# Monetary script
echo "{" >> $SCRIPT_PATH
echo "  \"type\": \"all\"," >> $SCRIPT_PATH
echo "  \"scripts\":" >> $SCRIPT_PATH
echo "  [" >> $SCRIPT_PATH
echo "    {" >> $SCRIPT_PATH
echo "      \"type\": \"sig\"," >> $SCRIPT_PATH
echo "      \"keyHash\": \"$MARS_VKEY\"" >> $SCRIPT_PATH
echo "    }" >> $SCRIPT_PATH
echo "  ]" >> $SCRIPT_PATH
echo "}" >> $SCRIPT_PATH

cat policy/marsToken.script
```

```bash
{
  "type": "all",
  "scripts":
  [
    {
      "type": "sig",
      "keyHash": "d5260b3f8a8b676dbf79b735c5f59881290405af8728a4893ac96291"
    }
  ]
}
```

### 4) Create the policy ID

The policy ID is the hash of the policy script.

```bash
cardano-cli transaction policyid \
--script-file policy/marsToken.script \
> policy/mars.policyid

cat policy/mars.policyid
```

```bash
4fd78aae5e7643885c5f0c63d26641e2e05870d8544af7c6c239ff46
```

### 5) Mint the token


  * #### 5.1 Get the UTXO from the address you want to mint the token 

```bash
MINT_ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr)

cardano-cli query utxo \
--address $MINT_ADDR \
--testnet-magic 1
```

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
0be38e65bec5eebde5ca3308a75e5c8adcd1b184534b445c984185d7d6f9aee5     1        100000000 lovelace + TxOutDatumNone
8d4f02739f9f921e7d50cd7765527242a1ca5bd37dc479a550af99463290d89f     1        100000000 lovelace + TxOutDatumNone
c370cb076dc5893f548a8acf42fd507f4c21d667c036602e9092ca38f5fafbc5     1        9799824379 lovelace + TxOutDatumNone
```

  * #### 5.2 Mint the token 

```bash
# Encoded the token name in hex format
ASSET_NAME=$(echo -n "MARS" | xxd -ps | tr -d '\n')
TOKEN_SUPPLY=1000000000
POLICY_ID=$(cat policy/mars.policyid)
MINT_ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr)
UTXO_IN=0be38e65bec5eebde5ca3308a75e5c8adcd1b184534b445c984185d7d6f9aee5#1
UTXO_IN_AMOUNT=100000000
# The tokens need to have a minimum value
# I set it to 2 ADA but it depend of bundle inside the txout (bundle = "AMOUNT of token policyID.AssetName")
MIN_ADA_VALUE=2000000

# Build the transaction 
cardano-cli transaction build \
--tx-in $UTXO_IN \
--tx-out $MINT_ADDR+$MIN_ADA_VALUE+"$TOKEN_SUPPLY $POLICY_ID.$ASSET_NAME" \
--change-address $MINT_ADDR \
--mint "$TOKEN_SUPPLY $POLICY_ID.$ASSET_NAME" \
--mint-script-file policy/marsToken.script \
--testnet-magic 1 \
--witness-override 2 \
--out-file tx-files/mintTX.raw

cardano-cli transaction sign \
--tx-body-file tx-files/mintTX.raw \
--signing-key-file priv/MARS/marsToken.skey \
--signing-key-file /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr.skey \
--out-file tx-files/mintTX.signed 
```

```bash
Estimated transaction fee: Lovelace 177865
```

### Submint the transaction

```bash
cardano-cli transaction submit \
--tx-file tx-files/mintTX.signed \
--testnet-magic 1
```

```bash
Transaction successfully submitted.
```

### Check UTXOs from the address

```bash
MINT_ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr)

cardano-cli query utxo \
--address $MINT_ADDR \
--testnet-magic 1
```

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
37906b562ec69696d58e57eaa06b73ef32a5a427d3f60a2389379e27585de5f9     0        97822135 lovelace + TxOutDatumNone
37906b562ec69696d58e57eaa06b73ef32a5a427d3f60a2389379e27585de5f9     1        2000000 lovelace + 1000000000 4fd78aae5e7643885c5f0c63d26641e2e05870d8544af7c6c239ff46.4d415253 + TxOutDatumNone
8d4f02739f9f921e7d50cd7765527242a1ca5bd37dc479a550af99463290d89f     1        100000000 lovelace + TxOutDatumNone
c370cb076dc5893f548a8acf42fd507f4c21d667c036602e9092ca38f5fafbc5     1        9799824379 lovelace + TxOutDatumNone
```

# How to send fungible token

```bash
# Encoded the token name in hex format
ASSET_NAME=$(echo -n "MARS" | xxd -ps | tr -d '\n')
TOKEN_SUPPLY=1000000000
POLICY_ID=$(cat policy/mars.policyid)
SENDER_ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr)
RECEIVER_ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Djessy/djessy.addr)
UTXO_IN_1=37906b562ec69696d58e57eaa06b73ef32a5a427d3f60a2389379e27585de5f9#1
UTXO_IN_2=8d4f02739f9f921e7d50cd7765527242a1ca5bd37dc479a550af99463290d89f#1
AMOUNT_TOKEN_TO_SEND=50000000
CHANGE_TOKEN=$((1000000000 - $AMOUNT_TOKEN_TO_SEND))
MIN_ADA_VALUE=2000000

# Build the transaction 
cardano-cli transaction build \
--tx-in $UTXO_IN_1 \
--tx-in $UTXO_IN_2 \
--tx-out $RECEIVER_ADDR+$MIN_ADA_VALUE+"$AMOUNT_TOKEN_TO_SEND $POLICY_ID.$ASSET_NAME" \
--tx-out $SENDER_ADDR+$MIN_ADA_VALUE+"$CHANGE_TOKEN $POLICY_ID.$ASSET_NAME" \
--change-address $SENDER_ADDR \
--testnet-magic 1 \
--out-file tx-files/sendToken.raw

# Sign the transaction
cardano-cli transaction sign \
--tx-body-file tx-files/sendToken.raw \
--signing-key-file /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr.skey \
--out-file tx-files/sendToken.signed

# Submit the transaction
cardano-cli transaction submit \
--tx-file tx-files/sendToken.signed \
--testnet-magic 1
```

```bash
Estimated transaction fee: Lovelace 179449
Transaction successfully submitted.
```

### Check UTXOs from the address

```bash
MINT_ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Djessy/djessy.addr)

cardano-cli query utxo \
--address $MINT_ADDR \
--testnet-magic 1
```

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
b74048f802c6001998cc544583d9473cd7e58adb79eac49f4da000e03c342a8b     0        9998983630 lovelace + TxOutDatumNone
ca367e87ed8750475279267363192e8e55ec3febb6b54e7c8ec2b96e296155f2     1        2000000 lovelace + 50000000 4fd78aae5e7643885c5f0c63d26641e2e05870d8544af7c6c239ff46.4d415253 + TxOutDatumNone
```

# How to burn fungible token

To burn tokens, you have to provide the signing key of the address where the tokens is locked,  
and the corresponding signing key from the policy script.


### 1) Get the address with the token is locked 

```bash
MINT_ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr)

cardano-cli query utxo \
--address $MINT_ADDR \
--testnet-magic 1
```

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
37906b562ec69696d58e57eaa06b73ef32a5a427d3f60a2389379e27585de5f9     0        97822135 lovelace + TxOutDatumNone
c370cb076dc5893f548a8acf42fd507f4c21d667c036602e9092ca38f5fafbc5     1        9799824379 lovelace + TxOutDatumNone
ca367e87ed8750475279267363192e8e55ec3febb6b54e7c8ec2b96e296155f2     0        97820551 lovelace + TxOutDatumNone
ca367e87ed8750475279267363192e8e55ec3febb6b54e7c8ec2b96e296155f2     2        2000000 lovelace + 950000000 4fd78aae5e7643885c5f0c63d26641e2e05870d8544af7c6c239ff46.4d415253 + TxOutDatumNone
```

### 2) Burn the tokens at the address

```bash
# Encoded the token name in hex format
ASSET_NAME=$(echo -n "MARS" | xxd -ps | tr -d '\n')
TOKEN_SUPPLY=1000000000
POLICY_ID=$(cat policy/mars.policyid)
ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr)
UTXO_IN_1=ca367e87ed8750475279267363192e8e55ec3febb6b54e7c8ec2b96e296155f2#2
UTXO_IN_2=ca367e87ed8750475279267363192e8e55ec3febb6b54e7c8ec2b96e296155f2#0
AMOUNT_TOKEN_TO_BURN=900000000
AMOUNT_TOKEN=950000000
OUTPUT=$(($AMOUNT_TOKEN - $AMOUNT_TOKEN_TO_BURN))
# The token need to have a minimum value to be mint and sent
MIN_ADA_VALUE=2000000

# Build the transaction 
cardano-cli transaction build \
--tx-in $UTXO_IN_1 \
--tx-in $UTXO_IN_2 \
--tx-out $ADDR+$MIN_ADA_VALUE+"$OUTPUT $POLICY_ID.$ASSET_NAME" \
--change-address $ADDR \
--mint "-$AMOUNT_TOKEN_TO_BURN $POLICY_ID.$ASSET_NAME" \
--mint-script-file policy/marsToken.script \
--testnet-magic 1 \
--witness-override 2 \
--out-file tx-files/burnToken.raw

# Sign the transaction
cardano-cli transaction sign \
--tx-body-file tx-files/burnToken.raw \
--signing-key-file /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr.skey \
--signing-key-file priv/MARS/marsToken.skey \
--testnet-magic 1 \
--out-file tx-files/burnToken.signed

# Submit the transaction
cardano-cli transaction submit \
--tx-file tx-files/burnToken.signed \
--testnet-magic 1
```

```bash
Estimated transaction fee: Lovelace 179449
Transaction successfully submitted.
```

### Check UTXOs from the address

```bash
MINT_ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Drake/drake.addr)

cardano-cli query utxo \
--address $MINT_ADDR \
--testnet-magic 1
```

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
37906b562ec69696d58e57eaa06b73ef32a5a427d3f60a2389379e27585de5f9     0        97822135 lovelace + TxOutDatumNone
75a02329eb8ba228ca15dfe4d59bedf1a18c6cb196aa06f35839232790b9b8d3     0        97641102 lovelace + TxOutDatumNone
75a02329eb8ba228ca15dfe4d59bedf1a18c6cb196aa06f35839232790b9b8d3     1        2000000 lovelace + 50000000 4fd78aae5e7643885c5f0c63d26641e2e05870d8544af7c6c239ff46.4d415253 + TxOutDatumNone
c370cb076dc5893f548a8acf42fd507f4c21d667c036602e9092ca38f5fafbc5     1        9799824379 lovelace + TxOutDatumNone
```

