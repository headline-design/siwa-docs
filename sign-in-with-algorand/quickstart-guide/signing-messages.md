---
description: >-
  This guide explains how to sign SIWA (Sign-In with Algorand) messages using
  different wallet providers: Pera, Defly, Kibisis, and Lute. Each wallet has
  its own method for signing messages.
---

# Message Signing

### Pera Wallet

Pera Wallet supports signing arbitrary data directly:

```typescript
const signMessageWithPera = async (message: string, address: string): Promise<{ signature: Uint8Array; transaction: null }> => {
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

### Defly Wallet

Defly Wallet requires creating a transaction with the message in the note field:

```typescript
const signMessageWithDefly = async (message: string, address: string): Promise<{ signature: Uint8Array; transaction: Uint8Array }> => {
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

### Kibisis Wallet

Kibisis Wallet uses a browser-based approach for signing:

```typescript
const signMessageWithKibisis = async (message: string): Promise<{ signature: Uint8Array; transaction: null }> => {
  if (typeof window === "undefined") {
    throw new Error("Kibisis is only available in the browser");
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

### Lute Wallet

Lute Wallet, like Defly, uses a transaction-based approach:

```typescript
const signMessageWithLute = async (message: string, address: string): Promise<{ signature: Uint8Array; transaction: Uint8Array }> => {
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

  const luteSigArray = await luteWallet!.signTxns([
    { txn: Buffer.from(algosdk.encodeUnsignedTransaction(luteTxn)).toString("base64") },
  ]);

  const decodedLuteTxn = algosdk.decodeSignedTransaction(luteSigArray[0]);

  return {
    signature: decodedLuteTxn.sig as unknown as Uint8Array,
    transaction: luteSigArray[0],
  };
};
```

### Unified Signing Function

Here's a unified function that handles signing for all supported wallet providers:

```typescript
const signMessage = async (message: string): Promise<{ signature: Uint8Array; transaction?: any | null }> => {
  if (!address) {
    throw new Error("No address connected");
  }

  const hashedMessage = hashMessage(message);
  const encodedHashedMessage = getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));

  const suggestedParams = ["Defly", "Lute"].includes(provider)
    ? await algodClient.getTransactionParams().do()
    : null;

  switch (provider) {
    case "Pera":
      return await signMessageWithPera(message, address);

    case "Defly":
      if (!suggestedParams) {
        throw new Error("Suggested params are not available");
      }
      return await signMessageWithDefly(message, address);

    case "Kibisis":
      return await signMessageWithKibisis(message);

    case "Lute":
      if (!suggestedParams) {
        throw new Error("Suggested params are not available");
      }
      return await signMessageWithLute(message, address);

    default:
      throw new Error("Unsupported wallet provider");
  }
};
```

This unified `signMessage` function determines the correct signing method based on the current wallet provider. It handles the differences in signing approaches between the wallets, ensuring a consistent interface for your application.

Remember to import and initialize the necessary wallet SDKs and utility functions (like `hashMessage`, `getMessageBytes`, and `algodClient`) before using these signing functions.
