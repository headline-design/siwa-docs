---
description: >-
  This guide will walk you through the process of integrating Sign-In with
  Algorand (SIWA) into a Next.js application using NextAuth.js.
---

# NextAuth.js

{% embed url="https://github.com/headline-design/algostack-ssr" %}
Get started quickly with a full-stack SIWA template
{% endembed %}

## Prerequisites

Before you begin, make sure you have the following installed:

* [Node.js](https://nodejs.org/en/)
* [Yarn](https://classic.yarnpkg.com/en/docs/install) or npm
* An Algorand wallet (e.g., Pera, Defly, etc)

### Getting Started

1. Clone the NextAuth.js example repository:

```bash
git clone https://github.com/nextauthjs/next-auth-example
cd next-auth-example
```

## Integrating SIWA with NextAuth.js

This guide will walk you through the process of integrating Sign-In with Algorand (SIWA) into a Next.js application using NextAuth.js.

### Prerequisites

Before you begin, make sure you have the following installed:

* [Node.js](https://nodejs.org/en/)
* [Yarn](https://classic.yarnpkg.com/en/docs/install) or npm
* An Algorand wallet (e.g., Pera, Defly)

### Getting Started

1. Clone the NextAuth.js example repository:

```bash
git clone https://github.com/nextauthjs/next-auth-example
cd next-auth-example
```

2. Create a `.env.local` file with the following content:

```plaintext
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=yoursecretkey
```

3. Install the required dependencies:

```shellscript
yarn add @avmkit/siwa algosdk
```

### Configuring NextAuth.js

Modify the `pages/api/auth/[...nextauth].ts` file to include the SIWA provider:

```typescript
import NextAuth from "next-auth"
import CredentialsProvider from "next-auth/providers/credentials"
import { getCsrfToken } from "next-auth/react"
import { SiwaMessage } from "@avmkit/siwa"

export default async function auth(req: any, res: any) {
  const providers = [
    CredentialsProvider({
      name: "Algorand",
      credentials: {
        message: {
          label: "Message",
          type: "text",
          placeholder: "0x0",
        },
        signature: {
          label: "Signature",
          type: "text",
          placeholder: "0x0",
        },
      },
      async authorize(credentials) {
        try {
          const siwa = new SiwaMessage(JSON.parse(credentials?.message || "{}"))
          const nextAuthUrl = new URL(process.env.NEXTAUTH_URL)

          const result = await siwa.verify({
            signature: credentials?.signature || "",
            domain: nextAuthUrl.host,
            nonce: await getCsrfToken({ req }),
          })

          if (result.success) {
            return {
              id: siwa.address,
            }
          }
          return null
        } catch (e) {
          return null
        }
      },
    }),
  ]

  // ... rest of the NextAuth configuration
}
```

### Creating a SIWA Sign-In Page

Create a new file `pages/siwa.tsx` for the SIWA sign-in page:

```typescript
import { getCsrfToken, signIn, useSession } from "next-auth/react"
import { SiwaMessage } from "@avmkit/siwa"
import { useEffect, useState } from "react"
import Layout from "../components/layout"

function Siwa() {
  const [address, setAddress] = useState("")
  const { data: session, status } = useSession()

  const handleLogin = async () => {
    try {
      const callbackUrl = "/protected"
      const message = new SiwaMessage({
        domain: window.location.host,
        address: address,
        statement: "Sign in with Algorand to the app.",
        uri: window.location.origin,
        version: "1",
        chainId: "416001", // Algorand TestNet
        nonce: await getCsrfToken(),
      })
      
      // Here, you would use your Algorand wallet library to sign the message
      // For example, with Pera:
      // const signMessageWithPera = async (message, address) => {
      // const encodedMessage = prepareMessage(message);
      // const signatureArray = await peraWallet.signData(
      // [{ data: encodedMessage, message: "" }],
      // address );
      // return signatureArray[0]; // Returns the signature
      // };

      // For this example, we'll use a placeholder signature
      const signature = "placeholder_signature"

      signIn("credentials", {
        message: JSON.stringify(message),
        redirect: false,
        signature,
        callbackUrl,
      })
    } catch (error) {
      console.error(error)
      alert("An error occurred during sign-in")
    }
  }

  return (
    <Layout>
      <h1>Sign-In with Algorand</h1>
      <input
        type="text"
        placeholder="Enter Algorand address"
        value={address}
        onChange={(e) => setAddress(e.target.value)}
      />
      <button onClick={handleLogin}>Sign-in with Algorand</button>
    </Layout>
  )
}

export default Siwa
```

### Updating the Header Component

Modify the `components/header.tsx` file to include a link to the SIWA page:

```typescript
import Link from "next/link"
import styles from "./header.module.css"

export default function Header() {
  // ... existing header code

  return (
    <header>
      {/* ... existing header content */}
      <nav>
        <ul className={styles.navItems}>
          <li className={styles.navItem}>
            <Link href="/">
              Home
            </Link>
          </li>
          <li className={styles.navItem}>
            <Link href="/siwa">
              SIWA
            </Link>
          </li>
        </ul>
      </nav>
    </header>
  )
}
```

### Running the Application

Start the development server:

```shellscript
yarn dev
```

Navigate to `http://localhost:3000` and click on the "SIWA" link in the header to test the Sign-In with Algorand functionality.

### Next Steps

This guide provides a basic integration of SIWA with NextAuth.js. For a production-ready implementation, you should:

1. Implement proper Algorand wallet integration for message signing.
2. Add error handling and user feedback.
3. Customize the UI to match your application's design.
4. Implement proper session management and protected routes.

Remember to always follow best practices for security when dealing with authentication in your application.
