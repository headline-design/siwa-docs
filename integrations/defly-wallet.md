---
description: >-
  This documentation outlines the steps to integrate Defly Wallet for signing
  "Sign In With Algorand" (SIWA) messages.
---

# Defly Wallet

{% embed url="https://github.com/headline-design/siwa-connect" %}
Get started quickly w/ SIWA Connect
{% endembed %}

### Prerequisites

1.  Install the Defly Wallet library:

    ```bash
    npm install @blockshake/defly-connect
    ```
2.  Import required modules:

    ```javascript
    import { DeflyWalletConnect } from "@blockshake/defly-connect";
    import algosdk from "algosdk";
    import { getMessageBytes, hashMessage, initializeAlgodClient } from "@/utils/siwaUtils";
    ```

### Overview

Defly wallet uses a transaction-based approach for signing SIWA messages, similar to Lute wallet. Here's how it works:

1. Create a zero-amount payment transaction
2. Set the transaction's note field to the encoded SIWA message
3. Sign the transaction using Defly wallet
4. Extract the signature from the signed transaction

### Implementation

#### 1. Connecting to Defly Wallet

To connect to Defly wallet, use the `DeflyWalletConnect` class:

```javascript
import { DeflyWalletConnect } from "@blockshake/defly-connect";

const deflyWallet = new DeflyWalletConnect();

const connectWallet = async () => {
  try {
    const newAccounts = await deflyWallet.connect();
    deflyWallet.connector?.on("disconnect", handleDisconnect);
    return newAccounts[0];
  } catch (error) {
    console.error("Error connecting to Defly wallet:", error);
    throw error;
  }
};
```

#### 2. Signing a SIWA Message

To sign a SIWA message with Defly wallet:

1. Create a payment transaction with the SIWA message in the note field
2. Sign the transaction using Defly wallet
3. Extract the signature from the signed transaction

Here's the code to accomplish this:

```javascript
import algosdk from "algosdk";
import { hashMessage, getMessageBytes } from "@/utils/siwaUtils";

const signMessage = async (message: string) => {
  if (!address) {
    throw new Error("No address connected");
  }

  const hashedMessage = hashMessage(message);
  const encodedHashedMessage = getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));

  const suggestedParams = await algodClient.getTransactionParams().do();

  const deflyTxn = algosdk.makePaymentTxnWithSuggestedParamsFromObject({
    note: encodedHashedMessage,
    from: address,
    to: address,
    amount: 0,
    suggestedParams,
  } as any);

  const deflyTxnGroup = [{ txn: deflyTxn, signerAddress: [address] }];
  const deflySigArray = await deflyWallet.signTransaction([deflyTxnGroup]);
  const decodedDeflyTxn = algosdk.decodeSignedTransaction(deflySigArray[0]);

  return {
    signature: decodedDeflyTxn.sig as unknown as Uint8Array,
    transaction: deflySigArray[0],
  };
};
```

### Verification

When verifying the SIWA message signature for Defly wallet, the process is similar to Lute wallet due to the transaction-based approach. Here's how it works:

1. The verification function receives the following parameters:
2. `message`: The original SIWA message
3. `signature`: The signature in base64 format
4. `provider`: The wallet provider (in this case, "Defly")
5. `encodedTransaction`: The encoded transaction in Base64 format
6. The verification process for Defly wallet:

```javascript
if (provider === "Defly") {
  if (!encodedTransaction) {
    return false; // Defly requires an encoded transaction
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
