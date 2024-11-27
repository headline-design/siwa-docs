---
description: >-
  This guide will walk you through the process of integrating Sign-In with
  Algorand (SIWA) using Pera Wallet in your web application.
---

# Pera Wallet

{% embed url="https://github.com/headline-design/siwa-connect" %}
Get started quickly with SIWA Connect
{% endembed %}

### Prerequisites

1.  Install the Pera Wallet library:

    ```bash
    npm install @perawallet/connect
    ```
2.  Import the required modules:

    ```javascript
    import { PeraWalletConnect } from "@perawallet/connect";
    import { getMessageBytes, hashMessage } from "@/utils/siwaUtils";
    ```

### Initialization

Create an instance of the Pera Wallet connection:

```javascript
const peraWallet = new PeraWalletConnect();
```

### Connecting the Wallet

To connect Pera Wallet, use the following function:

```javascript
const connectPeraWallet = async () => {
  try {
    const accounts = await peraWallet.connect();
    peraWallet.connector?.on("disconnect", () => {
      console.log("Disconnected from Pera Wallet");
    });
    return accounts[0]; // Returns the connected address
  } catch (error) {
    console.error("Error connecting to Pera Wallet:", error);
    throw error;
  }
};
```

### Signing Messages

Pera Wallet allows signing arbitrary data directly using the `signData` method. Follow these steps:

1.  Hash and encode the message:

    ```javascript
    const prepareMessage = (message) => {
      const hashedMessage = hashMessage(message);
      return getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));
    };
    ```
2.  Sign the message using Pera Wallet:

    ```javascript
    const signMessageWithPera = async (message, algoAddress) => {
      const encodedMessage = prepareMessage(message);

      const signatureArray = await peraWallet.signData(
        [{ data: encodedMessage, message: "" }],
        algoAddress
      );
      return signatureArray[0]; // Returns the signature
    };
    ```

### Disconnecting the Wallet

To disconnect Pera Wallet, call:

```javascript
const disconnectPeraWallet = () => {
  peraWallet.disconnect();
  console.log("Pera Wallet disconnected");
};
```

### Full Example

Here is a complete example of connecting, signing, and disconnecting the Pera Wallet:

```javascript
import { PeraWalletConnect } from "@perawallet/connect";
import { getMessageBytes, hashMessage } from "@/utils/siwaUtils";

const peraWallet = new PeraWalletConnect();

const connectPeraWallet = async () => {
  try {
    const accounts = await peraWallet.connect();
    peraWallet.connector?.on("disconnect", () => {
      console.log("Disconnected from Pera Wallet");
    });
    return accounts[0];
  } catch (error) {
    console.error("Error connecting to Pera Wallet:", error);
    throw error;
  }
};

const prepareMessage = (message) => {
  const hashedMessage = hashMessage(message);
  return getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));
};

const signMessageWithPera = async (message, algoAddress) => {
  const encodedMessage = prepareMessage(message);

  const signatureArray = await peraWallet.signData(
    [{ data: encodedMessage, message: "" }],
    algoAddress
  );
  return signatureArray[0];
};

const disconnectPeraWallet = () => {
  peraWallet.disconnect();
  console.log("Pera Wallet disconnected");
};

(async () => {
  const algoAddress = await connectPeraWallet();
  const message = "Sign this message to authenticate.";
  const signature = await signMessageWithPera(message, algoAddress);
  console.log("Signature:", signature);
  disconnectPeraWallet();
})();
```

### Conclusion

Pera Wallet simplifies the process of signing SIWA messages with its direct support for signing arbitrary data. Follow the steps outlined in this guide to integrate Pera Wallet into your application securely and efficiently.
