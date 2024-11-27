---
description: >-
  Here we learn how to connect the user's wallet with the web application and
  sign messages.
---

# Component Overview

The `SIWAConnect` component is a React component designed to manage the authentication flow using Sign-In With Algo. It leverages several hooks and state variables for its functionality. Below is a brief overview of its core components:

* **Hooks:**
  * `useWalletConnection()`: Provides necessary functions and state variables for wallet connectivity, such as connecting/disconnecting wallets, loading states, and signing messages.
  * `useSIWAAccount()`: Accepts `address` and returns account-related information, specifically the `address`.
* **State Variables:**
  * **signedMessage**: Stores the message signed by the user.
  * **pendingProvider**: Tracks the wallet provider waiting for user interaction.
  * **fullSigningMessage**: Holds the comprehensive signing message object.
  * **credentials**: Stores user credentials after successful authentication.
  * **error**: Captures error information if any issues occur during the authentication flow.
  * **verificationResult**: Maintains the result of the SIWA message verification process.
  * **siwaMessageInstance**: Stores an instance of the `SiwaMessage`.
  * **activeStep**: Keeps track of the current step in the authentication process.

**Key Functions**

The component implements several key functions:

1. `handleConnectWallet`: Connects to the selected wallet provider
2. `handleDisconnect`: Disconnects the current wallet
3. `signIn`: Creates and signs a SIWA message
4. `verifySIWAMessage`: Verifies the signed SIWA message

Here's an example of the `signIn` function:

```typescript
const signIn = async () => {
  if (!address) {
    setError(new Error("No address connected"));
    return;
  }

  setError(null);
  setSigning(true);
  try {
    const uri = typeof window !== "undefined" ? window.location.origin : "";
    const domain = typeof window !== "undefined" ? window.location.host : "";

    const siwaMessage = new SiwaMessage({
      domain,
      address,
      statement: "Sign in with Algorand to the app.",
      uri,
      version: "1",
      chainId: 416001,
      nonce: "randomNonce123", // In production, use a secure random nonce
    });

    setSiwaMessageInstance(siwaMessage);
    setFullSigningMessage(siwaMessage);

    const messageToSign = siwaMessage.prepareMessage();
    const { signature, transaction: encodedTransaction } = await signMessage(messageToSign);

    const algoSig = uint8ArrayToBase64(signature);

    setCredentials({
      message: JSON.stringify(siwaMessage),
      encodedTransaction: encodedTransaction || null,
      provider: provider || null,
      signature: algoSig,
      address: address,
    });

    setSignedMessage(Buffer.from(signature).toString("base64"));
    setActiveStep(2);
    setSigning(false);
  } catch (error) {
    console.error("Error signing message:", error);
    setError(error);
    setSigning(false);
  }
};
```

### UI Rendering

The UI is rendered based on the current `activeStep`. Here's the complete `renderStep` function:

```typescript
const renderStep = () => {
  switch (activeStep) {
    case 0:
      return (
        <div className="space-y-4">
          <PeraConnectButton
            isLoading={isLoading && pendingProvider === "Pera"}
            onConnect={() => handleConnectWallet("Pera")}
          />
          <DeflyConnectButton
            onConnect={() => handleConnectWallet("Defly")}
            isLoading={isLoading && pendingProvider === "Defly"}
          />
          <KibisisConnectButton
            isLoading={isLoading && pendingProvider === "Kibisis"}
            onConnect={() => handleConnectWallet("Kibisis")}
          />
          <LuteConnectButton
            onConnect={() => handleConnectWallet("Lute")}
            isLoading={isLoading && pendingProvider === "Lute"}
          />
        </div>
      );
    case 1:
      return (
        <div>
          <Button className="h-12 w-full" onClick={signIn} disabled={signing}>
            {signing ? <LoadingSpinner /> : "Sign In"}
          </Button>
        </div>
      );
    case 2:
      return (
        <div>
          <Button
            className="h-12 w-full"
            onClick={verifySIWAMessage}
            disabled={isLoading}
          >
            {isLoading ? <LoadingSpinner /> : "Verify SIWA Message"}
          </Button>
        </div>
      );
    case 3:
      return (
        <Alert variant="success">
          <p className="font-semibold">Verification Successful</p>
          <p>You have successfully signed in with Algorand.</p>
        </Alert>
      );
    default:
      return null;
  }
};
```

This function renders different UI elements based on the current step of the SIWA process:

1. Display connect buttons for all supported wallets (Pera, Defly, Kibisis, and Lute)
2. Show the "Sign In" button
3. Show the "Verify SIWA Message" button
4. Display a success message

The component also includes additional UI elements for displaying the connected wallet information, error messages, and JSON representations of the signing message, credentials, and verification result.

In the next section, we'll look at how to customize the UI of the SIWA Connect component to fit your application's design.

In the next section, we'll look at how to customize the UI of the SIWA Connect component.
