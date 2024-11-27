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

### Initialization

Create an instance of the Defly Wallet connection and initialize the Algod client:

```javascript
const deflyWallet = new DeflyWalletConnect();
const algodClient = initializeAlgodClient();
```

### Connecting the Wallet

To connect the Defly Wallet, use the following function:

```javascript
const connectDeflyWallet = async () => {
  try {
    const accounts = await deflyWallet.connect();
    deflyWallet.connector?.on("disconnect", () => {
      console.log("Disconnected from Defly Wallet");
    });
    return accounts[0]; // Returns the connected address
  } catch (error) {
    console.error("Error connecting to Defly Wallet:", error);
    throw error;
  }
};
```

### Signing Messages

Defly Wallet currently does not support signing arbitrary data directly. Instead, it signs a zero-amount transaction containing the message as a note.

1.  Hash and encode the message using utility functions:

    ```javascript
    const prepareMessage = (message) => {
      const hashedMessage = hashMessage(message);
      return getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));
    };
    ```
2.  Use a transaction as the signing mechanism:

    ```javascript
    const signMessageWithDefly = async (message, algoAddress) => {
      const encodedMessage = prepareMessage(message);

      const suggestedParams = await algodClient.getTransactionParams().do();
      const txn = algosdk.makePaymentTxnWithSuggestedParamsFromObject({
        note: encodedMessage,
        from: algoAddress,
        to: algoAddress,
        amount: 0,
        suggestedParams,
      } as any);

      const txnGroup = [{ txn, signerAddress: [algoAddress] }];
      const signedTxnArray = await deflyWallet.signTransaction([txnGroup]);

      const decodedTxn = algosdk.decodeSignedTransaction(signedTxnArray[0]);
      return decodedTxn.sig as unknown as Uint8Array; // Returns the signature
    };
    ```

### Disconnecting the Wallet

To disconnect the wallet, call:

```javascript
const disconnectDeflyWallet = () => {
  deflyWallet.disconnect();
  console.log("Defly Wallet disconnected");
};
```

### Full Example

Here is a complete example of connecting, signing, and disconnecting the Defly Wallet:

```javascript
import { DeflyWalletConnect } from "@blockshake/defly-connect";
import algosdk from "algosdk";
import { getMessageBytes, hashMessage, initializeAlgodClient } from "@/utils/siwaUtils";

const deflyWallet = new DeflyWalletConnect();
const algodClient = initializeAlgodClient();

const connectDeflyWallet = async () => {
  try {
    const accounts = await deflyWallet.connect();
    deflyWallet.connector?.on("disconnect", () => {
      console.log("Disconnected from Defly Wallet");
    });
    return accounts[0];
  } catch (error) {
    console.error("Error connecting to Defly Wallet:", error);
    throw error;
  }
};

const prepareMessage = (message) => {
  const hashedMessage = hashMessage(message);
  return getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));
};

const signMessageWithDefly = async (message, algoAddress) => {
  const encodedMessage = prepareMessage(message);

  const suggestedParams = await algodClient.getTransactionParams().do();
  const txn = algosdk.makePaymentTxnWithSuggestedParamsFromObject({
    note: encodedMessage,
    from: algoAddress,
    to: algoAddress,
    amount: 0,
    suggestedParams,
  } as any);

  const txnGroup = [{ txn, signerAddress: [algoAddress] }];
  const signedTxnArray = await deflyWallet.signTransaction([txnGroup]);

  const decodedTxn = algosdk.decodeSignedTransaction(signedTxnArray[0]);
  return decodedTxn.sig as unknown as Uint8Array;
};

const disconnectDeflyWallet = () => {
  deflyWallet.disconnect();
  console.log("Defly Wallet disconnected");
};

(async () => {
  const algoAddress = await connectDeflyWallet();
  const message = "Sign this message to authenticate.";
  const signature = await signMessageWithDefly(message, algoAddress);
  console.log("Signature:", signature);
  disconnectDeflyWallet();
})();
```

### Conclusion

By following this guide, you can integrate Defly Wallet to enable SIWA message signing in your application. Although Defly Wallet’s current limitations require the use of a zero-amount transaction for signing, this approach ensures secure and reliable authentication.
