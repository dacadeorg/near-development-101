This learning module consists of a tutorial, where you will learn how to create a smart contract for a decentralized marketplace on the NEAR blockchain.

We will use AssemblyScript to write our NEAR smart contracts. Writing AssemblyScript code is very similar to writing TypeScript code. But there are a few things specific to writing NEAR smart contracts in AssemblyScript, that we will cover in the following sections.

### Prerequisites

- Have some basic knowledge of Blockchain technology and smart contracts.
- Have some basic JavaScript knowledge.
- Be comfortable using a terminal.

### Tech Stack

We will use the following tech stack:

- [near-cli](https://www.npmjs.com/package/near-cli) - A CLI tool for NEAR that offers an API to interact with smart contracts.
- [assemblyscript](https://www.npmjs.com/package/assemblyscript) - A TypeScript-like language for WebAssembly.
- [asbuild](https://www.npmjs.com/package/asbuild) - A build tool for AssemblyScript.

## 1. Setup

In this first section of the tutorial, we will set up our development environment and project.

We will need to have a version of node.js higher or equal to 12 installed.
We will also use the package manager yarn, so make sure to have that installed.

### 1.1 Install CLI Tools

You can install the latest versions of CLI tools globally by running the following commands:

```bash
yarn global add near-cli
yarn global add assemblyscript
yarn global add asbuild
```

We recommend using a code editor that supports code completion and syntax highlighting for TypeScript to help you write AssemblyScript code, like Visual Studio Code or Atom. We will use Visual Studio Code for this learning module.

### 1.2 Project Setup

Now we will set up our project. Please create a new root directory for our project and call it something like `near-marketplace-contract`.

#### 1.2.1 `asconfig.json`

In our root directory, we need to create a file called `asconfig.json`. This config file provides CLI options and configurations for AssemblyScript.

To compile with the `near-sdk-as`, we will need to add the following to the `asconfig.json` file:

```javascript
{
    "extends": "near-sdk-as/asconfig.json"
}
```

#### 1.2.2 `tsconfig.json`

We create an `assembly` directory and the `assembly/tsconfig.json` file inside it. This file aims to specify compiler options and root level files that are necessary to compile TypeScript projects.

The `tsconfig.json` file needs to be in the root directory of the TypeScript project.

```javascript
{
  "extends": "../node_modules/assemblyscript/std/assembly.json",
  "include": [
    "./**/*.ts"
  ]
}
```

#### 1.2.3 `as_types.d.ts`

We create the `assembly/as_types.d.ts` file in our `assembly` directory. This file is used to define types that are used in our AssemblyScript code.
In this case, we import the types from the `near-sdk-as`.

```javascript
/// <reference types="near-sdk-as/assembly/as_types" />
```

### 1.3 Initialize project

Run the following command to initialize the project in the project root directory:

```bash
yarn init
```

It will create a package.json file where the dependencies are listed.

Run the following command to add the `near-sdk-as` to the project:

```bash
yarn add -D near-sdk-as
```

Mac M1 users might experince the `Error: Unsupported platform: Darwin arm64` error when attempt to install.
As a workaround, run the previous command with the `--ignore-scripts` option:
```
yarn add -D --ignore-scripts near-sdk-as
```

In the last step of the initialization process, we will need to create an entry file for the smart contract. Create a file called `index.ts` in the `assembly` directory.

The final project structure should look like this:

```
├── asconfig.json
├── assembly
│   ├── as_types.d.ts
│   ├── index.ts
│   └── tsconfig.json
├── package.json
└── yarn.lock
```

## 2. Contract Storage

In this section we will learn how to store data in the smart contract.

### 2.1 Storage

Our smart contracts will need to store data on the blockchain. NEAR offers different storage options depending on the use case and the data type.

We will use the `Storage` class from the `near-sdk-as`, which offers a great API to store and retrieve data from the blockchain. We can choose between the following collection types:

- `PersistentMap` - a key-value data structure.
- `PersistentVector` - an array-like data structure.
- `PersistentDeque` - a bidirectional queue.
- `PersistentUnorderedMap` - similar to `PersistentMap` but with useful additional functions that allow us to iterate over keys, values, entries.

You can learn more about storage and collections in the [NEAR documentation](https://docs.near.org/docs/develop/contracts/as/intro#state-and-data).

### 2.2 Read and Write State

There are two types of function calls, `view` and `change`. Calls that are `view` will only read data from the blockchain, while calls that are `change` will write data to the blockchain and modify its state. Calls that are `view` are free, while we need to pay gas for `change` calls.

For instance, calling `PersistentUnorderedMap#getSome(key: K)` and retrieving data behind the corresponding key won't cost gas. While calling `PersistentUnorderedMap#setSome(key: K, value: V)` and adding a new key-value pair will cost gas.

## 3. Read and Write Contract

In this section of the tutorial, we are going to write a simple smart contract that will store and retrieve data from the blockchain.

We will write this contract in our `assembly/index.ts` file.

Let's start by importing the `PersistentUnorderedMap` class from the `near-sdk-as` at the top of the file:

```javascript
import { PersistentUnorderedMap } from "near-sdk-as";
```

Next, we create a `PersistentUnorderedMap` instance to store our products:

```javascript
export const products = new PersistentUnorderedMap<string, string>("PRODUCTS");
```

We create a constant variable called `products` that is an `PersistentUnorderedMap` that will map product ids of type `string` to product names of type `string`.

The string `PRODUCTS` in the `PersistentUnorderedMap`'s constructor is the unique prefix to use for every key.

### 3.1 Write Function

Lets create a function to add a new product to the `products` map:

```javascript
export function setProduct(id: string, productName: string): void {
    products.set(id, productName);
}
```

Functions that need to be invoked from the outside of the smart contract need to be exported by adding the `export` keyword.

Our `setProduct` function needs two parameters: `id` and `productName`. The `id` parameter is the key of the product, and the `productName` parameter is the value of the product.

We need to specify the return type of the function, which is `void`, since we don't need to return anything.

To create a new entry to our products mapping we just need to call the `set` function on the mappings instance (`products`) and pass the key and value as parameters.

### 3.2 Read Function

To finish our first smart contract, we need to create a function to retrieve a product from the `products` map.

```javascript
export function getProduct(id: string): string | null {
    return products.get(id);
}
```

We export the `getProduct` function and just have one parameter `id`, which is the key of the product we want to retrieve.

The return type of the function is `string | null`, since we can return either a product name or `null` if the product doesn't exist.

In the function body we call the `get` function on the `products` mapping, pass the key as a parameter and return the value.

The final code for this section looks like this:

```javascript
import { PersistentUnorderedMap } from "near-sdk-as";

export const products = new PersistentUnorderedMap<string, string>("PRODUCTS");

export function setProduct(id: string, productName: string): void {
    products.set(id, productName);
}

export function getProduct(id: string): string | null {
    return products.get(id);
}
```

**Warning**: The key for the persistent collection should be as short as possible to reduce storage space because this key will be repeated for every record in the collection. Here, we only used the longer `PRODUCTS` key to add more readability for first-time NEAR developers.

## 4. Create Accounts

In order to test our smart contracts on the NEAR testnet, we need to create two accounts.

We will create one account that we will use to interact with the smart contract, and another account that we will deploy the smart contract to.
The second account will be a subaccount of the first account, and will look like a subdomain.

For this learning module we will use `myaccount.testnet` as the first account and create a subaccount of it called `mycontract.myaccount.testnet`, where we are going to deploy our smart contract to.

### 4.1 Create a Top-Level Account

Go to the [NEAR Testnet wallet](https://wallet.testnet.near.org/) page and create a new account by following these steps:

1. Open the wallet.
2. Choose a name for your account.
3. Choose a security method, we are going to choose passphrase for this learning module.
4. Store the passphrase somewhere safe.
5. Reenter the passphrase to confirm.

Here is a GIF showing the steps above:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/create_account.gif)

Now your test account is created and you should be able to use it. Your account will already have a balance of NEAR testnet tokens, so you don't need to use a faucet.

Next we will create a subaccount using the `near-cli`.

### 4.2 Login to the NEAR CLI

To log into your new account, open a terminal and run the following `near-cli` command:

```bash
near login
```

This command should open a new tab in your browser and ask you to log into your NEAR account. You will be asked to grant permmissions to the `near-cli` to access your account.

Now you are authenticated for this session and you can make transactions, like deployment and contract interaction calls via the `near-cli`.

Here is a GIF showing the steps above:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/login_to_shell.gif)

### 4.3 Create a Subaccount

To create a subaccount for your account, run the following command:

```bash
near create-account ${SUBACCOUNT_ID}.${ACCOUNT_ID} --masterAccount ${ACCOUNT_ID} --initialBalance ${INITIAL_BALANCE}
```

- `SUBACCOUNT_ID` - id of the subaccount.
- `ACCOUNT_ID` - id of the top-level account.
- `INITIAL_BALANCE` - initial balance for the subaccount in NEAR tokens.

As stated earlier we want to use the subaccount to deploy our smart contract to. So in our case an example call to create a subaccount could look like this:

```bash
near create-account mycontract.myaccount.testnet --masterAccount myaccount.testnet --initialBalance 5
```

## 5. Compile and Deploy

In this section, we will compile our simple smart contract and deploy it to the NEAR testnet.

### 5.1 Compile the Contract

Before we can deploy our smart contract to the NEAR testnet, we need to compile it to wasm code. To compile our contract we need to run the following command in the project root:

```bash
yarn asb
```

The compiled wasm code will be store in a file called `${CONTRACT_NAME}.wasm` in the following directory:

```bash
${PROJECT_ROOT}/build/release/${CONTRACT_NAME}.wasm
```

`${CONTRACT_NAME}` refers to the name of your project, that you can find in your package.json file.

### 5.2 Deploy the Contract

To deploy our smart contract to the NEAR testnet, we need to run the following command:

```bash
near deploy --accountId=${ACCOUNT_ID} --wasmFile=${PATH_TO_WASM}
```

- `${ACCOUNT_ID}` - the id of the account that will deploy the smart contract to.
- `${PATH_TO_WASM}` - the path to the `.wasm` file that contains the compiled smart contract.

In our case the deploy command could look like this:

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=build/release/near-marketplace-contract.wasm
```

Now our contract is deployed to the NEAR testnet and we can interact with it.

## 6. Contract Calls

In this section of the learning module, we will call the functions on the deployed smart contract.

As stated at the beginning of the learning module, there are two types of function calls that we can make: `view` and `change`.

### 6.1 Calling a Change Function

First, we are going to invoke a `change` function.

The `change` contract call in the `near-cli` looks like this:

```bash
near call ${CONTRACT_ACCOUNT_ID} ${METHOD_NAME} ${PAYLOAD} --accountId=${ACCOUNT_ID}
```

- `${CONTRACT_ACCOUNT_ID}` - the id of the account that contains the smart contract.
- `${METHOD_NAME}` - the name of the function that we want to call.
- `${PAYLOAD}` - the payload that will be passed to the function.
- `${ACCOUNT_ID}` - the id of the account that will make the call.

If want to add a new product to the contract we just deployed, the call could look like this:

```bash
near call mycontract.myaccount.testnet setProduct '{"id": "0", "productName": "tea"}' --accountId=myaccount.testnet
```

If you use PowerShell or CMD on Windows, you might need to escape double quotes in the payload, which could look like this:

```bash
near call mycontract.myaccount.testnet setProduct "{\"id\": \"0\", \"productName\": \"tea\"}" --accountId=myaccount.testnet
```

Keep this in mind for the following sections when we have double quotes in the payload.

If your contract call was successful, you will see something similar to the following output:

```
Scheduling a call: mycontract.myaccount.testnet.setProduct({"id": "0", "productName": "tea"})
Doing account.functionCall()
Transaction Id ${TRANSACTION_ID}
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.testnet.near.org/transactions/${TRANSACTION_ID}
''
```

### 6.2 Calling a View Function

Now that we have added a product to the contract, we can call the `view` function to retrieve the product.

The `view` contract call in the `near-cli` looks like this:

```bash
near view ${CONTRACT_ACCOUNT_ID} ${METHOD_NAME} ${PAYLOAD}
```

Since we don't need to pay any gas we can omit the account id at the end.

If we want to retrieve the product we just added, the call could look like this:

```bash
near view mycontract.myaccount.testnet getProduct '{"id": "0"}'
```

If your contract call was successful, you will see something similar to the following output:

```
View call: mycontract.myaccount.testnet.getProduct({"id": "0"})
'tea'
```

Now you are able to compile and deploy your contract to the NEAR testnet and interact with it.

## 7. Contract with Product Model

In this section, we are going to write a second iteration of our contract that can store more than just a string.

### 7.1 Create Product Model

We are going to use a model called a `Product` to represent our products, because we want to be able to store more than just the name of the product. A model is a custom data container that defines a new type and consists of an AssemblyScript class.

In our assembly directory we are going to create a new file called `assembly/model.ts`.

The content of the file should look like this:

```javascript
import { PersistentUnorderedMap, u128, context } from "near-sdk-as";

@nearBindgen
export class Product {
    id: string;
    name: string;
    description: string;
    image: string;
    location: string;
    price: u128;
    owner: string;
    sold: u32;
    public static fromPayload(payload: Product): Product {
        const product = new Product();
        product.id = payload.id;
        product.name = payload.name;
        product.description = payload.description;
        product.image = payload.image;
        product.location = payload.location;
        product.price = payload.price;
        product.owner = context.sender;
        return product;
    }
    public incrementSoldAmount(): void {
        this.sold = this.sold + 1;
    }
}

export const listedProducts = new PersistentUnorderedMap<string, Product>("LISTED_PRODUCTS");
```

We use the `@nearBindgen` decorator to serialize our custom class before storing it on the blockchain.

Our `Product` class consists of the fields for the `id`, `name`, `description`, `image`, `location` and `owner` of the product, which are all strings. We also have a `price` field which is a 128 bit unsigned integer, and a `sold` field which is a 32 bit unsigned integer. This numeric data types are specific to NEAR and are recommended to use instead of the TypeScript numeric types to avoid issues with data type conversion.

The `u128` `price` field allows us to store the NEAR price in [yocto](https://www.nanotech-now.com/metric-prefix-table.htm). 1 yocto-NEAR = 10<sup>-24</sup> NEAR, which is the smallest unit of NEAR.

Our class also consists of a static method called `fromPayload` which takes a payload and returns a new `Product` object, and a method called `incrementSoldAmount` which we are going to use later to increment the `sold` value after a product has been sold.

The `context` object contains addtional information about the transaction. In this case, we are using `context.sender` to retrive the account id of the account that is calling the function.

We also create a new map called `listedProducts` which is a persistent unordered map and will replace the `products` map that we previously stored in our `index.ts` file. We could also migrate the data from the `products` map to the `listedProducts` map, but this is an advanced topic that is out of scope for this learning module.

As explained earlier, we will use a more readable version of a key for `LISTED_PRODUCTS` to provide more readability to our code. To reduce storage space, you should pick a shorter key.

### 7.2 Update `index.ts`

We need to update our `assembly/index.ts` file to include the new model and map:

```javascript
import { Product, listedProducts } from './model';

export function setProduct(product: Product): void {
    let storedProduct = listedProducts.get(product.id);
    if (storedProduct !== null) {
        throw new Error(`a product with ${product.id} already exists`);
    }
    listedProducts.set(product.id, Product.fromPayload(product));
}

export function getProduct(id: string): Product | null {
    return listedProducts.get(id);
}

export function getProducts(): Product[] {
    return listedProducts.values();
}
```

First, we import the `Product` class and the `listedProducts` map, which is a new data structure that we added to the `model.ts` file after we removed `products` map.

Then we modify the `setProduct` function to adjust it to use the new `Product` class and the `listedProducts` map. We first check if the product id already exists in the map. If it does, we throw an error. Otherwise, we call the `fromPayload` method to create a new `Product` object from the payload and store it in the `listedProducts` map.

We also modify the `getProduct` function to adjust it to use the new `Product` class and the `listedProducts` map.

Finally, we create the `getProducts` function to return all products in the map.

### 7.3 Contract Redeployment

NEAR allows us to update our contract code on the blockchain. We can do this by redeploying the contract.

We need to compile our new contract first:

```bash
yarn asb
```

Then we can redeploy the contract to the same account id as before:

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=build/release/${WASM_FILE_NAME}
```

Let's add a new product to the contract by calling the `setProduct` function. Since we are using the `Product` class, we need to pass in a payload that is a `Product` object, which could look like this:

```bash
near call mycontract.myaccount.testnet setProduct '{"product": {"id": "0", "name": "BBQ", "description": "Grilled chicken and beef served with vegetables and chips.", "location": "Berlin, Germany", "price": "1000000000000000000000000", "image": "https://i.imgur.com/yPreV19.png"}}' --accountId=myaccount.testnet
```

After a successful `setProduct` call, we can call the `getProduct` function to retrieve the product we just added:

```bash
near view mycontract.myaccount.testnet getProduct '{"id": "0"}'
```

You should an output similar to the following:

```
View call: mycontract.myaccount.testnet.getProduct({"id": "0"})
{
  id: '0',
  name: 'BBQ',
  description: 'Grilled chicken and beef served with vegetables and chips.',
  location: 'Berlin, Germany',
  price: '1000000000000000000000000',
  image: 'https://i.imgur.com/yPreV19.png',
  owner: 'myaccount.testnet',
  sold: 0
}
```

That's it! We have successfully added a new product to our contract.

## 8. Contract with Buy Function

For the final section of this tutorial, we will create the `buyProduct` function to allow a user to buy a product.

The `near-sdk-as` library provides a `ContractPromiseBatch` class which allows us to batch actions within an AssemblyScript contract. We will use this class to transfer tokens from the caller of the function to the owner of the product.

The code for a token transfer looks like this:

```javascript
ContractPromiseBatch.create(${RECEIVING_ACCOUNT}).transfer(${DEPOSIT});
```

We previously mentioned that the `context` object contains information about a transaction. For our `buyProduct` function we will use `context.attchedDeposit` to get the amount of tokens that the caller of the function has attached to the transaction.

Let's write our `buyProduct` function in our `index.ts` file.

First, we import the `ContractPromiseBatch` and `context` from the `near-sdk-as` library at the top of the file:

```javascript
import { Product, listedProducts } from './model';
import { ContractPromiseBatch, context } from 'near-sdk-as';
```

Now we can write our `buyProduct` function at the bottom of the file:

```javascript
export function buyProduct(productId: string): void {
    const product = getProduct(productId);
    if (product == null) {
        throw new Error("product not found");
    }
    if (product.price.toString() != context.attachedDeposit.toString()) {
        throw new Error("attached deposit should equal to the product's price");
    }
    ContractPromiseBatch.create(product.owner).transfer(context.attachedDeposit);
    product.incrementSoldAmount();
    listedProducts.set(product.id, product);
}
```

First, we retrieve the product with the specified id through the `getProduct` function.

Then we check if the product exists. If it doesn't, we throw an error ("product not found"). Otherwise, we check if the attached deposit is equal to the product's price. If it isn't, we throw an error ("attached deposit should equal to the product's price").

We then create a new `ContractPromiseBatch` object and call the `transfer` method on it. This method takes the amount of tokens that the caller of the function has attached to the transaction and transfers them to the owner of the product. We get the account of the owner of the product by accessing the `owner` property of the product we retrieved.

Finally, we increment the `sold` field of the product by calling the `incrementSoldAmount` function and update the product in the `listedProducts` map.

This is it! The final contract should look like this:

```javascript
import { Product, listedProducts } from './model';
import { ContractPromiseBatch, context } from 'near-sdk-as';

export function setProduct(product: Product): void {
    let storedProduct = listedProducts.get(product.id);
    if (storedProduct !== null) {
        throw new Error(`a product with ${product.id} already exists`);
    }
    listedProducts.set(product.id, Product.fromPayload(product));
}

export function getProduct(id: string): Product | null {
    return listedProducts.get(id);
}

export function getProducts(): Product[] {
    return listedProducts.values();
}

export function buyProduct(productId: string): void {
  const product = getProduct(productId);
  if (product == null) {
      throw new Error("product not found");
  }
  if (product.price.toString() != context.attachedDeposit.toString()) {
      throw new Error("attached deposit should equal to the product's price");
  }
  ContractPromiseBatch.create(product.owner).transfer(context.attachedDeposit);
  product.incrementSoldAmount();
  listedProducts.set(product.id, product);
}

```

Now we need to compile our contract for the last time:

```bash
yarn asb
```

Then we can redeploy the contract to the same account id as before:

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=build/release/${WASM_FILE_NAME}
```

Let's test our contract by calling the `buyProduct` function. To do that, we will create a new sub-account that will act as the buyer and transfer some tokens to it:

```bash
near create-account buyeraccount.myaccount.testnet --masterAccount myaccount.testnet --initialBalance 6
```

Now we are ready to buy a product with the account. Here is how the code for buying a product looks like:

```bash
near call mycontract.myaccount.testnet buyProduct '{"productId": "0"}' --depositYocto=1000000000000000000000000 --accountId=buyeraccount.myaccount.testnet
```

New in this call is the `--depositYocto` parameter. This parameter specifies the amount of tokens that the buyer will attach to the transaction. In this case, we are attaching 1 NEAR tokens in Yocto-NEAR. We execute this call from the buyer account of course.

If we don't have any errors, we should see the following output:

```
Scheduling a call: mycontract.myaccount.testnet.buyProduct({"productId": "0"}) with attached 1 NEAR
Doing account.functionCall()
Transaction Id ${TRANSACTION_ID}
To see the transaction in the transaction explorer, please open this URL in your browser
https://explorer.testnet.near.org/transactions/${TRANSACTION_ID}
''
```

Let's see if the transaction went through correctly. Copy the link to the block explorer of the testnet and open it in your browser. You should see the transaction in the transaction explorer, check if the transaction went through correctly, and the token transfer amount is 1 NEAR.

Next, we will check if the product was bought correctly. We will use the `getProduct` function to retrieve the product and check if the `sold` field is equal to 1.

```bash
near view mycontract.myaccount.testnet getProduct '{"id": "0"}'
```

The output should now look like this:

```
View call: mycontract.myaccount.testnet.getProduct({"id": "0"})
{
  id: '0',
  name: 'BBQ',
  description: 'Grilled chicken and beef served with vegetables and chips.',
  location: 'Berlin, Germany',
  price: '1000000000000000000000000',
  image: 'https://i.imgur.com/yPreV19.png',
  owner: 'myaccount.testnet',
  sold: 1
}
```

That's it! We have successfully written a contract for a decentralized marketplace.

You can find the code for this project on [GitHub](https://github.com/dacadeorg/near-marketplace-dapp/tree/master/smartcontract).

Next, you can look at our learning modules that explain how to build the frontend for the marketplace.
