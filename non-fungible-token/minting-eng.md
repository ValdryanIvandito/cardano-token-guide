# Introduction

This documentation provides a simplified, step-by-step guide to minting Non Fungible Token on Cardano using Cardano-CLI, follow the steps below.

# Step by Step

## Step-1 Generate Wallet Address

If you haven't generated a wallet address, you should follow this [documentation](https://github.com/ValdryanIvandito/cardano-basic-transaction-guide/blob/main/generate-wallet-address-eng.md) first.

## Step-2 Initiate Cardano Network (Optional)

**_Hint: If you have choosen the network, you can skip this step_**

```bash
network="testnet-magic 1"
```

_or_

```bash
network="testnet-magic 2"
```

_or_

```bash
network="mainnet"
```

**The following is a table regarding the network on Cardano**
| cli Parameter | Network Name |
|---------|---------|
| testnet-magic 1 | Preprod |
| testnet-magic 2 | Preview |
| mainnet | Mainnet |

## Step-3 Display Information About the Wallet Address UTxO

**_Hint: Assuming you already have a Wallet Address Serving as Token Address with a balance_**

### Go to Wallet Directory

```bash
cd myWallet
```

**_Hint: If you already in myWallet directory, you can skip this step_**

### Display The Wallet Address

```bash
tokenAddress=$(cat payment.addr)
echo $tokenAddress
```

### Display The UTxO Information

```bash
cardano-cli query utxo \
--address $tokenAddress \
--$network
```

### Initiate TxHash, TxIx, and Balance

```bash
utxo="COPY THE TX-HASH HERE#COPY THE TX-IX NUMBER HERE"
balance="COPY THE BALANCE HERE"
```

**_Note: TxHash and TxIx are restricted between '#'_**

## Step-4 Generate Token Policy

**_Hint:_**
**_Since NFTs are likely to be traded or sold, they should follow a more strict policy. Most of the time, a value is defined by the (artificial) scarcity of an asset._**

