# How to create a non fungible token



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

cat $SCRIPT_PATH | 
```

### 4) Create the policy ID

```bash
cardano-cli transaction policyid \
--script-file policy/nft.script \
> policy/nft.policyid

cat policy/nft.policyid
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

### 5) Mint the token


  * #### 5.1 Get the UTXO from the address you want to mint the token 

```bash
cardano-cli query utxo \
--address $(cat /users/$(whoami)/testnet/priv/wallet/Djessy/djessy.addr) \
--testnet-magic 1 
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

### Check the address where the NFT was minted 

```bash
cardano-cli query utxo \
--address $(cat /users/$(whoami)/testnet/priv/wallet/Djessy/djessy.addr) \
--testnet-magic 1 
```

```python
!jupytext --to markdown how-to-create-a-non-fungible-token.ipynb
```

### Get the on-chain data of the asset

```python
from cardano_explorer import blockfrost_api as bf

preprod= bf.Auth("preprod")
# On blockfrost the assetis the policyID + Asset Name
asset=c8d961ee981b4c7944b9b82b48fd609108fcf63dfc6196c6536d99a04d61727369616e
preprod.specific_asset("c8d961ee981b4c7944b9b82b48fd609108fcf63dfc6196c6536d99a04d61727369616e")
```

```bash
{'asset': 'c8d961ee981b4c7944b9b82b48fd609108fcf63dfc6196c6536d99a04d61727369616e',
 'policy_id': 'c8d961ee981b4c7944b9b82b48fd609108fcf63dfc6196c6536d99a0',
 'asset_name': '4d61727369616e',
 'fingerprint': 'asset162z4yxy28ydzmnm0ns7k9snqr9vp6wm2lcnl0f',
 'quantity': '1',
 'initial_mint_tx_hash': '0c7863e82595e534e5e66cb96aeffe6f62db88bd9d9dc4a3335e5c6571d791b8',
 'mint_or_burn_count': 1,
 'onchain_metadata': {'image': 'ipfs://QmbdvDECDxrZfcGoM6tzQLF68VgiKiCB2pNy8ioKZ4rTJS',
  'descripption': 'First man on mars generate with DALL-E-2 AI'},
 'metadata': None}
```
