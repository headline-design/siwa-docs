---
description: >-
  In a full-stack implementation of "Sign In With Algorand" (SIWA), message
  verification should be handled securely on the server.
---

# Server-side operations

{% hint style="info" %}
The SIWA Connect template executes message validation on the client for demonstration purposes.
{% endhint %}

***

### Server-Side Verification Overview

The SIWA Connect client-side template includes the following logic for verifying a signed message:

```javascript
/**
 * Verify the SIWA message
 */
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
      signature: credentials.signature,
      domain: window.location.host,
      address: address || undefined,
      address: address || undefined,
      signature: credentials.signature,
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

In a full-stack environment, **message verification must occur on the server** to prevent tampering and ensure security.

***

### Setting Up Server-Side Verification

#### Using the SIWA Verify Method

The `SiwaMessage.verify` function is the core of server-side verification. It validates the following:

* The message's integrity (Grammar API compliance).
* The authenticity of the Algorand address and signature.
* All required parameters in the credentials object.

**Verify Method Signature**

```typescript
(method) SiwaMessage.verify(params: VerifyParams, opts?: VerifyOpts): Promise<SiwaResponse>
```

**Parameters:**

* `params` (required): Contains the message, signature, domain, and other verification parameters.
* `opts` (optional): Additional options for the verification process.

**Returns:**

* A `SiwaResponse` object indicating the verification result.

#### Server Implementation Example

Here is an example implementation in a Node.js environment:

```javascript
import { SiwaMessage } from "@siwa/connect";

/**
 * Verifies the SIWA message on the server
 * @param {Object} payload - The payload from the client containing credentials
 * @returns {Object} - Verification result
 */
const verifySIWAMessageOnServer = async (payload) => {
  try {
    // Extract message and credentials from the payload
    const { message, signature, address, signature } = payload;

    // Initialize the SiwaMessage instance
    const siwaMessageInstance = new SiwaMessage(JSON.parse(message));

    // Define the parameters for verification
    const verifyParams = {
      signature: signature,
      domain: "yourdomain.com", // Replace with your app's domain
      address: address,
    };

    // Perform the verification
    const result = await siwaMessageInstance.verify(verifyParams);

    if (result.success) {
      console.log("SIWA message verified successfully");
      return { success: true, user: result.payload }; // Attach verified user data
    } else {
      throw result.error || new Error("Failed to verify SIWA message");
    }
  } catch (error) {
    console.error("Verification error:", error);
    return { success: false, error: error.message };
  }
};
```

#### Integration in an API Endpoint

To integrate this into your backend API, create an endpoint to handle SIWA verification requests:

**Example with Express.js**

```javascript
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

***

### Key Points to Remember

1. **Server-Side Security:** Always perform message verification on the server to maintain the integrity of authentication.
2. **SIWA Verify Method:** Leverage the `SiwaMessage.verify` method to ensure compliance with Algorand’s signature verification and credentials validation.
3. **Error Handling:** Implement robust error handling to provide meaningful feedback and ensure a secure flow.
4. **Domain Matching:** Ensure the `domain` parameter matches your application’s actual domain to prevent cross-origin attacks.

By integrating server-side SIWA verification, you create a secure and reliable authentication mechanism leveraging Algorand’s blockchain technology.
