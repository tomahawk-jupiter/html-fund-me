# HTML Fund Me Notes

Blockchain Course Lesson 8 (vid 12:32:57). [Vid](https://www.youtube.com/watch?v=gyMwXuJrbJQ) and [repo](https://github.com/smartcontractkit/full-blockchain-solidity-course-js).

## Contents

- [Intro](#intro)
- [How Websites Work With Web3 Wallets](#how-websites-work-with-web3-wallets)
- [HTML](#html)
- [Install VScode extension](#install-vscode-extension)
- [Install http-server](#install-http-server)
- [Connecting HTML to Metamask](#connecting-html-to-metamask)
- [Connect Function](#connect-function)
- [Fund Function](#fund-function)
- [Ethers](#ethers)
- [Sending a Transaction From a Website](#sending-a-transaction-from-a-website)
- [Add Local Network](#add-local-network)
- [Import Account Into Metamask](#import-account-into-metamask)
- [Resetting Account](#resetting-account)
- [Listening For Events and Completed Transactions](#listening-for-events-and-completed-transactions)
- [Withdraw Function](#withdraw-function)

## Intro

People can programmatically interact with our smart contracts at any time once they are deployed to the blockchain. However, most users won't be developers so we need to create a UI for it, eg. a website.

We will build a minimalist website to show the functionality. This won't be best practice but is to illustrate what is going on.

All decentralized applications will have a way to connect wallets to it.

When building Dapps, you usually have 2 repos, the combination of which makes up the full stack:

- One for the smart contracts (backend)
- One for the front end / website (frontend)

We will need our hardhat-fund-me project open so we can interact with it with our frontend.

[Page Top](#html-fund-me-notes)

## How Websites Work With Web3 Wallets

We'll have a connect and an execute. Connect to connect wallet, execute to interact with a contract in some way.

When we have extensions in our browser like metamask, they automatically get injected into a window object in javaScript.

See it in sources -> page to see the metamask extension, I tried debug -> sources, didn't look the same as in vid, using different browser, also looking at different page, mine had a more stuff in it.

Type `window` into the console and you can see the window object.

Type in `window.ethereum`, this object only exists if you have a metamask or something similar.

**NOTE**: metamask wallet is for EVM, other wallets are for other kinds of blockchains. Type window.solana for the phantom wallet (which I don't have at the moment).

We can check for these objects in our javaScript. The reason these wallets are so important is that they, have under the hood, a blockchain node connected to them. We need a node to be able to interact with the blockchain.

Remember we used Alchemy before, you basically rent a node from services like this. This is a way to do it from the backend. But for the frontend we use the metamask wallet.

There are different wallets with different setups, but they all do the same thing, they expose some URL / node, that gives us the provider.

If you go into your metamask and look at the different networks you will see the RPC URL for each of them and a chainID. If you have a local network running you can also see that.

[Page Top](#html-fund-me-notes)

## HTML

Create the HTML.

### Install VScode extension

Live Server: ExtensionID: ritwickdey.LiveServer

There should be a button one the bottom panel called `Go Live`, or press `Ctrl + Shift + P` to open command palette and type `live server: Open with live server` and choose that command.

This will open up your file in the browser using the `localhost` also known as `loopback`.

When you save changes will automatically refresh.

You may get a `.vscode` folder that allows you to put settings for vscode for the current project.

**NOTE**: It didn't autogen this for me its `.vscode/settings.json` with `{ "liveServer.settings.port": 5501 }` in it, this just chooses the port to use.

### Install http-server

Install the http-server package `yarn add --dev http-server`.

Run with `$ yarn http-server`. This will do the same thing as the vscode extension.

This way requires re-running after making changes so the vscode extension is recommended.

[Page Top](#html-fund-me-notes)

## Connecting HTML to Metamask

[Metamask Docs](https://docs.metamask.io/guide/mobile-best-practices.html#the-provider-window-ethereum)

Check If User Has Wallet

### Connect metamask

[eth_requestAccounts](https://docs.metamask.io/guide/rpc-api.html#restricted-methods)

This line `window.ethereum.request({ method: "eth_requestAccounts" });` will make the metamask signin pop up.

Once you connect an account, you can open metamask and click on `connected`, this will show what the account is connected to. You can disconnect the account from here too.

This means the website can now make api calls to the account, you (accounts holder) still need to approve them.

### Connect Function

Any time we refresh the metamask will pop up again (if it is not connected). We don't want it to do this, its annoying.

We wrap the connect code in an async function so that it will only happen when the function is invoked.

We can use a onclick event on a button to run this connect function.

```javascript
async function connect() {
  if (typeof window.ethereum !== "undefined") {
    await window.ethereum.request({ method: "eth_requestAccounts" });
  } else {
    console.log("No metamask!");
  }
}
```

[Page Top](#html-fund-me-notes)

## JavaScript

We can put our javascript in its own file and add the src attribute to the script tag in the html file.

```html
<script src="./index.js" type="text/javascript"></script>
```

Imports work different on frontend, we won't use `require()`. More on this later.

### Add prettier

Create `.prettierrc` and put in your style rules. These rules will be used if you have vscode extension, but for people who don't (who might clone our repo), we can add the prettier package `yarn add --dev prettier`.

**NOTE**: I didn't add this.

[Page Top](#html-fund-me-notes)

## Fund Function

To send a transaction what do we need?

1. Provider / connection to blockchain
1. Signer / wallet / someone with some gas
1. The contract we are interacting with
   - Its ABI
   - Its Address

### Ethers

To get the provider we are going to work with ethers again but this time a little differently.

Before we did `const { ethers } = require("ethers")`, but we can't use require in the frontend and we don't want to install ethers as a package.

**NOTE**: Later on in other lessons we will install ethers like before and the frameworks we'll use will deal with it.

[Ethers Getting Started](https://docs.ethers.io/v5/getting-started/)

We can copy the ethers library frontend version into a file in our project and import it.

```javascript
import { ethers } from "./ethers-5.6.esm.min.js";
```

Change the script in the html to a module type `<script src="./index.js" type="module"></script>`, this will allow us to import things into the `index.js` file.

### Fix Buttons

The onclick need to be in the javascript file, changing the script type to module stopped them from working.

You now have access to the ethers object, `console.log(ethers)` will show it in the console.

[Page Top](#html-fund-me-notes)

## Sending a Transaction From a Website

### Provider

[Web3Provider](https://docs.ethers.io/v5/api/providers/other/#Web3Provider) is an object in ethers that allows us to wrap around stuff like `metamask`, it is similar to that [JsonRpcProvider](https://docs.ethers.io/v5/api/providers/jsonrpc-provider/) we used before to put in exactly that endpoint, our alchemy endpoint, or when working with Metamask whatever endpoint we have in our network section.

The Web3Provider takes that HTTP endpoint and automatically sticks it in ethers for us. So this line of code:

```javascript
const provider = new ethers.providers.Web3Provider(window.ethereum);
```

Looks at our Metamask, finds the HTTP endpoint, and uses that as the provider.

### Signer

Since our provider is connected to our Metamask, we can get a signer or a wallet by running:

```javascript
const signer = provider.getSigner();
```

This will return whatever wallet / account is connected from the provider (the metamask).

We can `console.log(signer)` to see the signer object.

### Contract

Next we need the contract. We need the address and the ABI.

Once deployed, the address of a contact isn't going to change so you'll have some kind of constants file eg. `constants.js`, and in here will be the addresses and any ABIs.

This is where the frontend and backend is going to need to interact a bit.

### ABI

We'll go back to our `hardhat-fund-me` project. Go to `artifacts/contracts/FundMe.sol/FundMe.json`, and in here will be a section for the `abi`. We copy that section (its an array), put it in our constants file and export it.

```javascript
export const abi = [
  /* copied abi */
];
```

Then import into the main `index.js` file:

```javascript
import { abi } from "./constants.js";
```

Now we have the `ABI`!

### Address

Since we'll be running this locally, we need to get the locally run address for the contract.

Start the backend node from your frontend projects terminal, eg:

```
$ cd ..
$ cd hardhat-fund-me
$ yarn hardhat node
```

Then copy the address for the contract into the constants file `export const contractAddress = "0x4324...";` and import into the index.js `import { contractAddress } from "./constants.js";`. Address is where it says deployed at 0x....

**NOTE**: The address seems to always be the same when the node is restarted.

Now we have the contract address!

### Put It Together

This will give us a contract object that is connected to our signer:

```javascript
const contract = new ethers.Contract(contractAddress, abi, signer);
```

Now we have the contract object, we can make transactions just like before:

```javascript
const transactionResponse = await contract.fund({
  value: ethers.utils.parseEther(ethAmount),
});
```

## Add Local Network

There is a localhost network in our Metamask, but we can be more specific and add a new one.

1. Go to `Add Network`
1. Give it a name, eg. `hardhat-localhost`.
1. Get the `RPC URL` from the node running in your terminal where it says something like `Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545`.
1. ChainID will be 31337.
1. Currency Symbol is ETH or GO.
1. Block Explorer URL leave empty because its a local chain

## Import Account Into Metamask

We won't have any ETH for the local blockchain. We can import one of the fake accounts into our Metamask. You can do this for any account with a private key.

To import the private key from the fake account into your metamask:

1. In your Metamask, click the colorful button in the top right corner
1. Click import account
1. Select type private key
1. Past private key (from fake account)

**NOTE**: If you choose json file instead of private key, remember how we encrpted our private key into a json file with a password, we can use that.

### Make Transaction

Once we do this, we'll have ETH for our local blockchain and we can use the fund function. When you click fund, Metamask will popup and ask for confirmation.

Look at the node terminal to see the transaction details after its sent.

**NOTE**: Just hard code the ethAmount in the index.js fund function for now.

**ERROR**: The metamask sends the ETH but the node says "You need to spend more ETH!" even though is should be enough. Other people have this problem in the discussion. The ETH isn't getting sent back to the account either. I uncommented the recieve() and fallback() functions in the FundMe contract, before I did this there was some other error going on. **The contract does recieve the ETH though**.

## Resetting Account

If you restart your node, the nonce won't match with the account, they will be out of sync. Fix this by resetting the account. Resetting the account only effects the current network.

1. Click colorful button
1. Settings
1. Advanced
1. Reset account

**NOTE**: You wouldn't want to do this on a real network.

[Page Top](#html-fund-me-notes)

## Listening For Events and Completed Transactions

### Create a function that returns a promise

```javascript
function listenForTransactionMine(transactionResponse, provider) {
  console.log(`Mining ${transactionResponse.hash}...`);
  return new Promise();
}
```

And we can await it in another function (fund):

```javascript
await listenForTransactionMine(transactionResponse, provider);
```

This is how to write a new promise:

```javascript
return new Promise((resolve, reject) => {
  provider.once(transactionResponse.hash, (transactionReceipt) => {
    console.log(
      `Completed with ${transactionReceipt.confirmations} confirmations`
    );
    resolve();
  });
});
```

We only resolve after the event is fired, this is done by putting the `resolve()` inside the callback function. We are waiting for the transaction to finish.

The `reject` would normally involve a timeout.

[Ethers contract.once](https://docs.ethers.io/v5/api/contract/contract/#Contract-once)

### Input Forms

Make an input for ETH amount.

[Page Top](#html-fund-me-notes)

## Withdraw Function

This is similar to the other functions. We can reuse the listener we wrote before.

```javascript
async function withdraw() {
  if (typeof window.ethereum != "undefined") {
    console.log("Withdrawing funds...");
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const signer = provider.getSigner();
    const contract = new ethers.Contract(contractAddress, abi, signer);
    try {
      const transactionResponse = await contract.withdraw();
      await listenForTransactionMine(transactionResponse, provider);
    } catch (error) {
      console.log(error);
    }
  }
}
```

**ERROR**: Withdraw doesn't work. Error mentions gas limit, I looked it up and it mentioned signer not matching.

[Page Top](#html-fund-me-notes)