**_You can regulate such factors with [multi-signature scripts](https://github.com/IntersectMBO/cardano-node/blob/c6b574229f76627a058a7e559599d2fc3f40575d/doc/reference/simple-scripts.md)._**

**_For this guide, we will choose the following constraints:_**
**_1. There should be only one defined signature allowed to mint (or burn) the NFT._**
**_2. The signature will expire in 10000 slots from now._**

### Create Directories

```bash
mkdir -p nft/policy
```

### Create Policy Verification and Signing Key

```bash
cardano-cli address key-gen \
--verification-key-file nft/policy/policy.vkey \
--signing-key-file nft/policy/policy.skey
```

### Determines the Expiration Time

```bash
slotNumber=$(expr $(cardano-cli query tip --$network | jq .slot?) + 10000)
```

**_Note: The signature will expire in 10000 slots from now to leave the room if we screw something up._**

### Create Policy Script File

```bash
touch nft/policy/policy.script && echo "" > nft/policy/policy.script

echo "{" >> nft/policy/policy.script
echo "  \"type\": \"all\"," >> nft/policy/policy.script
echo "  \"scripts\":" >> nft/policy/policy.script
echo "  [" >> nft/policy/policy.script
echo "   {" >> nft/policy/policy.script
echo "     \"type\": \"before\"," >> nft/policy/policy.script
echo "     \"slot\": $slotNumber" >> nft/policy/policy.script
echo "   }," >> nft/policy/policy.script
echo "   {" >> nft/policy/policy.script
echo "     \"type\": \"sig\"," >> nft/policy/policy.script
echo "     \"keyHash\": \"$(cardano-cli address key-hash --payment-verification-key-file nft/policy/policy.vkey)\"" >> nft/policy/policy.script
echo "   }" >> nft/policy/policy.script
echo "  ]" >> nft/policy/policy.script
echo "}" >> nft/policy/policy.script
```

### Create Policy-ID

```bash
cardano-cli transaction policyid \
--script-file nft/policy/policy.script > nft/policy/policyID
```

## Step-5 Generate NFT Metadata

### Initiate Token Parameter

```bash
policyId=$(cat nft/policy/policyID)
nftName="MY-NFT"
nftNameHex=$(echo -n $nftName | xxd -b -ps -c 80 | tr -d '\n')
nftAmount="1"
adaAmount="1400000"
description="This is a trial NFT, it used for educational purposes."
version="1.0"
```

### Load Token Image to IPFS

**Instructions:**

1. Prepare the image.
2. Go to [Pinata Cloud](https://app.pinata.cloud/signin).
3. Load the image.
4. Get CID, for example: _QmRiAgCf9J3NaF5u2BG4jqZF981m5hTdA6jq4swHdgoVcA_
5. Copy and paste the CID into the icon parameter:

```bash
ipfs="ipfs://Copy CID here"
mediaType="image/png"
```

### Create Metadata JSON

```bash
echo "{" >> nft/metadata.json
echo "  \"721\": {" >> nft/metadata.json
echo "    \"$(cat nft/policy/policyID)\": {" >> nft/metadata.json
echo "      \"$(echo $nftName)\": {" >> nft/metadata.json
echo "        \"name\": \"$(echo $nftName)\"," >> nft/metadata.json
echo "        \"image\": \"$(echo $ipfs)\"," >> nft/metadata.json
echo "        \"mediaType\": \"$(echo $mediaType)\"," >> nft/metadata.json
echo "        \"description\": \"$(echo $description)\"," >> nft/metadata.json
echo "        \"files\": [{" >> nft/metadata.json
echo "          \"name\": \"$(echo $nftName)\"," >> nft/metadata.json
echo "          \"mediaType\": \"$(echo $mediaType)\"," >> nft/metadata.json
echo "          \"src\": \"$(echo $ipfs)\"" >> nft/metadata.json
echo "        }]" >> nft/metadata.json
echo "      }" >> nft/metadata.json
echo "    }," >> nft/metadata.json
echo "    \"version\": \"$(echo $version)\"" >> nft/metadata.json
echo "  }" >> nft/metadata.json
echo "}" >> nft/metadata.json
```

## Step-6 Build Transaction

```bash
cardano-cli transaction build \
--$network \
--babbage-era \
--tx-in $utxo \
--tx-out $tokenAddress+$adaAmount+"$nftAmount $policyId.$nftNameHex" \
--change-address $tokenAddress \
--mint="$nftAmount $policyId.$nftNameHex" \
--minting-script-file nft/policy/policy.script \
--metadata-json-file nft/metadata.json  \
--invalid-hereafter $slotNumber \
--witness-override 2 \
--out-file nft/mint.raw 
```

## Step-7 Sign Transaction

```bash
cardano-cli transaction sign \
--$network \
--signing-key-file payment.skey \
--signing-key-file nft/policy/policy.skey \
--tx-body-file nft/mint.raw \
--out-file nft/mint.signed
```

## Step-8 Submit Transaction

```bash
cardano-cli transaction submit \
--$network \
--tx-file nft/mint.signed
```

## Step-9 Display The UTxO Information

```bash
cardano-cli query utxo \
--address $tokenAddress \
--$network
```

**Example Result:**

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
194593e5b7fac4b5d81adbb57407c96faeb7dbbcb222f882d2e21ced9e5a9ff1     0        9999813643 lovelace + 1000000000 e5444aaa4f3b82411dd1017a8d28324485550c14a35aa02a480586d6.4d59544f4b454e + TxOutDatumNone
```

# Demo

The following is a video recorded by the Indonesian Cardano Developers Community where I demonstrated the steps above. Watch the recorded video at timestamp **_1:27:27_**, here is the [link](https://youtu.be/03hXLZ_07N0?list=PLUj8499OocHiL8gXPv8wMlLW-zIcyYdrQ).

# References

[Cardano Developer Portal: Minting Native Assets](https://developers.cardano.org/docs/native-tokens/minting)

[CIP-35 On-Chain Token Metadata Standard](https://github.com/cardano-foundation/CIPs/blob/1d9fbd0e29f07b931bf1524c7aed6635d478cd75/CIP-0035/CIP-0035.md)
