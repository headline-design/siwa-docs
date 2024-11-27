---
description: >-
  This guide explains how to implement server-side verification for Sign-In with
  Algorand (SIWA) messages. The server-side verification is crucial for
  maintaining security and integrity.
---

# Server-side operations

{% hint style="info" %}
The SIWA Connect template executes message validation on the client for demonstration purposes.
{% endhint %}

***

### Verifying SIWA Messages

Here's an example implementation of server-side SIWA message verification in a Node.js environment:

```typescript
import { SiwaMessage } from "@avmkit/siwa";
import { verifySignature } from "./utils";

/**
 * Verifies the SIWA message on the server
 * @param {Object} payload - The payload from the client containing credentials
 * @returns {Object} - Verification result
 */
const verifySIWAMessageOnServer = async (payload) => {
  try {
    // Extract message and credentials from the payload
    const { message, signature, address, provider, encodedTransaction, nfd } = payload;

    // Initialize the SiwaMessage instance
    const siwaMessageInstance = new SiwaMessage(JSON.parse(message));

    // Perform the verification
    const isValid = await verifySignature(
      siwaMessageInstance,
      signature,
      provider,
      encodedTransaction,
      nfd
    );

    if (isValid) {
      console.log("SIWA message verified successfully");
      return { success: true, user: { address, provider } }; // Attach verified user data
    } else {
      throw new Error("Failed to verify SIWA message");
    }
  } catch (error) {
    console.error("Verification error:", error);
    return { success: false, error: error.message };
  }
```

### Integration in an API Endpoint

To integrate this into your backend API, create an endpoint to handle SIWA verification requests. Here's an example using Express.js:

```typescript
import express from "express";
import { verifySIWAMessageOnServer } from "./siwaVerification";

const app = express();
app.use(express.json());

// SIWA verification endpoint
app.post("/api/siwa/verify", async (req, res) => {
  const result = await verifySIWAMessageOnServer(req.body);

  if (result.success) {
    res.status(200).json({ message: "Verification successful", user: result.user });
  } else {
    res.status(400).json({ message: "Verification failed", error: result.error });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Client-side Verification for Demonstration

While server-side verification is crucial for security in production environments, the SIWA Connect demo implements client-side verification for demonstration purposes. This allows users to see the entire process, including verification, without the need for a backend server.

Here's how the client-side verification is implemented in the demo:

```typescript
const verifySIWAMessage = async () => {
  if (!siwaMessageInstance || !signedMessage) {
    setError(new Error("No signed message to verify"));
    return;
  }

  setError(null);
  try {
    const siwaMessage = new SiwaMessage(
      JSON.parse(credentials?.message || "{}")
    );

    const verifyParams = {
      signature: credentials?.signature,
      domain: typeof window !== "undefined" ? window.location.host : "",
      provider: provider || null,
      encodedTransaction: credentials?.encodedTransaction || null,
    };

    const result = await siwaMessage.verify(verifyParams);
    setVerificationResult(result);

    if (result.success) {
      console.log("Signature verified successfully");
      setActiveStep(3);
    } else {
      throw result.error || new Error("Error verifying SIWA signature");
    }
  } catch (error) {
    console.error("Error verifying SIWA message:", error);
    setError(error);
  }
};
```

This function performs the following steps:

1. It checks if a SIWA message instance and a signed message exist.
2. It creates a new `SiwaMessage` instance from the stored credentials.
3. It prepares verification parameters, including the signature, domain, provider, and encoded transaction (if applicable).
4. It calls the `verify` method on the SIWA message instance with these parameters.
5. It updates the component state based on the verification result.

Important notes about this client-side verification:

1. **Demonstration Only**: This approach is used for demonstration purposes to show the full SIWA flow in a client-side application.
2. **Not Secure for Production**: In a real-world application, this verification should be performed on the server to ensure the integrity and security of the authentication process.
3. **Educational Value**: It allows developers to see how the verification process works without needing to set up a backend server.
4. **Transparency**: Users can see the full details of the signing and verification process, including the message contents and verification results.

When implementing SIWA in a production environment, always use server-side verification as described in the previous sections of this guide.
