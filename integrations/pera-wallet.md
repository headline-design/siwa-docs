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

1. Install the mobile app for Pera on Apple or Android
2.  Install the Pera Wallet library:

    ```bash
    npm install @perawallet/connect
    ```

### Overview

Pera wallet uses a direct message signing approach for SIWA messages. Here's how it works:

1. Connect to the Pera wallet
2. Prepare the SIWA message
3. Sign the message using Pera wallet's `signData` method
4. Verify the signature

### Implementation

#### 1. Connecting to Pera Wallet

To connect to Pera wallet, use the `PeraWalletConnect` class:

```javascript
import { PeraWalletConnect } from "@perawallet/connect";

const peraWallet = new PeraWalletConnect();

const connectWallet = async () => {
  try {
    const newAccounts = await peraWallet.connect();
    peraWallet.connector?.on("disconnect", handleDisconnect);
    return newAccounts[0];
  } catch (error) {
    console.error("Error connecting to Pera wallet:", error);
    throw error;
  }
};
```

#### . Signing a SIWA Message

To sign a SIWA message with Pera wallet:

1. Prepare the message by hashing and encoding it
2. Use the `peraWallet.signData` method to sign the message

Here's the code to accomplish this:

```javascript
import { hashMessage, getMessageBytes } from "@/utils/siwaUtils";

const signMessage = async (message: string) => {
  if (!address) {
    throw new Error("No address connected");
  }

  const hashedMessage = hashMessage(message);
  const encodedHashedMessage = getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));

  const peraSigArray = await peraWallet.signData(
    [{ data: encodedHashedMessage, message: "" }],
    address
  );

  return {
    signature: peraSigArray[0],
    transaction: null,
  };
};
```

### Verification

When verifying the SIWA message signature for Pera wallet, the process is straightforward as it uses direct message signing. Here's how it works:

1. The verification function receives the following parameters:
2. `message`: The original SIWA message
3. `signature`: The signature in base64 format
4. `provider`: The wallet provider (in this case, "Pera")
5. The verification process for Pera wallet:

```javascript
if (provider === "Pera") {
  try {
    // Prepare hashed message bytes
    const hashedMessage = new Uint8Array(Buffer.from(JSON.stringify(message.prepareMessage())));

    // Decode the signature from base64 to Uint8Array
    const signatureUint8Array = Uint8Array.from(atob(signature).split("").map((char) => char.charCodeAt(0)));

    // Verify the signature using algosdk
    const isValid = algosdk.verifyBytes(hashedMessage, signatureUint8Array, message.address);

    return isValid;
  } catch (error) {
    return false; // Return false if any error occurs during verification
  }
}
```

This verification process ensures that:

1. The message is properly hashed
2. The signature is valid for the given message and address

Key points to remember for Pera wallet verification:

1. The `algosdk.verifyBytes` function is used to verify the signature against the hashed message and the signer's address.
2. Any errors during the verification process will result in a `false` return value.

Remember to handle any potential errors during the verification process and ensure that all required parameters are provided.
