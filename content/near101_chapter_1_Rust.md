This learning module consists of a tutorial, where you will learn how to create a smart contract for a decentralized marketplace on the NEAR blockchain.

In this module we are going to use Rust language to write our NEAR smart contract. NEAR provides the same SDK for contracts development however the language itself brings additional complexity. Before we start, we need to prepare a development environment, install tools and setup a project and we are going to cover all of this in the following sections.

### Prerequisites

- Have some basic knowledge of Blockchain technology and smart contracts.
- Be familiar with Rust knowledge.
- Be comfortable using a terminal.

### Tech Stack

We will use the following tech stack:

- [near-cli](https://www.npmjs.com/package/near-cli) - A CLI tool for NEAR that offers an API to interact with smart contracts.
- [Rustup toolchain](https://www.npmjs.com/package/asbuild) - an installer for Rust and Cargo(package manager for Rust).

## 1. Setup

In this first section of the tutorial, we will set up our development environment and project.

### 1.1 Install CLI Tools

You can install the latest versions of CLI tools globally by running the following commands:

```bash
yarn global add near-cli
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

After installing Rustup on a UNIX-based system, run the following commange to configure your current shell:
```bash
source $HOME/.cargo/env
```

Also, you use a Windows-based machine, it is recommended using Windows Subsystem for Linux (WSL 2). More information on how to setup WSL can be found [here](https://docs.microsoft.com/en-us/windows/wsl/install). After that, you can follow the instruction above to proceed with installing `rust-toolkit`.

When Rustup is installed, we need to add `wasm` target to the toolchain which is a target for Rust that produces WebAssembly as output. To do that, run following command in your terminal:
```bash
rustup target add wasm32-unknown-unknown
```

We recommend using a code editor that supports code completion and syntax highlighting and code completion for Rust, like Visual Studio Code or Atom. We will use Visual Studio Code for this learning module.

### 1.2 Project Setup

Now we will set up our project. We will use `cargo` to setup a repository for a smart contract and navigate to the root directory:
```bash
cargo new near-marketplace-contract --lib
cd near-marketplace-contract
```

Next, open `Cargo.toml` in a text editor and replace its content with the following:
```toml
[package]
name = "near-marketplace-contract"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
near-sdk = "4.0.0"
serde = { version = "1.0", features = ["derive"] }

[profile.release]
codegen-units = 1
# Tell `rustc` to optimize for small code size.
opt-level = "z"
lto = true
debug = false
panic = "abort"
# Opt into extra safety checks on arithmetic operations https://stackoverflow.com/a/64136471/249801
overflow-checks = true
```

After that, run the next command to update dependencies in the `Cargo.lock` file:
```bash
cargo update
```

After running this command the structure of the project should look like this:
```bash
near-marketplace-contract
├── Cargo.lock
├── Cargo.toml # Hold a list of dependencies to be used for development. Similar to package.json
├── src
    └── lib.rs # Entry file for the contract
```
`lib.rs` works as an entry file for the contract and this is a conventional filename for Rust libraries which compile to WebAssembly.

## 2. Contract Storage

In this section we will learn how to store data in the smart contract.

### 2.1 Storage

Our smart contracts will need to store data on the blockchain. NEAR offers different storage options depending on the use case and the data type.

Also, with Rust SDK for NEAR it is possible to use both `std::collections` from the Rust's standard library and `near_sdk::collections` from the NEAR SDK.

Both variants work differently in terms of memory and gas usage and what and when to use depends on the use case. 
For example, `HashMap<K, V>` from `std::` keep all data in-memory and when a contract access it, it deserializes the whole map which can be very expensive. Even when you need to access a single element in this collection it deserializes the entire data structure. Also, it suitable for use cases when you need to iterated over a collection of data.

On the other hand, there is an equivalent for this data structure in the NEAR SDK - `UnorderedMap<K, V>`. This collection works completely opposite comparing to `HashMap<K, V>`. When it comes to accessing elements in this collection, a contract extracts only selected elements.

You can learn more about storage and collections in the [NEAR documentation](https://www.near-sdk.io/contract-structure/collections).

### 2.2 Read and Write State

There are two types of function calls, `view` and `change`. Calls that are `view` will only read data from the blockchain, while calls that are `change` will write data to the blockchain and modify its state. Calls that are `view` are free, while we need to pay gas for `change` calls.

For instance, calling `UnorderedMap#get(key: K)` and retrieving data behind the corresponding key won't cost gas. While calling `UnorderedMap#insert(key: K, value: V)` and adding a new key-value pair will cost gas.

## 3. Read and Write Contract

In this section of the tutorial, we are going to write a simple smart contract that will store and retrieve data from the blockchain.

We will write this contract in our `src/lib.rs` file.

Let's start by importing modules that are needed for the contract:

```rust
use near_sdk::collections::UnorderedMap;
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::{near_bindgen, PanicOnDefault};
```
With `UnorderedMap` it is pretty much clear what does it mean - we import a data structure that we are going to use as a storage.
Next, `BorshDeserialize` and `BorshSerialize` are traits that must be derived for data structures to be persisted and read from a blockchain. For instance, when there is a JSON structure like `{"id": "1", "name": "John Doe"}` it can be serialized into Borsh format and later it can be deserialized into `struct SomeStruct {id: String, name: String}`.

Also, we import `near_bindgen` attribute which is used both on structs and functions so the necessary code can be generated and these functions will be exposed so they can be called externally.

The last one is `PanicOnDefault` trait which generates implementation for `Default` trait that panics when the contract is called before it is actually initialized.

Next, we declare `Marketplace` struct where we are going to store products:

```rust
#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
pub struct Marketplace {
     products: UnorderedMap<String, String>
}
```

We declare a variable called `products` that is an `UnorderedMap` that will map product ids of type `String` to product names of type `String`.

With this line `#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]` we tell the compiler to generate necessary items from the traits listed there.

The next step is to implement the `Marketplace` struct:
```rust
#[near_bindgen]
impl Marketplace {
    // marketplace methods will be implemented here
}
```
And now we are going to implement marketplace methods;

### 3.1 Initializtion

Before we implement write and read functions, we need to initialize the contract. In the previous step we declare `products` structure but it has not been initialized yet. To do that, let's write `init` method where we actually create a new instance of `UnorderedMap`:
```rust
#[init]
pub fn init() -> Self {
    Self {
        products: UnorderedMap::new(b"product".to_vec()),
    }
}
```

Also, we added `#[init]` attribute which marks the methods a the one that holds custom initialization logic. 

By default, `Default::default()` will be used to initialize the contract. However, override this behavior `PanicOnDefault` macro should be used instead. 

With all that above, before we can actually invoke functions of the cotracts, the `init` method must be called first (and only once).

The string `products` in the `UnorderedMap`'s constructor is the unique prefix to use for every key.

### 3.2 Write Function

Lets create a method to add a new product to the `products` map:

```rust
pub fn set_product(&mut self, id: String, product_name: String) {
    self.products.insert(&id, &product_name);
}
```

Our `set_product` method needs two parameters: `id` and `product_name`. The `id` parameter is the key of the product, and the `product_name` parameter is the value of the product.

To create a new entry to our products mapping we just need to call the `insert` method on the `products` data structure and pass the key and value as parameters.

### 3.3 Read Function

To finish our first smart contract, we need to create a method to retrieve a product from the `products` map.

```rust
pub fn get_product(self, id: &String) -> Option<String> {
    self.products.get(id)
}
```

`get_product` method has just one parameter `id`, which is the key of the product we want to retrieve.

The return type of the method is `Option<String>`, since we can return either a product name or `null` (`None` will be translated to `null` where product is not found) if the product doesn't exist.

In the method body we call the `get` method on the `products` data structure.

Important note: for all read methods like `get_product`, `self` should be immutable. Otherwise, you might experience some issues when calling those methods.

The final code for this section looks like this:

```rust
use near_sdk::collections::UnorderedMap;
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::{near_bindgen, PanicOnDefault};

#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
pub struct Marketplace {
     products: UnorderedMap<String, String>
}

#[near_bindgen]
impl Marketplace {
    
    #[init]
    pub fn init() -> Self {
        Self {
            products: UnorderedMap::new(b"product".to_vec()),
        }
    }

    pub fn set_product(&mut self, id: String, product_name: String) {
        self.products.insert(&id, &product_name);
    }

    pub fn get_product(self, id: &String) -> Option<String> {
        self.products.get(id)
    }

}
```

**Warning**: The key for the persistent collection should be as short as possible to reduce storage space because this key will be repeated for every record in the collection. Here, we only used the longer `products` key to add more readability for first-time NEAR developers.

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
RUSTFLAGS='-C link-arg=-s' cargo build --target wasm32-unknown-unknown --release
```

The compiled wasm code will be stored in a file called `${CONTRACT_NAME}.wasm` in the following directory:

```bash
${PROJECT_ROOT}/target/wasm32-unknown-unknown/release/${CONTRACT_NAME}.wasm
```

`${CONTRACT_NAME}` refers to the name of the package specified in the `Cargo.toml` file.

### 5.2 Deploy the Contract

To deploy our smart contract to the NEAR testnet, we need to run the following command:

```bash
near deploy --accountId=${ACCOUNT_ID} --wasmFile=${PATH_TO_WASM}
```

- `${ACCOUNT_ID}` - the id of the account that will deploy the smart contract to.
- `${PATH_TO_WASM}` - the path to the `.wasm` file that contains the compiled smart contract.

In our case the deploy command could look like this:

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=target/wasm32-unknown-unknown/release/near_marketplace_contract.wasm
```

Now our contract is deployed to the NEAR testnet and we can interact with it.

## 6. Contract Calls

In this section of the learning module, we will call the functions on the deployed smart contract.

As stated at the beginning of the learning module, there are two types of function calls that we can make: `view` and `change`.

### 6.1 Initialization

Before we can invoke smart contracts' functions, we need to initialize it. To do that, we should invoke `init` function:
```bash
near call mycontract.myaccount.testnet init --accountId=myaccount.testnet
```

With this call we initialize the contract and the underlying data structure that we are going to use to store data.

### 6.2 Calling a Change Function

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
near call mycontract.myaccount.testnet set_product '{"id": "0", "product_name": "tea"}' --accountId=myaccount.testnet
```

If you use PowerShell or CMD on Windows, you might need to escape double quotes in the payload, which could look like this:

```bash
near call mycontract.myaccount.testnet set_product "{\"id\": \"0\", \"product_name\": \"tea\"}" --accountId=myaccount.testnet
```

Keep this in mind for the following sections when we have double quotes in the payload.

If your contract call was successful, you will see something similar to the following output:

```
Scheduling a call: mycontract.myaccount.testnet.set_product({"id": "0", "product_name": "tea"})
Doing account.functionCall()
Transaction Id ${TRANSACTION_ID}
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.testnet.near.org/transactions/${TRANSACTION_ID}
''
```

### 6.3 Calling a View Function

Now that we have added a product to the contract, we can call the `view` function to retrieve the product.

The `view` contract call in the `near-cli` looks like this:

```bash
near view ${CONTRACT_ACCOUNT_ID} ${METHOD_NAME} ${PAYLOAD}
```

Since we don't need to pay any gas we can omit the account id at the end.

If we want to retrieve the product we just added, the call could look like this:

```bash
near view mycontract.myaccount.testnet get_product '{"id": "0"}'
```

If your contract call was successful, you will see something similar to the following output:

```
View call: mycontract.myaccount.testnet.get_product({"id": "0"})
'tea'
```

Now you are able to compile and deploy your contract to the NEAR testnet and interact with it.

## 7. Contract with Product Model

In this section, we are going to write a second iteration of our contract that can store more than just a string.

### 7.1 Create Product Model

We are going to use a struct called a `Product` to represent our products, because we want to be able to store more than just the name of the product.

Let's add this strucutre to the `lib.rs` file.

```rust
#[near_bindgen]
#[derive(BorshSerialize, BorshDeserialize, Serialize, PanicOnDefault)]
pub struct Product {
    id: String,
    name: String,
    description: String,
    image: String,
    location: String,
    price: String,
    owner: AccountId,
    sold: u32
}
```

The `Product` structure consists of the next properties: `id`, `name`, `description`, `image`, `location` and `owner` of the product, which are all strings. We also have a `price` which is a 128 bit unsigned integer that we store as a string for a simpler seriliazation to avoid scientific notation for big numbers in json, and a `sold` property which is a 32 bit unsigned integer.

The `u128` `price` field allows us to store the NEAR price in [yocto](https://www.nanotech-now.com/metric-prefix-table.htm). 1 yocto-NEAR = 10<sup>-24</sup> NEAR, which is the smallest unit of NEAR.

Also, we add an additional struct called `Payload` which will be used as an intermediate object that hold payload data that will be mapped to the `Product` struct.
```rust
#[near_bindgen]
#[derive(Serialize, Deserialize, PanicOnDefault)]
pub struct Payload {
    id: String,
    name: String,
    description: String,
    image: String,
    location: String,
    price: String
}
```

Next, we are going to implement `Product` strucutre and add a factory function `from_payload` that will map `Payload` to `Product`.
```rust
#[near_bindgen]
impl Product {

    pub fn from_payload(payload: Payload) -> Self {
        Self {
            id: payload.id,
            description: payload.description,
            name: payload.name,
            location: payload.location,
            price: payload.price,
            sold: 0,
            image: payload.image,
            owner: env::signer_account_id()
        }
    }
}
```
It is a simple function that maps data between structs excepts for three properties: `price`, `sold` and `owner`.

We initilize `sold` to 0 as it is just a counter of sold products.

And the last one is `owner` which is set to the current signer account id automatically from `env`.

Also, we add a method called `increment_sold_amount` which we are going to use later to increment the `sold` value after a product has been sold.
```rust
pub fn increment_sold_amount(&mut self) {
    self.sold = self.sold + 1;
}
```

We also create a new map called `listed_products` which is a persistent unordered map and will replace the `products`. We could also migrate the data from the `products` map to the `listed_products` map, but this is an advanced topic that is out of scope for this learning module.

As explained earlier, we will use a more readable version of a key for `listed_products` to provide more readability to our code. To reduce storage space, you should pick a shorter key.

### 7.2 Update `lib.rs`

We need to update methods in the `src/lib.rs` file to use new structs:

```rust
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::UnorderedMap;
use near_sdk::{env, near_bindgen, AccountId, PanicOnDefault};
use near_sdk::serde::{Serialize, Deserialize};

#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
pub struct Marketplace {
    listed_products: UnorderedMap<String, Product>
}

#[near_bindgen]
impl Marketplace {
    
    #[init]
    pub fn init() -> Self {
        Self {
            listed_products: UnorderedMap::new(b"listed_products".to_vec()),
        }
    }

    pub fn set_product(&mut self, payload: Payload) {
        let product = Product::from_payload(payload);
        self.listed_products.insert(&product.id, &product);
    }

    pub fn get_product(self, id: &String) -> Option<Product> {
        self.listed_products.get(id)
    }

    pub fn get_products(self) -> Vec<Product> {
        return self.listed_products.values_as_vector().to_vec();
    }
}
```

Here we modify `set_product` method so it can use the new `Payload` struct as a paramter. We first check if the product id already exists in the map. If it does, we throw an error. Otherwise, we call the `from_payload` method to create a new `Product` from the payload and store it in the `listed_products` map.

We also modify the `get_product` method so it can return the `Product` struct representation of a stored product.

Finally, we create the `get_products` method to return all products in the map.

### 7.3 Contract Redeployment

NEAR allows us to update our contract code on the blockchain. We can do this by redeploying the contract.

We need to compile our new contract first:

```bash
RUSTFLAGS='-C link-arg=-s' cargo build --target wasm32-unknown-unknown --release
```

Then we can redeploy the contract to the same account id as before:

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=target/wasm32-unknown-unknown/release/${WASM_FILE_NAME}
```

Let's add a new product to the contract by calling the `set_product` function. Since we are using the `Product` struct, we need to pass in a payload that is a `Product` object, which could look like this:

```bash
near call mycontract.myaccount.testnet set_product '{"payload": {"id": "1", "name": "BBQ", "description": "Grilled chicken and beef served with vegetables and chips.", "location": "Berlin, Germany", "price": "1000000000000000000000000", "image": "https://i.imgur.com/yPreV19.png"}}' --accountId=myaccount.testnet
```

After a successful `set_product` call, we can call the `get_product` function to retrieve the product we just added:

```bash
near view mycontract.myaccount.testnet get_product '{"id": "1"}'
```

You should an output similar to the following:

```
View call: mycontract.myaccount.testnet.get_product({"id": "1"})
{
  id: '1',
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

For the final section of this tutorial, we will create the `buy_product` function to allow a user to buy a product.

The `near_sdk` provides a `Promise` struct which allows us to assemble transfer calls to other accounts.

The code for a token transfer looks like this:

```rust
Promise::new(${RECEIVING_ACCOUNT}).transfer(${DEPOSIT});
```

We previously mentioned that the `env` object contains information about a transaction. For our `buy_product` function we will use `env::attached_deposit` to get the amount of tokens that the caller of the function has attached to the transaction.

Let's write our `buy_product` function in the `lib.rs` file.

```rust
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::{UnorderedMap};
use near_sdk::{env, near_bindgen, AccountId, PanicOnDefault, Promise};
use near_sdk::serde::{Serialize, Deserialize};
```

Now we can write our `buy_product` function at the bottom of the file:

```rust
#[payable]
pub fn buy_product(&mut self, product_id: &String) {
    match self.listed_products.get(product_id) {
        Some(ref mut product) => {
            assert_eq! (env::attached_deposit(), product.price, "attached deposit should be equal to the product's");
            let owner = &product.owner.as_str();
            Promise::new(owner.parse().unwrap()).transfer(product.price);
            product.increment_sold_amount();
            self.listed_products.insert(&product.id, &product);
        },
        _ => {
            env::panic_str("product not found");
        }
    }
}
```

First, we retrieve the product with the specified id.

Then we check if the product exists. If it doesn't, we throw an error ("product not found"). Otherwise, we check if the attached deposit is equal to the product's price. If it isn't, we throw an error ("attached deposit should equal to the product's price").

We then create a new `Promise` object and call the `transfer` method on it. This method takes the amount of tokens that the caller of the function has attached to the transaction and transfers them to the owner of the product. We get the account of the owner of the product by accessing the `owner` property of the product we retrieved.

Finally, we increment the `sold` field of the product by calling the `increment_sold_amount` function and update the product in the `listed_products` map.

This is it! The final contract should look like this:

```rust
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::{UnorderedMap};
use near_sdk::{env, near_bindgen, AccountId, PanicOnDefault, Promise};
use near_sdk::serde::{Serialize, Deserialize};

#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
pub struct Marketplace {
    listed_products: UnorderedMap<String, Product>
}

#[near_bindgen]
impl Marketplace {
    
    #[init]
    pub fn init() -> Self {
        Self {
            listed_products: UnorderedMap::new(b"listed_products".to_vec()),
        }
    }

    pub fn set_product(&mut self, payload: Payload) {
        let product = Product::from_payload(payload);
        self.listed_products.insert(&product.id, &product);
    }

    pub fn get_product(self, id: &String) -> Option<Product> {
        self.listed_products.get(id)
    }

    pub fn get_products(self) -> Vec<Product> {
        self.listed_products.values_as_vector().to_vec()
    }

    #[payable]
    pub fn buy_product(&mut self, product_id: &String) {
        match self.listed_products.get(product_id) {
            Some(ref mut product) => {
                let price = product.price.parse().unwrap();
                assert_eq! (env::attached_deposit(), price, "attached deposit should be equal to the price of the product");
                let owner = &product.owner.as_str();
                Promise::new(owner.parse().unwrap()).transfer(price);
                product.increment_sold_amount();
                self.listed_products.insert(&product.id, &product);
            },
            _ => {
                env::panic_str("product not found");
            }
        }
    }
}

#[near_bindgen]
#[derive(Serialize, Deserialize, PanicOnDefault)]
pub struct Payload {
    id: String,
    name: String,
    description: String,
    image: String,
    location: String,
    price: String
}

#[near_bindgen]
#[derive(BorshSerialize, BorshDeserialize, Serialize, PanicOnDefault)]
pub struct Product {
    id: String,
    name: String,
    description: String,
    image: String,
    location: String,
    price: String,
    owner: AccountId,
    sold: u32
}

#[near_bindgen]
impl Product {

    pub fn from_payload(payload: Payload) -> Self {
        Self {
            id: payload.id,
            description: payload.description,
            name: payload.name,
            location: payload.location,
            price: payload.price,
            sold: 0,
            image: payload.image,
            owner: env::signer_account_id()
        }
    }

    pub fn increment_sold_amount(&mut self) {
        self.sold = self.sold + 1;
    }
}
```

Now we need to compile our contract for the last time:

```bash
RUSTFLAGS='-C link-arg=-s' cargo build --target wasm32-unknown-unknown --release
```

Then we can redeploy the contract to the same account id as before:

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=target/wasm32-unknown-unknown/release/${WASM_FILE_NAME}
```

Let's test our contract by calling the `buy_product` function. To do that, we will create a new sub-account that will act as the buyer and transfer some tokens to it:

```bash
near create-account buyeraccount.myaccount.testnet --masterAccount myaccount.testnet --initialBalance 6
```

Now we are ready to buy a product with the account. Here is how the code for buying a product looks like:

```bash
near call mycontract.myaccount.testnet buy_product '{"product_id": "1"}' --depositYocto=1000000000000000000000000 --accountId=buyeraccount.myaccount.testnet
```

New in this call is the `--depositYocto` parameter. This parameter specifies the amount of tokens that the buyer will attach to the transaction. In this case, we are attaching 1 NEAR tokens in Yocto-NEAR. We execute this call from the buyer account of course.

If we don't have any errors, we should see the following output:

```
Scheduling a call: mycontract.myaccount.testnet.buy_product({"product_id": "1"}) with attached 1 NEAR
Doing account.functionCall()
Transaction Id ${TRANSACTION_ID}
To see the transaction in the transaction explorer, please open this URL in your browser
https://explorer.testnet.near.org/transactions/${TRANSACTION_ID}
''
```

Let's see if the transaction went through correctly. Copy the link to the block explorer of the testnet and open it in your browser. You should see the transaction in the transaction explorer, check if the transaction went through correctly, and the token transfer amount is 1 NEAR.

Next, we will check if the product was bought correctly. We will use the `get_product` function to retrieve the product and check if the `sold` field is equal to 1.

```bash
near view mycontract.myaccount.testnet get_product '{"id": "1"}'
```

The output should now look like this:

```
View call: mycontract.myaccount.testnet.get_product({"id": "1"})
{
  id: '1',
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

You can find the code for this project on [GitHub](https://github.com/dacadeorg/near-marketplace-dapp/tree/master/smartcontract/rust).

Next, you can look at our learning modules that explain how to build the frontend for the marketplace.
