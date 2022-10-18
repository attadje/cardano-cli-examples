# How to create a Non Fungible Token



Native tokens on cardano are executed directly on-chain,  
unlike the ERC-20 tokens for example, which are executed by means of smart contracts on the Ethereum blockchain.

Non fungible token property:  
  * Are unique and finite in quantity  
  * Not equivalent to each other  


### 1) Defines the token architecture


Token name: `Marsian`  
Token supply: `1`   
Token time constraint: `1200 slots after minting` (1 slot = 1 seconde)   
Who can mint: `One address`  


### 2) Create a verification key and signing key for mint and burn the token

```bash
cardano-cli address key-gen \
--verification-key-file priv/NFT/nft.vkey \
--signing-key-file priv/NFT/nft.skey
```

### 3) Create the policy script

```bash
NFT_VKEY=$(cardano-cli address key-hash --payment-verification-key-file priv/NFT/nft.vkey)
SCRIPT_PATH=policy/nft.script
# Time constraint is set to 1200 slot after the current slot 
TIME_CONSTRAINT=$(($(cardano-cli query tip --testnet-magic 1 | jq .slot) + 1200))

# Monetary script
echo "{" >> $SCRIPT_PATH
echo "  \"type\": \"all\"," >> $SCRIPT_PATH
echo "  \"scripts\":" >> $SCRIPT_PATH
echo "  [" >> $SCRIPT_PATH
echo "    {" >> $SCRIPT_PATH
echo "      \"type\":\"before\"," >> $SCRIPT_PATH
echo "      \"slot\":$TIME_CONSTRAINT" >> $SCRIPT_PATH
echo "    }," >> $SCRIPT_PATH
echo "    {" >> $SCRIPT_PATH
echo "      \"type\": \"sig\"," >> $SCRIPT_PATH
echo "      \"keyHash\": \"$NFT_VKEY\"" >> $SCRIPT_PATH
echo "    }" >> $SCRIPT_PATH
echo "  ]" >> $SCRIPT_PATH
echo "}" >> $SCRIPT_PATH

cat $SCRIPT_PATH  
```

```bash
{
  "type": "all",
  "scripts":
  [
    {
      "type":"before",
      "slot":10442040
    },
    {
      "type": "sig",
      "keyHash": "13ba8318391c81b4b486ec3e718c6357c289dc73ea8c2ec59933e31d"
    }
  ]
}
```

### 4) Create the policy ID

```bash
cardano-cli transaction policyid \
--script-file policy/nft.script \
> policy/nft.policyid

cat policy/nft.policyid
```

```bash
c8d961ee981b4c7944b9b82b48fd609108fcf63dfc6196c6536d99a0
```

### 5) Create the metadata

