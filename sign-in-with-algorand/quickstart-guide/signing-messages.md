---
description: Here we learn how to customize the SIWA Connect client
---

# Message Signing

Signing SIWA Messages

**Pera Wallet**

Pera Wallet supports signing arbitrary data directly:

```typescript
const signMessageWithPera = async (message: string): Promise<Uint8Array> => {
  const hashedMessage = hashMessage(message);
  const encodedHashedMessage = getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));

  const peraSigArray = await peraWallet.signData(
    [{ data: encodedHashedMessage, message: "" }],
    algoAddress
  );
  return peraSigArray[0];
};
```

**Defly Wallet**

Defly Wallet requires an alternative approach since it does not currently support signing arbitrary messages:

```typescript
const signMessageWithDefly = async (message: string): Promise<Uint8Array> => {
  const hashedMessage = hashMessage(message);
  const encodedHashedMessage = getMessageBytes(Buffer.from(hashedMessage).toString("utf8"));

  const suggestedParams = await algodClient.getTransactionParams().do();
  const txn = algosdk.makePaymentTxnWithSuggestedParamsFromObject({
    note: encodedHashedMessage,
    from: algoAddress,
    to: algoAddress,
    amount: 0,
    suggestedParams,
  } as any);

  const txnGroup = [{ txn, signerAddress: [algoAddress] }];
  const deflySigArray = await deflyWallet.signTransaction([txnGroup]);
  const decodedTxn = algosdk.decodeSignedTransaction(deflySigArray[0]);
  return decodedTxn.sig as unknown as Uint8Array;
};
```

#### Unified Signing Function

Use the appropriate wallet-specific signing logic:

```typescript
const signMessage = async (message: string): Promise<Uint8Array> => {
  if (!algoAddress) {
    throw new Error("No address connected");
  }

  switch (provider) {
    case "PeraWallet":
      return await signMessageWithPera(message);
    case "Defly":
      return await signMessageWithDefly(message);
    default:
      throw new Error("Unsupported wallet provider");
  }
};
```
