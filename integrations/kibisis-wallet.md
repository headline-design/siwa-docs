---
description: >-
  This guide explains how to integrate Kibisis wallet with Sign-In with Algorand
  (SIWA) for message signing.
---

# Kibisis Wallet

{% embed url="https://github.com/headline-design/siwa-connect" %}
Get started quickly with SIWA Connect
{% endembed %}

### Prerequisites

Install the Kibisis wallet extension on Chrome

### Overview

Kibisis wallet uses a direct message signing approach for SIWA messages. Here's how it works:

1. Inject the Kibisis wallet script
2. Enable the wallet and get the user's address
3. Sign the SIWA message using the wallet's `signBytes` method

### Implementation

#### 1. Injecting and Connecting to Kibisis Wallet

To use Kibisis wallet, you need to inject its script and enable it:

```javascript
const injectKibisis = async () => {
  if (typeof window === "undefined") {
    throw new Error("Kibisis is only available in the browser");
  }

  async function enableWallet() {
    if (!window.algorand) {
      console.error("AVM Wallets not available");
      return null;
    }

    try {
      const result = await window.algorand.enable("kibisis");
      console.log("Wallet enabled:", result);
      if (result.accounts && result.accounts.length > 0) {
        return result.accounts[0].address;
      } else {
        throw new Error("No accounts available");
      }
    } catch (error) {
      console.error("Error enabling wallet:", error);
      return null;
    }
  }

  let address = await enableWallet();
  if (address) {
    return address;
  } else {
    console.log("No address obtained or user cancelled.");
    throw new Error("User cancelled or no accounts available");
  }
};
```

#### 2. Signing a SIWA Message

To sign a SIWA message with Kibisis wallet:

1. Prepare the message by prefixing it with "MX"
2. Use the `window.algorand.signBytes` method to sign the message

Here's the code to accomplish this:

```javascript
const signMessage = async (message: string) => {
  if (!address) {
    throw new Error("No address connected");
  }

  const kibisisMessage = "MX" + JSON.stringify(message);
  const kibisisResult = await window.algorand.signBytes({
    data: new Uint8Array(Buffer.from(kibisisMessage)),
  });

  return {
    signature: kibisisResult?.signature,
    transaction: null,
  };
};
```

### Verification

When verifying the SIWA message signature for Kibisis wallet, the process is straightforward as it uses direct message signing. Here's how it works:

1. The verification function receives the following parameters:
   * `message`: The original SIWA message
   * `signature`: The signature in base64 format
   * `provider`: The wallet provider (in this case, "Kibisis")
2. The verification process for Kibisis wallet:

```javascript
if (provider === "Kibisis") {
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

Key points to remember for Kibisis wallet verification:

1. The message used for verification should be the same as the one used for signing (including the "MX" prefix).
2. The `algosdk.verifyBytes` function is used to verify the signature against the hashed message and the signer's address.
3. Any errors during the verification process will result in a `false` return value.

Remember to handle any potential errors during the verification process and ensure that all required parameters are provided.
