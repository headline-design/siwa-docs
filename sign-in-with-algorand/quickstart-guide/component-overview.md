---
description: >-
  Here we learn how to connect the user's wallet with the web application and
  sign messages.
---

# Component Overview

#### SIWAConnect Component Overview

The `SIWAConnect` component is a React component designed to manage the authentication flow using Sign-In With Algo. It leverages several hooks and state variables for its functionality. Below is a brief overview of its core components:

* **Hooks:**
  * `useWalletConnection()`: Provides necessary functions and state variables for wallet connectivity, such as connecting/disconnecting wallets, loading states, and signing messages.
  * `useSIWAAccount()`: Accepts `algoAddress` and returns account-related information, specifically the `address`.
*   **State Variables:**

    * **signedMessage**: Stores the message signed by the user.
    * **pendingProvider**: Tracks the wallet provider waiting for user interaction.
    * **fullSigningMessage**: Holds the comprehensive signing message object.
    * **credentials**: Stores user credentials after successful authentication.
    * **error**: Captures error information if any issues occur during the authentication flow.
    * **verificationResult**: Maintains the result of the SIWA message verification process.
    * **siwaMessageInstance**: Stores an instance of the `SiwaMessage`.
    * **activeStep**: Keeps track of the current step in the authentication process.



    #### Key Functions

    The component powers various essential functions that facilitate the authentication process:

    * **handleConnectWallet**: Initiates the connection to the chosen wallet provider.
    * **signIn**: Generates and signs a SIWA message to authenticate the user.
    * **verifySIWAMessage**: Ensures the authenticity of the signed SIWA message, verifying its validity and integrity.

```typescript
import React, { useState, useEffect } from "react";
import { SiwaMessage } from "@avmkit/siwa";
import useSIWAAccount from "@/hooks/useSIWAAccount";
import { useWalletConnection, WalletProvider } from "@/hooks/useWalletConnection";
import { uint8ArrayToBase64, uint8ArrayToEthereumHexString } from "@/utils/siwaUtils";
// ... other imports
```

2. The component uses several hooks and state variables to manage the authentication flow:

```typescript
export default function SIWAConnect() {
  const {
    algoAddress,
    provider,
    isLoading,
    connectWallet,
    disconnectWallet,
    signMessage,
  } = useWalletConnection();

  const [signedMessage, setSignedMessage] = useState<string | null>(null);
  const [pendingProvider, setPendingProvider] = useState<WalletProvider | null>(null);
  const [fullSigningMessage, setFullSigningMessage] = useState<any | null>(null);
  const [credentials, setCredentials] = useState<any | null>(null);
  const [error, setError] = useState<any | null>(null);
  const [verificationResult, setVerificationResult] = useState<any | null>(null);
  const [siwaMessageInstance, setSiwaMessageInstance] = useState<SiwaMessage | null>(null);
  const [activeStep, setActiveStep] = useState(0);

  const { address } = useSIWAAccount(algoAddress || "");

  // ... rest of the component
}
```

3. The component implements several key functions:

* `handleConnectWallet`: Connects to the selected wallet provider
* `signIn`: Creates and signs a SIWA message
* `verifySIWAMessage`: Verifies the signed SIWA message

4. The UI is rendered based on the current `activeStep`:

```typescript
const renderStep = () => {
  switch (activeStep) {
    case 0:
      return (
        <div className="space-y-4">
          <PeraConnectButton
            isLoading={isLoading && pendingProvider === "PeraWallet"}
            onConnect={() => handleConnectWallet("PeraWallet")}
          />
          <DeflyConnectButton
            onConnect={() => handleConnectWallet("Defly")}
            isLoading={isLoading && pendingProvider === "Defly"}
          />
        </div>
      );
    case 1:
      return (
        <div>
          <Button onClick={signIn} disabled={isLoading}>
            {isLoading ? <LoadingSpinner /> : "Sign In"}
          </Button>
        </div>
      );
    // ... cases for steps 2 and 3
  }
};
```

In the next section, we'll look at how to customize the UI of the SIWA Connect component.
