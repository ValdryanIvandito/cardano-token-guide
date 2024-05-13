# Introduction

This documentation provides a simplified, step-by-step guide to minting fungible token on Cardano, follow the steps below.

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

## Step-4 Create Token Policy

### Create Directory for Policy Script

```bash
mkdir ft
```

### Create Policy Script File

```bash
touch ft/policy.script && echo "" > ft/policy.script
echo "{" > ft/policy.script
echo "  \"keyHash\": \"$(cardano-cli address key-hash --payment-verification-key-file payment.vkey)\"," >> ft/policy.script
echo "  \"type\": \"sig\"" >> ft/policy.script
echo "}" >> ft/policy.script
```

### Create Policy ID

```bash
cardano-cli transaction policyid \
--script-file ft/policy.script > ft/policyID
```

### Initiate Token Parameter

```bash
policyId=$(cat ft/policyID)
name="MYTOKEN"
hexName=$(echo -n $name | xxd -ps | tr -d '\n')
ticker="MTKN"
desc="This is experimental token"
mintSupply=1000000000
decimals=6
version="1.0"
```

## Step-5 Load Token Image to IPFS

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

## Step-6 Create Metadata JSON

```bash
echo "{" >> ft/metadata.json
echo "  \"20\": {" >> ft/metadata.json
echo "    \"$(echo $policyId)\": {" >> ft/metadata.json
echo "      \"$(echo $hexName)\": {" >> ft/metadata.json
echo "        \"ticker\": \"$(echo $ticker)\"," >> ft/metadata.json
echo "        \"name\": \"$(echo $name)\"," >> ft/metadata.json
echo "        \"desc\": \"$(echo $desc)\"," >> ft/metadata.json
echo "        \"description\": \"$(echo $desc)\"," >> ft/metadata.json
echo "        \"icon\": \"$(echo $ipfs)\"," >> ft/metadata.json
echo "        \"image\": \"$(echo $ipfs)\"," >> ft/metadata.json
echo "        \"mediaType\": \"$(echo $mediaType)\"," >> ft/metadata.json
echo "        \"decimals\": \"$(echo $decimals)\"," >> ft/metadata.json
echo "        \"files\": [{" >> ft/metadata.json
echo "          \"name\": \"$(echo $name)\"," >> ft/metadata.json
echo "          \"mediaType\": \"$(echo $mediaType)\"," >> ft/metadata.json
echo "          \"src\": \"$(echo $ipfs)\"" >> ft/metadata.json
echo "        }]" >> ft/metadata.json
echo "      }" >> ft/metadata.json
echo "    }," >> ft/metadata.json
echo "    \"version\": \"$(echo $version)\"" >> ft/metadata.json
echo "  }," >> ft/metadata.json
echo "  \"721\": {" >> ft/metadata.json
echo "    \"$(echo $policyId)\": {" >> ft/metadata.json
echo "      \"$(echo $name)\": {" >> ft/metadata.json
echo "        \"name\": \"$(echo $name)\"," >> ft/metadata.json
echo "        \"ticker\": \"$(echo $ticker)\"," >> ft/metadata.json
echo "        \"image\": \"$(echo $ipfs)\"," >> ft/metadata.json
echo "        \"mediaType\": \"$(echo $mediaType)\"," >> ft/metadata.json
echo "        \"description\": \"$(echo $desc)\"," >> ft/metadata.json
echo "        \"files\": [{" >> ft/metadata.json
echo "          \"name\": \"$(echo $name)\"," >> ft/metadata.json
echo "          \"mediaType\": \"$(echo $mediaType)\"," >> ft/metadata.json
echo "          \"src\": \"$(echo $ipfs)\"" >> ft/metadata.json
echo "        }]" >> ft/metadata.json
echo "      }" >> ft/metadata.json
echo "    }," >> ft/metadata.json
echo "    \"version\": \"$(echo $version)\"" >> ft/metadata.json
echo "  }" >> ft/metadata.json
echo "}" >> ft/metadata.json
```

