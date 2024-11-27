---
description: >-
  This guide walks you through implementing the client-side logic to integrate
  "Sign In With Algorand" (SIWA) into your application.
---

# Client-side operations

### Initializing Pera Wallet

The Pera Wallet SDK is required to enable wallet interactions. Start by importing and initializing the SDK:

```javascript
import { PeraWalletConnect } from "@perawallet/connect";

const peraWallet = new PeraWalletConnect();
```

This initializes a new Pera Wallet connection instance for your application.

***

### Connecting the Wallet

Create a function to handle user interactions for connecting their wallet:

```javascript
const connectPeraWallet = async () => {
  try {
    const accounts = await peraWallet.connect();

    // Listen for wallet disconnection
    peraWallet.connector?.on("disconnect", () => {
      console.log("Disconnected from Pera Wallet");
    });

    return accounts[0]; // Returns the connected wallet address
  } catch (error) {
    console.error("Error connecting to Pera Wallet:", error);
    throw error;
  }
};
```

This function establishes a connection to the Pera Wallet and retrieves the user’s account address.

***

### Preparing a Message for Signing

Before signing a message, it must be properly formatted. Use helper functions for encoding and hashing the message:

```javascript
import { getMessageBytes, hashMessage } from "@/utils/siwaUtils";

const prepareMessage = (message) => {
  const hashedMessage = hashMessage(message);
  return getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));
};
```

The `prepareMessage` function converts the message into a format compatible with Algorand’s signature verification.

***

### Signing the Message

To request a signature from the user, implement the following:

```javascript
const signMessageWithPera = async (message, algoAddress) => {
  const encodedMessage = prepareMessage(message);

  const signatureArray = await peraWallet.signData(
    [{ data: encodedMessage, message: "" }],
    algoAddress
  );

  return signatureArray[0]; // Returns the signed message
};
```

This function signs the prepared message using the connected wallet and returns the signature.

***

### Disconnecting the Wallet

Implement a function to handle wallet disconnections:

```javascript
const disconnectPeraWallet = () => {
  peraWallet.disconnect();
  console.log("Pera Wallet disconnected");
};
```

This ensures that the wallet session is closed properly.

***

### Managing Wallet Events

Listen for wallet events to handle connection state changes:

```javascript
peraWallet.connector?.on("connect", (accounts) => {
  console.log("Wallet connected:", accounts);
});

peraWallet.connector?.on("disconnect", () => {
  console.log("Wallet disconnected");
});
```

These event listeners keep the application state synchronized with the wallet’s connection state.

***

### Full Example: Client-Side Workflow

Below is a complete implementation of the client-side functionality:

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

With this implementation, the client can securely connect to the Pera Wallet, sign messages, and manage wallet connections. This completes the client-side integration for SIWA.
