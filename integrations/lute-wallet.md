---
description: >-
  This guide explains how to integrate Lute wallet with Sign-In with Algorand
  (SIWA) for transaction signing.
---

# Lute Wallet

{% embed url="https://github.com/headline-design/siwa-connect" %}
Get started quickly with SIWA Connect
{% endembed %}

### Prerequisites

1. Install the Lute wallet extension on Chrome
2. Install the lute-connect NPM package

### Overview

Lute wallet uses a transaction-based approach for signing SIWA messages. Here's how it works:

1. Create a zero-amount payment transaction
2. Set the transaction's note field to the encoded SIWA message
3. Sign the transaction using Lute wallet
4. Extract the signature from the signed transaction

### Implementation

#### 1. Connecting to Lute Wallet

To connect to Lute wallet, use the `LuteConnect` class:

```javascript
import LuteConnect from "lute-connect";

const luteWallet = new LuteConnect("SIWA Connect");

const connectLute = async () => {
  const genesis = await algodClient.genesis().do();
  const genesisID = `${genesis.network}-${genesis.id}`;
  const addresses = await luteWallet.connect(genesisID);
  return addresses[0];
};
```

#### 2. Signing a SIWA Message

To sign a SIWA message with Lute wallet:

1. Create a payment transaction with the SIWA message in the note field
2. Sign the transaction using Lute wallet
3. Extract the signature from the signed transaction

Here's the code to accomplish this:

```javascript
import algosdk from "algosdk";

const signMessage = async (message: string) => {
  if (!address) {
    throw new Error("No address connected");
  }

  const hashedMessage = hashMessage(message);
  const encodedHashedMessage = getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));

  const suggestedParams = await algodClient.getTransactionParams().do();

  const luteTxn = algosdk.makePaymentTxnWithSuggestedParamsFromObject({
    note: encodedHashedMessage,
    from: address,
    to: address,
    amount: 0,
    suggestedParams,
  });

  const luteSigArray = await luteWallet.signTxns([
    { txn: Buffer.from(algosdk.encodeUnsignedTransaction(luteTxn)).toString("base64") },
  ]);

  const decodedLuteTxn = algosdk.decodeSignedTransaction(luteSigArray[0]);

  return {
    signature: decodedLuteTxn.sig as unknown as Uint8Array,
    transaction: luteSigArray[0],
  };
};
```

### Verification

When verifying the SIWA message signature for Lute wallet, the process is more complex due to the transaction-based approach. Here's how it works:

1. The verification function receives the following parameters:
   * `message`: The original SIWA message
   * `signature`: The signature in base64 format
   * `provider`: The wallet provider (in this case, "Lute")
   * `encodedTransaction`: The encoded transaction in Base64 format
2. The verification process for Lute wallet:

```javascript
if (provider === "Lute") {
  if (!encodedTransaction) {
    return false; // Lute requires an encoded transaction
  }

  try {
    // Decode the transaction
    const packTransaction = Buffer.from(encodedTransaction, "base64");
    const decodedTransaction = algosdk.decodeSignedTransaction(packTransaction);

    // Verify the signed transaction
    const transactionResult = verifySignedTransaction(decodedTransaction);
    if (!transactionResult) {
      return false;
    }

    const { isValid: isTransactionValid, signature: txnSignature } = transactionResult;

    // Check if the transaction signature matches the provided signature
    const isSignatureValid = txnSignature === signature;

    // The final result is true only if both the transaction is valid and the signatures match
    return isTransactionValid && isSignatureValid;
  } catch (error) {
    return false; // Return false if any error occurs during verification
  }
}
```

3. The `verifySignedTransaction` function is used to validate the transaction:

```javascript
function verifySignedTransaction(stxn: SignedTransaction) {
  if (stxn.sig === undefined) return false;

  const pk_bytes = stxn.txn.from.publicKey;
  const sig_bytes = new Uint8Array(stxn.sig);
  const txn_bytes = algosdk.encodeObj(stxn.txn.get_obj_for_encoding());
  const msg_bytes = new Uint8Array(txn_bytes.length + 2);
  msg_bytes.set(Buffer.from("TX"));
  msg_bytes.set(txn_bytes, 2);

  const isValid = nacl.sign.detached.verify(msg_bytes, sig_bytes, pk_bytes);
  const signature = Buffer.from(stxn.sig).toString("base64");

  return {
    isValid,
    signature,
  };
}
```

This verification process ensures that:

1. The transaction is properly signed
2. The signature in the transaction matches the provided signature
3. The transaction is valid according to Algorand's rules

Remember to handle any potential errors during the verification process and ensure that all required parameters are provided.