```bash
echo "{" >> ft/metadata.json
echo "  \"20\": {" >> ft/metadata.json
echo "    \"$(echo $policyId)\": {" >> ft/metadata.json
echo "      \"$(echo $hexName)\": {" >> ft/metadata.json
echo "        \"ticker\": \"$(echo $ticker)\"," >> ft/metadata.json
echo "        \"name\": \"$(echo $name)\"," >> ft/metadata.json
echo "        \"desc\": \"$(echo $desc)\"," >> ft/metadata.json
echo "        \"description\": \"$(echo $desc)\"," >> ft/metadata.json
echo "        \"icon\": \"$(echo $ipfs)\"," >> ft/metadata.json
echo "        \"image\": \"$(echo $ipfs)\"," >> ft/metadata.json
echo "        \"mediaType\": \"$(echo $mediaType)\"," >> ft/metadata.json
echo "        \"decimals\": \"$(echo $decimals)\"," >> ft/metadata.json
echo "        \"files\": [{" >> ft/metadata.json
echo "          \"name\": \"$(echo $name)\"," >> ft/metadata.json
echo "          \"mediaType\": \"$(echo $mediaType)\"," >> ft/metadata.json
echo "          \"src\": \"$(echo $ipfs)\"" >> ft/metadata.json
echo "        }]" >> ft/metadata.json
echo "      }" >> ft/metadata.json
echo "    }," >> ft/metadata.json
echo "    \"version\": \"$(echo $version)\"" >> ft/metadata.json
echo "  }" >> ft/metadata.json
echo "}" >> ft/metadata.json
```

```bash
echo "{" >> ft/metadata.json
echo "  \"20\": {" >> ft/metadata.json
echo "    \"$(echo $policyId)\": {" >> ft/metadata.json
echo "      \"$(echo $hexName)\": {" >> ft/metadata.json
echo "        \"ticker\": \"$(echo $ticker)\"," >> ft/metadata.json
echo "        \"desc\": \"$(echo $desc)\"," >> ft/metadata.json
echo "        \"icon\": \"$(echo $ipfs)\"," >> ft/metadata.json
echo "        \"decimals\": \"$(echo $decimals)\"," >> ft/metadata.json
echo "        \"version\": \"$(echo $version)\"" >> ft/metadata.json
echo "      }" >> ft/metadata.json
echo "    }" >> ft/metadata.json
echo "  }" >> ft/metadata.json
echo "}" >> ft/metadata.json
```

```bash
echo "{" >> ft/metadata.json
echo "  \"20\": {" >> ft/metadata.json
echo "    \"$(echo $policyId)\": {" >> ft/metadata.json
echo "      \"$(echo $hexName)\": {" >> ft/metadata.json
echo "        \"ticker\": \"$(echo $ticker)\"," >> ft/metadata.json
echo "        \"name\": \"$(echo $name)\"," >> ft/metadata.json
echo "        \"desc\": \"$(echo $desc)\"," >> ft/metadata.json
echo "        \"description\": \"$(echo $desc)\"," >> ft/metadata.json
echo "        \"icon\": \"$(echo $ipfs)\"," >> ft/metadata.json
echo "        \"image\": \"$(echo $ipfs)\"," >> ft/metadata.json
echo "        \"mediaType\": \"$(echo $mediaType)\"," >> ft/metadata.json
echo "        \"version\": \"$(echo $version)\"" >> ft/metadata.json
echo "      }" >> ft/metadata.json
echo "    }" >> ft/metadata.json
echo "  }" >> ft/metadata.json
echo "}" >> ft/metadata.json
```

## Step-7 Create Protocol JSON

```bash
cardano-cli query protocol-parameters \
--$network \
--out-file ft/protocol.json
```

## Step-8 Manually Calculate Fee

```bash
cardano-cli transaction build-raw \
--fee 0 \
--tx-in $utxo \
--tx-out $tokenAddress+$balance+"$mintSupply $policyId.$hexName" \
--mint "$mintSupply $policyId.$hexName" \
--minting-script-file ft/policy.script \
--metadata-json-file ft/metadata.json  \
--out-file ft/mint.draft

fee=$(cardano-cli transaction calculate-min-fee --tx-body-file ft/mint.draft --tx-in-count 1 --tx-out-count 1 --witness-count 1 --$network --protocol-params-file ft/protocol.json | cut -d " " -f1)
echo $fee

balance=$(expr $balance - $fee)
echo $balance
```

## Step-9 Build Transaction

```bash
cardano-cli transaction build-raw \
--fee $fee \
--tx-in $utxo \
--tx-out $tokenAddress+$balance+"$mintSupply $policyId.$hexName" \
--mint "$mintSupply $policyId.$hexName" \
--mint-script-file ft/policy.script \
--protocol-params-file ft/protocol.json \
--metadata-json-file ft/metadata.json  \
--out-file ft/mint.raw
```

## Step-10 Sign Transaction

```bash
cardano-cli transaction sign \
--signing-key-file payment.skey \
--$network \
--tx-body-file ft/mint.raw \
--out-file ft/mint.signed
```

## Step-11 Submit Transaction

```bash
cardano-cli transaction submit \
--tx-file ft/mint.signed \
--$network
```

## Step-12 Display The UTxO Information

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
