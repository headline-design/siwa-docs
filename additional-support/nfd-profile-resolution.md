---
description: Resolve user NFD profiles
icon: globe-pointer
---

# NFD Profile Resolution

### Introduction

Non Fungible Domains (NFD) is a decentralized naming system for Algorand addresses. When integrated with Sign-In with Algorand (SIWA), NFD allows users to associate human-readable names with their Algorand addresses, enhancing user experience and making the sign-in process more intuitive.

### What is NFD?

NFD (Non Fungible Domains) is similar to the Ethereum Name Service (ENS) but built for the Algorand ecosystem. It allows users to:

1. Register human-readable names (e.g., "alice.algo") for their Algorand addresses.
2. Use these names instead of long, complex Algorand addresses in various applications.
3. Resolve NFD names to Algorand addresses and vice versa.

### Importance in SIWA

Integrating NFD resolution into SIWA offers several benefits:

1. **Improved User Experience**: Users can sign in with memorable names instead of long Algorand addresses.
2. **Enhanced Security**: By allowing users to verify their identity through both their Algorand address and NFD, it adds an extra layer of verification.
3. **Seamless Integration**: Applications can display user-friendly names while still working with underlying Algorand addresses.

### Implementation

To implement NFD resolution in your SIWA integration, you'll need to use the NFD API. Here are the key functions for NFD resolution:

#### 1. Resolving NFD to Address

```typescript
export const resolveNFDToAddress = async (nfd: string): Promise<string | null> => {
  try {
    const response = await fetch(`https://api.nf.domains/nfd/${nfd}`, {
      method: 'GET',
      headers: {},
    });

    if (!response.ok) {
      console.log(`HTTP error! status: ${response.status}`);
      return null;
    }

    const data = await response.json();

    if (data && data.owner) {
      return data.owner;
    } else {
      console.log("Owner property not found in the response.");
    }
  } catch (error) {
    console.error("Error resolving NFD to address:", error);
  }
  return null;
};
```

#### 2. Resolving Address to NFD

```typescript
export const resolveAddressToNFD = async (address: string): Promise<string | null> => {
  try {
    const response = await fetch(
      `https://api.nf.domains/nfd/lookup?address=${address}`,
      {
        method: "GET",
        headers: {},
      }
    );

    if (!response.ok) {
      console.log(`HTTP error! status: ${response.status}`);
      return null;
    }

    const data = await response.json();

    if (data && Object.keys(data).length > 0) {
      const key = Object.keys(data)[0];
      if (data[key] && data[key].name) {
        return data[key].name;
      } else {
        console.log("Name property not found in the response.");
      }
    } else {
      console.log("Response data is empty or malformed.");
    }
  } catch (error) {
    console.log("Error resolving address to NFD:", error);
  }
  return null;
};
```

#### 3. Validating NFD Address

```typescript
export const validateNFDAddress = async (address: string, nfd: string): Promise<boolean> => {
  const nfdAddress = await resolveNFDToAddress(nfd);

  if (!nfdAddress) {
    console.log('NFD not found');
    return false;
  }
  return nfdAddress === address;
}
```

### Integration with SIWA

When integrating NFD resolution into your SIWA flow:

1. **During Sign-In**: Allow users to enter either their Algorand address or their NFD name.
2. **Resolution**: If an NFD name is provided, resolve it to an Algorand address using `resolveNFDToAddress`.
3. **Verification**: After signature verification, use `validateNFDAddress` to ensure the provided NFD matches the signing address.

Example integration in the SIWA verification process:

```typescript
export const verifySignature = async (
  message: SiwaMessage,
  signature: string,
  nfd?: string,
): Promise<boolean> => {
  // ... existing signature verification logic ...

  if (nfd) {
    if (!await validateNFDAddress(address, nfd)) {
      console.log('NFD validation failed');
      return false;
    }
  }

  // ... continue with other verification steps ...
};
```

### Best Practices

1. **Error Handling**: Always handle potential errors in NFD resolution gracefully. NFD servers might be down or slow to respond.
2. **Caching**: Consider caching NFD resolutions to reduce API calls and improve performance. Ensure you have a strategy to invalidate the cache when needed.
3. **User Choice**: Allow users to choose whether they want to use their NFD or Algorand address for sign-in.
4. **Display Both**: When possible, display both the NFD and Algorand address to the user for transparency.
5. **Security**: Remember that NFD ownership can change. Always verify the NFD-to-address mapping during each sign-in attempt.

### Conclusion

Integrating NFD resolution into your SIWA implementation can significantly enhance user experience and add an extra layer of identity verification. By following the implementation guidelines and best practices outlined above, you can create a more user-friendly and secure authentication process for your Algorand-based application.
