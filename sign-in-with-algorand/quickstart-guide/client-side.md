---
description: >-
  This section covers the client-side operations for Sign In with Algorand
  (SIWA), including wallet initialization, connection, message signing, and
  disconnection for multiple wallet providers.
---

# Client-side operations

### Initializing Wallet Connections

We use a custom hook `useWalletConnection` to manage wallet connections. This hook handles multiple wallet providers and abstracts the complexity of each wallet's specific implementation.

```typescript
import { useWalletConnection, WalletProvider } from "@/hooks/useWalletConnection";

// In your component
const {
  address,
  provider,
  isLoading,
  connectWallet,
  disconnectWallet,
  signMessage,
} = useWalletConnection();
```

### Connecting a Wallet

To connect a wallet, use the `connectWallet` function provided by the hook:

```typescript
const handleConnect = async (selectedProvider: WalletProvider) => {
  try {
    await connectWallet(selectedProvider);
    console.log(`Connected to ${selectedProvider} wallet`);
  } catch (error) {
    console.error(`Error connecting to ${selectedProvider}:`, error);
  }
};
```

### Signing a Message

To sign a message with the connected wallet, use the `signMessage` function:

```typescript
const handleSignMessage = async () => {
  if (!address) {
    console.error("No wallet connected");
    return;
  }

  const message = "Sign this message to authenticate.";
  try {
    const { signature, transaction } = await signMessage(message);
    console.log("Signature:", signature);
    if (transaction) {
      console.log("Transaction:", transaction);
    }
  } catch (error) {
    console.error("Error signing message:", error);
  }
};
```

### Disconnecting a Wallet

To disconnect the current wallet, use the `disconnectWallet` function:

```typescript
const handleDisconnect = () => {
  disconnectWallet();
  console.log("Wallet disconnected");
};
```

### Wallet-Specific Considerations

#### Pera Wallet

Pera Wallet uses direct message signing. No additional setup is required beyond the `useWalletConnection` hook.

#### Defly Wallet

Defly Wallet uses a transaction-based approach for signing. The hook handles creating a zero-amount payment transaction with the message in the note field.

#### Kibisis Wallet

Kibisis Wallet requires injection into the browser environment. The hook handles this process internally.

#### Lute Wallet

Lute Wallet also uses a transaction-based approach similar to Defly. The hook manages the connection and signing process.

### Full Example: Client-Side Workflow

Here's a complete example of how to use the `useWalletConnection` hook in a React component:

```typescript
import React from 'react';
import { useWalletConnection, WalletProvider } from "@/hooks/useWalletConnection";

const WalletComponent: React.FC = () => {
  const {
    address,
    provider,
    isLoading,
    connectWallet,
    disconnectWallet,
    signMessage,
  } = useWalletConnection();

  const handleConnect = async (selectedProvider: WalletProvider) => {
    try {
      await connectWallet(selectedProvider);
      console.log(`Connected to ${selectedProvider} wallet`);
    } catch (error) {
      console.error(`Error connecting to ${selectedProvider}:`, error);
    }
  };

  const handleSignMessage = async () => {
    if (!address) {
      console.error("No wallet connected");
      return;
    }

    const message = "Sign this message to authenticate.";
    try {
      const { signature, transaction } = await signMessage(message);
      console.log("Signature:", signature);
      if (transaction) {
        console.log("Transaction:", transaction);
      }
    } catch (error) {
      console.error("Error signing message:", error);
    }
  };

  return (
    <div>
      {!address ? (
        <>
          <button onClick={() => handleConnect("Pera")} disabled={isLoading}>Connect Pera Wallet</button>
          <button onClick={() => handleConnect("Defly")} disabled={isLoading}>Connect Defly Wallet</button>
          <button onClick={() => handleConnect("Kibisis")} disabled={isLoading}>Connect Kibisis Wallet</button>
          <button onClick={() => handleConnect("Lute")} disabled={isLoading}>Connect Lute Wallet</button>
        </>
      ) : (
        <>
          <p>Connected Address: {address}</p>
          <p>Wallet Provider: {provider}</p>
          <button onClick={handleSignMessage}>Sign Message</button>
          <button onClick={disconnectWallet}>Disconnect</button>
        </>
      )}
    </div>
  );
};

export default WalletComponent;
```

This implementation provides a unified interface for connecting to different Algorand wallets, signing messages, and managing wallet connections. It abstracts away the complexities of each wallet's specific implementation, providing a consistent experience across different wallet providers.
