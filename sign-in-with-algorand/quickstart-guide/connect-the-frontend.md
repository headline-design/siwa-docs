---
description: Run tests and troubleshoot SIWA Connect
---

# Testing and Troubleshooting

### Common Issues and Solutions

#### Issue 1: Connection Error

* **Description**: Unable to connect to the wallet.
* **Solution**: Ensure your wallet app is updated to the latest version and check your internet connection. Try clearing browser cache and reloading the page.

#### Issue 2: Signature Request Fails

* **Description**: Signature request is not approved.
* **Solution**: Double-check that you are signed into the correct wallet in the app. Verify wallet permissions and restart the application if needed.

#### Issue 3: SIWA Verification Error

* **Description**: SIWA message verification fails.
* **Solution**: Confirm the integrity of the backend service handling the SIWA message. Check server logs for detailed error messages and ensure the server is correctly configured to handle authentication requests.

Navigate to `http://localhost:3000` in your browser.

### Testing the Authentication Flow

1. Click on either the "Connect Pera Wallet" or "Connect Defly Wallet" button.
2. Approve the connection in your wallet app.
3. Once connected, click the "Sign In" button.
4. Approve the signature request in your wallet app.
5. Click the "Verify SIWA Message" button.
6. If successful, you should see a success message.

### Common Issues and Solutions

#### Wallet Not Connecting

If you're having trouble connecting your wallet:

1. Ensure your wallet app is installed and set up correctly.
2. Check that you're on the correct network (testnet or mainnet) in your wallet app.

#### Signature Verification Failing

If the signature verification is failing:

1. Check the console for any error messages.
2. Ensure that the `chainId` in the `SiwaMessage` creation matches your current network.
3. Verify that the `nonce` is being generated uniquely for each request.

#### UI Not Updating Correctly

If the UI is not updating as expected:

1. Check that all state variables are being updated correctly in the `SIWAConnect` component.
2. Verify that the `activeStep` state is being set correctly at each stage of the process.

### Debugging Tips

1. Use `console.log` statements to track the flow of data through your application.
2. Utilize the React Developer Tools browser extension to inspect component state and props.
3. Check the Network tab in your browser's developer tools to ensure API requests are being made correctly.

### Getting Help

If you're still having issues:

1. Check the [SIWA documentation](https://siwa.oeg) for any updates or known issues.
2. Post your question on the [Algorand Developer Forum](https://forum.algorand.org/).
3. Join the [Algorand Discord](https://discord.gg/algorand) and ask for help in the appropriate channel.

Remember to always provide as much detail as possible about your issue, including any error messages, when seeking help.
