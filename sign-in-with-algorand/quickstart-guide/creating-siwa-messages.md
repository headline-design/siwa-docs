---
description: Clone/fork the SIWA Connect repo to get started
---

# Setting Up the Project

### Clone the Repo

```bash
git clone https://github.com/headline-design/siwa-connect.git
cd siwa-connect
```

{% embed url="https://github.com/headline-design/siwa-connect" %}
Get started quickly with SIWA Connect
{% endembed %}

Install the necessary dependencies:

```bash
npm install
# or
yarn
```

### Environment Setup

Create a `.env.local` file in the root of your project and add the following:

```
NEXT_PUBLIC_ALGOD_SERVER=https://testnet-api.algonode.cloud
NEXT_PUBLIC_ALGOD_PORT=443
NEXT_PUBLIC_ALGOD_TOKEN=
```

These environment variables are used to connect to the Algorand testnet. You can adjust these values if you're using a different network or your own node.

### Project Structure

The project structure should look like this:

```plaintext
siwa-connect/
├── components/
│   ├── SIWAConnect.tsx
│   ├── Button.tsx
│   ├── Card.tsx
│   ├── Alert.tsx
│   └── ...
├── hooks/
│   ├── useSIWAAccount.ts
│   └── useWalletConnection.ts
├── pages/
│   ├── _app.tsx
│   └── index.tsx
├── utils/
│   └── siwaUtils.ts
├── .env.local
├── package.json
└── tsconfig.json
```

In the next section, we'll dive into implementing the SIWA Connect functionality.
