# Introduction

This documentation provides a simplified, step-by-step guide to burning fungible token on Cardano, follow the steps below.

# Step by Step

## Step-1 Minting Fungible Token

First, you must mint the fungible token. The following is the [documentation]() on how to mint fungible token.

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

### Initiate TxHash and TxIx

```bash
utxo="COPY THE TX-HASH HERE#COPY THE TX-IX NUMBER HERE"
adaBalance="COPY THE ADA BALANCE HERE"
tokenBalance="COPY THE TOKEN BALANCE HERE"
```

**_Note: TxHash and TxIx are restricted between '#'_**

## Step-4 Initiate the output: Recipient Address and Token Amount to be Sent

```bash
policyId=$(cat ft/policyID)
name="MYTOKEN"
hexName=$(echo -n $name | xxd -ps | tr -d '\n')
recipientAddress="COPY THE RECIPIENT ADDRESS HERE"
adaSendAmount="AMOUNT IN LOVELACE TO BE SENT"
tokenSendAmount="TOKEN AMOUNT TO BE SENT"
adaAmount=$(expr $adaBalance - $adaSendAmount)
tokenAmount=$(expr $tokenBalance - $tokenSendAmount)
```

**_Note: The minimum amount of ADA is 1500000 lovelace that must be sent when sending tokens_**

## Step-5 Manually Calculate Fee

```bash
cardano-cli transaction build-raw \
--fee 0 \
--tx-in $utxo \
--tx-out $recipientAddress+$adaSendAmount+"$tokenSendAmount $policyId.$hexName" \
--tx-out $tokenAddress+$adaAmount+"$tokenAmount $policyId.$hexName" \
--out-file ft/send.draft

fee=$(cardano-cli transaction calculate-min-fee --tx-body-file ft/send.draft --tx-in-count 1 --tx-out-count 2 --witness-count 1 --$network --protocol-params-file ft/protocol.json | cut -d " " -f1)
echo $fee

adaAmount=$(expr $adaAmount - $fee)
echo $adaAmount
```

## Step-5 Build Transaction

```bash
cardano-cli transaction build-raw \
--fee $fee \
--tx-in $utxo \
--tx-out $recipientAddress+$adaSendAmount+"$tokenSendAmount $policyId.$hexName" \
--tx-out $tokenAddress+$adaAmount+"$tokenAmount $policyId.$hexName" \
--out-file ft/send.raw
```

## Step-7 Sign Transaction

```bash
cardano-cli transaction sign \
--signing-key-file payment.skey \
--$network \
--tx-body-file ft/send.raw \
--out-file ft/send.signed
```

## Step-8 Submit Transaction

```bash
cardano-cli transaction submit \
--tx-file ft/send.signed \
--$network
```

# Demo

The following is a video recorded by the Indonesian Cardano Developers Community where I demonstrated the steps above. Watch the recorded video at timestamp **_1:27:27_**, here is the [link](https://youtu.be/03hXLZ_07N0?list=PLUj8499OocHiL8gXPv8wMlLW-zIcyYdrQ).

# References

[Cardano Developer Portal: Minting Native Assets](https://developers.cardano.org/docs/native-tokens/minting)

[CIP-35 On-Chain Token Metadata Standard](https://github.com/cardano-foundation/CIPs/blob/1d9fbd0e29f07b931bf1524c7aed6635d478cd75/CIP-0035/CIP-0035.md)
