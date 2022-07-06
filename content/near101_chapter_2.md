In this learning module, we will follow a tutorial to connect to a marketplace contract on the NEAR testnet.

### Prerequisites

- You should have created and deployed a NEAR smart contract for the marketplace as described in our [AssemblyScript contract development](/near101_chapter_1.md) learning module.
- [Node JS](https://nodejs.org/en/download/) - Please make sure you have Node.js v16 or higher installed.
- You should have a basic understanding of [React](https://reactjs.org/): know how to use JSX, props, state, lifecycle methods, and hooks.

### Tech Stack

We will use the following tech stack:

- [React](https://reactjs.org/) - A JavaScript library for building user interfaces.
- [near-api-js](https://docs.near.org/docs/api/javascript-library) - A JavaScript/Typescript library for interacting with NEAR's blockchain.

## 1. Project Setup

In the first section of this tutorial, we will set up the project and install the necessary dependencies.

Make sure that you have `nodejs` v16 or higher installed:

```bash
node -v
```

We will use the `create-react-app` utility to create a new react project:

```bash
npx create-react-app near-marketplace
```

Let's cd into the newly created project:

```bash
cd near-marketplace
```

Unfourtunately, `react-scripts` of version 5 might not work with the latest node version, so we should use `react-scripts` of version 4.0.3:

```bash
npm install react-scripts@4.0.3
```

We also need to install the `near-api-js` library:

```bash
npm install near-api-js
```

Finally, we will install the `uuid` library, which is used to generate unique IDs for our products:

```bash
npm install uuid
```

Thats it! Now we can start the project and see if everything is working:

```bash
npm start
```

## 2. Connecting to NEAR

Now that we have a project, we can set up our connection to the NEAR network and our smart contract.

### 2.1 Config

We create a `utils` folder in the `src` directory with the `src/utils/config.js` file to define the configuration for our connection to the NEAR network:

```js
const CONTRACT_NAME = process.env.CONTRACT_NAME || "${CONTRACT_NAME}"; // line 1

function environment(env) {
  switch (env) {
    case "mainnet": // line 5
      return {
        networkId: "mainnet",
        nodeUrl: "https://rpc.mainnet.near.org",
        contractName: CONTRACT_NAME,
        walletUrl: "https://wallet.near.org",
        helperUrl: "https://helper.mainnet.near.org",
        explorerUrl: "https://explorer.mainnet.near.org",
      };
    case "testnet": // line 14
      return {
        networkId: "testnet",
        nodeUrl: "https://rpc.testnet.near.org",
        contractName: CONTRACT_NAME,
        walletUrl: "https://wallet.testnet.near.org",
        helperUrl: "https://helper.testnet.near.org",
        explorerUrl: "https://explorer.testnet.near.org",
      };
    default:
      throw Error(`Unknown environment '${env}'.`);
  }
}

export default environment;
```

At line 1, we define the name of the smart contract that we want to interact with. This is the name of the account where the smart contract is deployed to.
We using the contract created in our [AssemblyScript contract development](/near101_chapter_1.md) learning module, so the `${CONTRACT_NAME}` variable would be `mycontract.myaccount.testnet`. Replace it with the accountID of the account where your smart contract is deployed to.

In lines 5 and 14, we define the different environments that we can connect to. In this tutorial, we will only use the testnet.

### 2.2 Connecting to NEAR

In this section, we will connect to the NEAR network and our smart contract.
Create a `src/utils/near.js` file. Let's do first our imports and define the environment:

```js
import environment from "./config";
import { connect, Contract, keyStores, WalletConnection } from "near-api-js";
import { formatNearAmount } from "near-api-js/lib/utils/format";

const nearEnv = environment("testnet");
//...
```

Now we create a function to initialize our contract:

```js
//...
export async function initializeContract() {
  const near = await connect(
    Object.assign(
      { deps: { keyStore: new keyStores.BrowserLocalStorageKeyStore() } },
      nearEnv
    )
  );
  window.walletConnection = new WalletConnection(near);
  window.accountId = window.walletConnection.getAccountId();
  window.contract = new Contract(
    window.walletConnection.account(),
    nearEnv.contractName,
    {
      viewMethods: ["getProduct", "getProducts"],
      changeMethods: ["buyProduct", "setProduct"],
    }
  );
}
//...
```

IMPORTANT NOTE

If you use a contract written in Rust, the initialization part should change names of the method so they have exactly the same names as in the contract itself:
```js
{
  viewMethods: ["get_product", "get_products"],
  changeMethods: ["buy_product", "set_product"],
}
```

We create a `near` object that we will use to interact with the NEAR network. It holds a `keyStore` object that stores the wallet information, which is stored in the browser's local storage.

Then we create a `WalletConnection` object that we will use to interact with the wallet. To sign in, sign out, get the account ID, and get the account balance.

We create a `Contract` object that we will use to interact with the smart contract. We pass the account, the name of the smart contract, and the methods that we want to use to the constructor.

When developing with NEAR, we don't need an ABI, as we need for Ethereum contracts. We can just use the `viewMethods` and `changeMethods` properties of the `Contract` object to define the methods we want to use. In this case, we have an array of `viewMethods` that don't modify the state (`["getProduct", "getProducts"]`) and an array of `changeMethods` that modify the state (`["buyProduct", "setProduct"]`).

Finally, we are going to create some functions to interact with the wallet:

```js
//...
export async function accountBalance() {
  return formatNearAmount(
    (await window.walletConnection.account().getAccountBalance()).total,
    2
  );
}

export async function getAccountId() {
  return window.walletConnection.getAccountId();
}

export function login() {
  window.walletConnection.requestSignIn(nearEnv.contractName);
}

export function logout() {
  window.walletConnection.signOut();
  window.location.reload();
}
```

In these functions, we use the `walletConnection` to get the account balance and the account ID, and we connect and disconnect our dapp from the wallet.

## 3. Implementing the Marketplace

Now that we have set up our connection to the NEAR blockchain and our smart contract, we can implement the marketplace functionality.

### 3.1 Marketplace Functionality

We create a `src/utils/marketplace.js` file, that will contain the functions that we will use to interact our smart contract:

```js
import { v4 as uuid4 } from "uuid";
import { parseNearAmount } from "near-api-js/lib/utils/format";

const GAS = 100000000000000;

export function createProduct(product) {
  product.id = uuid4();
  product.price = parseNearAmount(product.price + "");
  return window.contract.setProduct({ product }); // set_product for the Rust contract
}

export function getProducts() {
  return window.contract.getProducts(); // get_products for the Rust contract
}

export async function buyProduct({ id, price }) {
  await window.contract.buyProduct({ productId: id }, GAS, price); // buy_product for the Rust contract
}
```

With the `createProduct` function, we create a new product. We use the `uuid4` function to generate a unique ID for the product and the `parseNearAmount` function to convert the price to the correct format.
Then we use the `setProduct` function to create the product with the `product` object.

The `getProducts` function returns all the products stored in the smart contract.

Finally, we use the `buyProduct` function to buy a product. We pass the product ID, the amount of gas to use, and the product's price.

For the `GAS` constant we use the hardcoded value of `100000000000000`, which is 100 TGas(terra gas). If not all gas is used, the rest will be refunded to the account. If you want to know how you can calculate the approximate gas cost of a transaction, you can look at the [Near documentation](https://docs.near.org/docs/concepts/gas#the-cost-of-common-actions).

### 3.2 Adding the Marketplace to the App

We are going to add the functionality of the marketplace to our app. Open the `src/App.js` file and change the code to the following:

```js
import React, { useCallback, useEffect, useState } from "react";
import "./App.css";
import { getProducts } from "./utils/marketplace";
import { login } from "./utils/near";

function App() {
  const account = window.walletConnection.account();
  const [products, setProducts] = useState([]);
  const fetchProducts = useCallback(async () => {
    if (account.accountId) {
      setProducts(await getProducts());
    }
  });
  useEffect(() => {
    fetchProducts();
  }, []);
  return (
    <>
      {account.accountId ? (
        products.forEach((product) => console.log(product))
      ) : (
        <button onClick={login}>CONNECT WALLET</button>
      )}
    </>
  );
}

export default App;
```

This code is just for testing purposes. We try to connect to the wallet, and if we are connected, we fetch the products. Then we display the products in the console, making use of the `getProducts` function that we just created.
If we are not connected, we display a button that will connect to the wallet. For that we use the `login` utility function from the `utils/near.js` file.

### 3.2 Updating the index.js file

We also need to update the `src/index.js` file to intialize the contract:

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import { initializeContract } from "./utils/near";

window.nearInitPromise = initializeContract()
  .then(() => {
    ReactDOM.render(
      <React.StrictMode>
        <App />
      </React.StrictMode>,
      document.getElementById("root")
    );
  })
  .catch(console.error);

reportWebVitals();
```

Here we use the `initializeContract` utility function from the `utils/near.js` file to initialize the contract. We use the `window.nearInitPromise` variable to make sure that the app doesn't render until the contract is initialized.

Now you can start the app:

```bash
npm start
```

And you should see something like this:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/login_and_print_products_to_console.gif)

Great! Now you can see the products in the console. Continue in the next learning module to learn how to build the fronted components for your marketplace dapp.