Look into the [NFT Metadata Standard (CIP 25)](https://cips.cardano.org/cips/cip25/).

```bash
NFT_ID=721
POLICY_ID=$(cat policy/nft.policyid)
ASSET_NAME="Marsian"
IMAGE="ipfs://QmbdvDECDxrZfcGoM6tzQLF68VgiKiCB2pNy8ioKZ4rTJS"
DESCRIPTION="First man on mars generate with DALL-E-2 AI"
NFT_METADATA_PATH=metadata/nft.metadata

# Basic metadata schemme 
echo "{" >> $NFT_METADATA_PATH
echo "  \"$NFT_ID\": {" >> $NFT_METADATA_PATH
echo "    \"$POLICY_ID\": {" >> $NFT_METADATA_PATH
echo "     \"$ASSET_NAME\": {" >> $NFT_METADATA_PATH
echo "     \"descripption\": \"$DESCRIPTION\"," >> $NFT_METADATA_PATH
echo "     \"image\": \"$IMAGE\"" >> $NFT_METADATA_PATH
echo "   }" >> $NFT_METADATA_PATH
echo "  }" >> $NFT_METADATA_PATH
echo " }" >> $NFT_METADATA_PATH
echo "}" >> $NFT_METADATA_PATH

cat $NFT_METADATA_PATH
```

```bash

{
  "721": {
    "c8d961ee981b4c7944b9b82b48fd609108fcf63dfc6196c6536d99a0": {
     "Marsian": {
     "descripption": "First man on mars generate with DALL-E-2 AI",
     "image": "ipfs://QmbdvDECDxrZfcGoM6tzQLF68VgiKiCB2pNy8ioKZ4rTJS"
   }
  }
 }
}
```

### 5) Mint the token


  * #### 5.1 Get the UTXO from the address you want to mint the token 

```bash
cardano-cli query utxo \
--address $(cat /users/$(whoami)/testnet/priv/wallet/Djessy/djessy.addr) \
--testnet-magic 1 
```

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
b74048f802c6001998cc544583d9473cd7e58adb79eac49f4da000e03c342a8b     0        9998983630 lovelace + TxOutDatumNone
ca367e87ed8750475279267363192e8e55ec3febb6b54e7c8ec2b96e296155f2     1        2000000 lovelace + 50000000 4fd78aae5e7643885c5f0c63d26641e2e05870d8544af7c6c239ff46.4d415253 + TxOutDatumNone
```

  * #### 5.2 Mint the token 

```bash
# Encoded the token name in hex format
ASSET_NAME=$(echo -n "Marsian" | xxd -ps | tr -d '\n')
TOKEN_SUPPLY=1
POLICY_ID=$(cat policy/nft.policyid)
MINT_ADDR=$(cat /users/$(whoami)/testnet/priv/wallet/Djessy/djessy.addr)
UTXO_IN=b74048f802c6001998cc544583d9473cd7e58adb79eac49f4da000e03c342a8b#0
UTXO_IN_AMOUNT=9998983630
MIN_ADA_VALUE=2000000
INVALID_AFTER=

# Build the transaction 
cardano-cli transaction build \
--tx-in $UTXO_IN \
--tx-out $MINT_ADDR+$MIN_ADA_VALUE+"$TOKEN_SUPPLY $POLICY_ID.$ASSET_NAME" \
--change-address $MINT_ADDR \
--mint "$TOKEN_SUPPLY $POLICY_ID.$ASSET_NAME" \
--minting-script-file policy/nft.script \
--metadata-json-file metadata/nft.metadata \
--invalid-hereafter 10442040 \
--witness-override 2 \
--testnet-magic 1 \
--out-file tx-files/nftMintTx.raw

# Sign the transaction
cardano-cli transaction sign \
--tx-body-file tx-files/nftMintTx.raw \
--signing-key-file /users/$(whoami)/testnet/priv/wallet/Djessy/djessy.addr.skey \
--signing-key-file priv/NFT/nft.skey \
--out-file tx-files/nftMintTx.signed

# Submi the transaction
cardano-cli transaction submit \
--tx-file tx-files/nftMintTx.signed \
--testnet-magic 1
```

```bash
Estimated transaction fee: Lovelace 186049
Transaction successfully submitted.
```

### Check the address where the NFT was minted 

```bash
cardano-cli query utxo \
--address $(cat /users/$(whoami)/testnet/priv/wallet/Djessy/djessy.addr) \
--testnet-magic 1 
```

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
0c7863e82595e534e5e66cb96aeffe6f62db88bd9d9dc4a3335e5c6571d791b8     0        9996797581 lovelace + TxOutDatumNone
0c7863e82595e534e5e66cb96aeffe6f62db88bd9d9dc4a3335e5c6571d791b8     1        2000000 lovelace + 1 c8d961ee981b4c7944b9b82b48fd609108fcf63dfc6196c6536d99a0.4d61727369616e + TxOutDatumNone
ca367e87ed8750475279267363192e8e55ec3febb6b54e7c8ec2b96e296155f2     1        2000000 lovelace + 50000000 4fd78aae5e7643885c5f0c63d26641e2e05870d8544af7c6c239ff46.4d415253 + TxOutDatumNone
```
