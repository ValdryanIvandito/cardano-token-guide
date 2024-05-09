# Introduction

This documentation provides a simplified, step-by-step guide to minting and burning fungible token on Cardano, follow the steps below.

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

**Example Result:**

```bash
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
a0d2972de62348694d0677a7d6137cde6653d33d9c45cc3a3deb7f9749e34af4     0        10000000000 lovelace + TxOutDatumNone
```

### Initiate TxHash and TxIx

```bash
utxo="COPY THE TX-HASH HERE#COPY THE TX-IX NUMBER HERE"
```

**_Note: TxHash and TxIx are restricted between '#'_**

## Step-4 Generate Policy ID

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

## Step-5 Initiate Token

```bash
policyid=$(cat ft/policyID)
ticker="TOKEN"
hexTicker=$(echo -n $ticker | xxd -ps | tr -d '\n')
supply=1000000
```

## Step-6 Load Token Image to IPFS

**Intructions:**

1. Prepare the image
2. Go to [Pinata Cloud](https://app.pinata.cloud/signin)
3. Load the image
4. Get CID, for example:

```bash
QmRiAgCf9J3NaF5u2BG4jqZF981m5hTdA6jq4swHdgoVcA
```

5. Check to:

```bash
https://ipfs.io/ipfs/QmRiAgCf9J3NaF5u2BG4jqZF981m5hTdA6jq4swHdgoVcA
```

**_Hint: Wait for the process, sometimes it takes a long time._**

## Step-7 Generate Protocol JSON

```bash
cardano-cli query protocol-parameters \
--$network \
--out-file ft/protocol.json
```

## Step-8 Build Transaction

```bash
cardano-cli transaction build \
--babbage-era \
--$network \
--tx-in $utxo \
--tx-out $tokenAddress+"1500000 + $supply $policyid.$hexTicker" \
--mint "$supply $policyid.$hexTicker" \
--mint-script-file ft/policy.script \
--change-address $tokenAddress \
--protocol-params-file ft/protocol.json \
--metadata-json-file ft/metadata.json  \
--out-file ft/minting.draft
```

## Step-9 Sign Transaction

```bash
cardano-cli transaction sign \
--signing-key-file payment.skey \
--$network \
--tx-body-file ft/matx.draft \
--out-file ft/minting.signed
```

## Step-10 Submit Transaction

```bash
cardano-cli transaction submit \
--tx-file ft/minting.signed \
--$network
```

# Demo

# References
