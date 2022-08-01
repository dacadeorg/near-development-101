In this learning module, we will follow a tutorial to build the frontend for a marketplace contract.
This tutorial assumes that you have already completed the [Connect a React Dapp to NEAR](/near101_chapter_2.md) learning module and continue in the same project.

### Prerequisites

- [Node JS](https://nodejs.org/en/download/) - Please make sure you have Node.js v16 or higher installed.
- You should have a basic understanding of [React](https://reactjs.org/): know how to use JSX, props, state, lifecycle methods, and hooks.
- You should have followed the [Connect a React Dapp to NEAR](/near101_chapter_2.md) learning module and have `react`, `react-scripts` v.2.1.4, `near-api-js` and `uuid` installed.

### Tech Stack

We will use the following tech stack:

- [React](https://reactjs.org/) - A JavaScript library for building user interfaces.
- [Bootstrap](https://getbootstrap.com/) - A CSS framework.
- [near-api-js](https://docs.near.org/docs/api/javascript-library) - A JavaScript/Typescript library for interacting with NEAR's blockchain.

## 1. Project Setup

Since we already have the important dependencies installed, we now only need to add our dependencies for the frontend and styling:

```bash
npm install react-bootstrap bootstrap bootstrap-icons react-toastify
```

We will use `react-bootstrap` to handle the `Bootstrap` styling of our react components. We will also use `react-toastify` to display notifications to the user, so we don't have to handle that ourselves.

### 1.1 index.js

Let's open the `src/index.js` file and start adding our bootstrap component and styles:

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import { initializeContract } from "./utils/near";

import "bootstrap";
import "bootstrap-icons/font/bootstrap-icons.css";
import "bootstrap/dist/css/bootstrap.min.css";

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

We initialize the contract by calling the `initializeContract` function from the `utils/near.js` file, that we created in the [Connect a React Dapp to NEAR](/near101_chapter_2.md) learning module.

### 1.2 utils

The `src/utils/` directory should look like this if you followed the [Connect a React Dapp to NEAR](/near101_chapter_2.md) learning module:

```
├── utils
│   ├── config.js
│   ├── marketplace.js
│   └── near.js
```

### 1.3 App.js

We will set up the `App.js` file to render our UI. Let's open the `src/App.js` file and start with the imports:

```js
import React, { useEffect, useCallback, useState } from "react";
import { Container, Nav } from "react-bootstrap";
import { login, logout as destroy, accountBalance } from "./utils/near";
import Wallet from "./components/Wallet";
// import { Notification } from "./components/utils/Notifications";
// import Products from "./components/marketplace/Products";
import Cover from "./components/utils/Cover";
import coverImg from "./assets/img/sandwich.jpg";
import "./App.css";
//..
```

We import the `login`, `logout` and `accountBalance` functions from the `utils/near.js` file. We also import the `Wallet` and `Cover` components and the `coverImg` image file. All of which have not been created yet.

For now, the `Notification` and `Products` components will remain uncommented as we will implement them later.

Let's create our `App` component now:

```js
//..
const App = function AppWrapper() {
  const account = window.walletConnection.account();
  const [balance, setBalance] = useState("0");
  const getBalance = useCallback(async () => {
    if (account.accountId) {
      setBalance(await accountBalance());
    }
  });

  useEffect(() => {
    getBalance();
  }, [getBalance]);
//..
```

We get the `account` when connected to the wallet. If we are connected, we get the `accountId` and set the `balance` state. We call the `accountBalance` function from the `utils/near.js` file to get the balance.

Let's return the JSX for our `App` component:

```js
//..
  return (
    <>
      {/* <Notification /> */}
      {account.accountId ? (
        <Container fluid="md">
          <Nav className="justify-content-end pt-3 pb-5">
            <Nav.Item>
              <Wallet
                address={account.accountId}
                amount={balance}
                symbol="NEAR"
                destroy={destroy}
              />
            </Nav.Item>
          </Nav>
          <main>{/* <Products /> */}</main>
        </Container>
      ) : (
        <Cover name="Street Food" login={login} coverImg={coverImg} />
      )}
    </>
  );
};

export default App;
```

If the user is connected to the wallet, we display our dapp. If they aren't, we render the `Cover` component.

We pass a `name` for our dapp and a `coverImg` as props to the `Cover` component. We also pass a `login` function to log in to the wallet.

The dapp consists of the `Wallet` component, which displays the user's account address and balance. We also show the `Products` component, which we will implement later.
The `Wallet` needs the account address, the user's balance, a symbol for the currency we display, and a `destroy` function to log out of the wallet as props.

Now let's create the components we already used.

## 2. Components

In this section of the tutorial, we will create the custom components we will use in our dapp.

The components directory will look like this:

```
├── components
│   ├── marketplace
│   ├── utils
│   └── Wallet.js
```

We will start with the `Cover` component.

### 2.1 Cover.js

Create a `components` folder in the `src` directory and create a `src/components/utils/Cover.js` file with the following code:

```js
import React from "react";
import PropTypes from "prop-types";
import { Button } from "react-bootstrap";

const Cover = ({ name, login, coverImg }) => {
  if ((name, login, coverImg)) {
    return (
      <div
        className="d-flex justify-content-center flex-column text-center "
        style={{ background: "#000", minHeight: "100vh" }}
      >
        <div className="mt-auto text-light mb-5">
          <div
            className=" ratio ratio-1x1 mx-auto mb-2"
            style={{ maxWidth: "320px" }}
          >
            <img src={coverImg} alt="" />
          </div>
          <h1>{name}</h1>
          <p>Please connect your wallet to continue.</p>
          <Button
            onClick={login}
            variant="outline-light"
            className="rounded-pill px-3 mt-3"
          >
            Connect Wallet
          </Button>
        </div>
        <p className="mt-auto text-secondary">Powered by NEAR</p>
      </div>
    );
  }
  return null;
};

Cover.propTypes = {
  name: PropTypes.string,
};

Cover.defaultProps = {
  name: "",
};

export default Cover;
```

This component is pretty simple. If it receives the `name`, `login` and `coverImg` as props, we render the `coverImg` and the `name` of the dapp. We also display a `Connect Wallet` button that calls the `login` function when clicked.

#### 2.1.1 Cover Image

Since we are using a cover image, we need to import the `coverImg` image file.
For this tutorial, we chose a `sandwich.jpg` image that you can find [here](https://raw.githubusercontent.com/dacadeorg/near-marketplace-dapp/master/src/assets/img/sandwich.jpg). We create two new nested folders in the assets directory and store the image `assets/img/sandwich.jpg`.

Now let's continue with the `Wallet` component.

### 2.1 Wallet.js

The wallet component will display the user's account address, balance, and logout button. Create a `src/components/Wallet.js` file with the following code:

```js
import React from "react";
import { Dropdown, Stack, Spinner } from "react-bootstrap";

const Wallet = ({ address, amount, symbol, destroy }) => {
  if (address) {
    return (
      <>
        <Dropdown>
          <Dropdown.Toggle
            variant="light"
            align="end"
            id="dropdown-basic"
            className="d-flex align-items-center border rounded-pill py-1"
          >
            {amount ? (
              <>
                {amount} <span className="ms-1"> {symbol}</span>
              </>
            ) : (
              <Spinner animation="border" size="sm" className="opacity-25" />
            )}
          </Dropdown.Toggle>

          <Dropdown.Menu className="shadow-lg border-0">
            <Dropdown.Item
              href={`https://explorer.testnet.near.org/accounts/${address}`}
              target="_blank"
            >
              <Stack direction="horizontal" gap={2}>
                <i className="bi bi-person-circle fs-4" />
                <span className="font-monospace">{address}</span>
              </Stack>
            </Dropdown.Item>

            <Dropdown.Divider />
            <Dropdown.Item
              as="button"
              className="d-flex align-items-center"
              onClick={() => {
                destroy();
              }}
            >
              <i className="bi bi-box-arrow-right me-2 fs-4" />
              Disconnect
            </Dropdown.Item>
          </Dropdown.Menu>
        </Dropdown>
      </>
    );
  }

  return null;
};

export default Wallet;
```

We receive the `address`, the user balance (`amount`), and the `symbol` of the currency we display as props. We also receive a `destroy` function to log out of the wallet. As described earlier, these are passed from the `App` component.

Now we should be ready to have run our dapp and see if we can log in, log out, and see our balance and address.

Run the dapp:

```bash
npm start
```

The app should now look behave like this:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/test_wallet_login.gif)

### 2.2 utils

Let's next work on some utility components. The `utils` directory will look like this:

```
├── components
│   ├── utils
│   │   ├── Cover.js
│   │   ├── Loader.js
│   │   └── Notifications.js
```

We already have the `Cover` component. Let's create the `Loader` component next.

#### 2.2.1 utils/Loader.js

The `Loader` component will display a loading animation. Create a new `components/utils/Loader.js` file with the following code:

```js
import React from "react";
import { Spinner } from "react-bootstrap";

const Loader = () => (
  <div className="d-flex justify-content-center">
    <Spinner animation="border" role="status" className="opacity-25">
      <span className="visually-hidden">Loading...</span>
    </Spinner>
  </div>
);
export default Loader;
```

This component is pretty simple. It just displays the bootstrap `Spinner` component.

#### 2.2.2 utils/Notifications.js

The `Notification` component will display notifications to the user.
Create a new `components/utils/Notifications.js` file with the following code:

```js
import React from "react";
import { ToastContainer } from "react-toastify";
import PropTypes from "prop-types";
import "react-toastify/dist/ReactToastify.css";

const Notification = () => (
  <ToastContainer
    position="bottom-center"
    autoClose={5000}
    hideProgressBar
    newestOnTop
    closeOnClick
    rtl={false}
    pauseOnFocusLoss
    draggable={false}
    pauseOnHover
  />
);

const NotificationSuccess = ({ text }) => (
  <div>
    <i className="bi bi-check-circle-fill text-success mx-2" />
    <span className="text-secondary mx-1">{text}</span>
  </div>
);

const NotificationError = ({ text }) => (
  <div>
    <i className="bi bi-x-circle-fill text-danger mx-2" />
    <span className="text-secondary mx-1">{text}</span>
  </div>
);

const Props = {
  text: PropTypes.string,
};

const DefaultProps = {
  text: "",
};

NotificationSuccess.propTypes = Props;
NotificationSuccess.defaultProps = DefaultProps;

NotificationError.propTypes = Props;
NotificationError.defaultProps = DefaultProps;

export { Notification, NotificationSuccess, NotificationError };
```

This component uses the `react-toastify` library to display notifications.
We distinguish between success and error notifications and otherwise display the `text` as a string.
The `Notification` component is implemented in the `App` component.

### 2.3 marketplace

Now we create our final component, where we will create the UI for the marketplace. The `components/marketplace` directory will look like this:

```
├── components
│   ├── marketplace
│   │   ├── AddProduct.js
│   │   ├── Product.js
│   │   └── Products.js
│   ├── utils
│   └── Wallet.js
```

Let's start with the `Products` component.

#### 2.3.1 Products.js

The `Products` component will display a list of products.
It will be the main component of the marketplace that will contain the `AddProducts` and `Product` components.
Create a new `marketplace` folder in the `components` directory and create a `src/components/marketplace/Products.js` file.

Let's start with the imports:

```js
import React, { useEffect, useState, useCallback } from "react";
import { toast } from "react-toastify";
import AddProduct from "./AddProduct";
import Product from "./Product";
import Loader from "../utils/Loader";
import { Row } from "react-bootstrap";
import { NotificationSuccess, NotificationError } from "../utils/Notifications";
import {
  getProducts as getProductList,
  buyProduct,
  createProduct,
} from "../../utils/marketplace";
//...
```

We import the `AddProduct` and `Product` components, that we will create later.
We also import the `Loader` and `NotificationSuccess` and `NotificationError` components from the `utils` directory.
Finally, we import the `getProductList`, `buyProduct` and `createProduct` utility functions from the `utils/marketplace` directory.

Let's create our main `Products` component and a `getProducts` function, which we use to fetch the list of products:

```js
//...
const Products = () => {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);

  const getProducts = useCallback(async () => {
    try {
      setLoading(true);
      setProducts(await getProductList());
    } catch (error) {
      console.log({ error });
    } finally {
      setLoading(false);
    }
  });
//...
```

We set the `loading` state to `true` when we fetch the products and set the `products` state to the list of products. To fetch the products, we use the `getProductList` utility that we imported earlier.
Once we have the products, we set the `loading` state to `false`.

Next we create the `AddProduct` function:

```js
//...
const addProduct = async (data) => {
  try {
    setLoading(true);
    createProduct(data).then((resp) => {
      getProducts();
    });
    toast(<NotificationSuccess text="Product added successfully." />);
  } catch (error) {
    console.log({ error });
    toast(<NotificationError text="Failed to create a product." />);
  } finally {
    setLoading(false);
  }
};
//...
```

We receive the product data as a parameter and use the `createProduct` utility function to create the product. We then fetch the products again and display a success message, or an error message if the product could not be created.

Finally, we create the `buyProduct` function:

```js
//...
const buy = async (id, price) => {
  try {
    await buyProduct({
      id,
      price,
    }).then((resp) => getProducts());
    toast(<NotificationSuccess text="Product bought successfully" />);
  } catch (error) {
    toast(<NotificationError text="Failed to purchase product." />);
  } finally {
    setLoading(false);
  }
};

useEffect(() => {
  getProducts();
}, []);
//...
```

We need the `id` and `price` of the product, and the we can call the `buyProduct` utility function to buy the product. We then fetch the products again and display a success message, or an error message if the product could not be bought.

Now can return the `Products` component and write the JSX code:

```js
//...
  return (
    <>
      {!loading ? (
        <>
          <div className="d-flex justify-content-between align-items-center mb-4">
            <h1 className="fs-4 fw-bold mb-0">Street Food</h1>
            <AddProduct save={addProduct} />
          </div>
          <Row xs={1} sm={2} lg={3} className="g-3  mb-5 g-xl-4 g-xxl-5">
            {products.map((_product) => (
              <Product
                product={{
                  ..._product,
                }}
                buy={buy}
              />
            ))}
          </Row>
        </>
      ) : (
        <Loader />
      )}
    </>
  );
};

export default Products;
```

In the not yet created `AddProduct` component, we pass the `addProduct` function that we created earlier as a prop.

Then we map the list of `products` to the not yet created `Product` component, which is a component will display the individual product as a card. We pass the `_product`'s data as props to the `Product` component to render the product. We also pass the `buy` function as a prop.

Finally, we export the `Products` component.

Let's now create the `Product` component that we just discussed.

#### 2.3.2 Product.js

This file will contain the `Product` component, which will display the individual product as a card. In the `marketplace` directory, create a `src/components/marketplace/Product.js` file with the following code:

```js
import React from "react";
import PropTypes from "prop-types";
import { utils } from "near-api-js";
import { Card, Button, Col, Badge, Stack } from "react-bootstrap";

const Product = ({ product, buy }) => {
  const { id, price, name, description, sold, location, image, owner } =
    product;

  const triggerBuy = () => {
    buy(id, price);
  };

  return (
    <Col key={id}>
      <Card className=" h-100">
        <Card.Header>
          <Stack direction="horizontal" gap={2}>
            <span className="font-monospace text-secondary">{owner}</span>
            <Badge bg="secondary" className="ms-auto">
              {sold} Sold
            </Badge>
          </Stack>
        </Card.Header>
        <div className=" ratio ratio-4x3">
          <img src={image} alt={name} style={{ objectFit: "cover" }} />
        </div>
        <Card.Body className="d-flex  flex-column text-center">
          <Card.Title>{name}</Card.Title>
          <Card.Text className="flex-grow-1 ">{description}</Card.Text>
          <Card.Text className="text-secondary">
            <span>{location}</span>
          </Card.Text>
          <Button
            variant="outline-dark"
            onClick={triggerBuy}
            className="w-100 py-3"
          >
            Buy for {utils.format.formatNearAmount(price)} NEAR
          </Button>
        </Card.Body>
      </Card>
    </Col>
  );
};

Product.propTypes = {
  product: PropTypes.instanceOf(Object).isRequired,
  buy: PropTypes.func.isRequired,
};

export default Product;
```

This component is pretty simple. We get the product data: the `id`, `price`, `name`, `description`, how many times it has been `sold`, `location`, `image` and `owner` from the `product` prop.

In the `triggerBuy` function, we call the `buy` function that we passed as a prop with the `id` and `price` of the product as parameters.

Then we display the product data as a bootstrap `Card` component and call the `triggerBuy` function when the `Buy` button is clicked.

In the next section, we will create the `AddProduct` component.

#### 2.3.3 AddProduct.js

This file will contain the `AddProduct` component, which will display a form to add a new product. In the `marketplace` directory, create a `src/components/marketplace/AddProduct.js` file. We will start with the import statements and the `AddProduct` component:

```js
import React, { useState } from "react";
import PropTypes from "prop-types";
import { Button, Modal, Form, FloatingLabel } from "react-bootstrap";

const AddProduct = ({ save }) => {
  const [name, setName] = useState("");
  const [image, setImage] = useState("");
  const [description, setDescription] = useState("");
  const [location, setLocation] = useState("");
  const [price, setPrice] = useState(0);
  const isFormFilled = () => name && image && description && location && price;

  const [show, setShow] = useState(false);

  const handleClose = () => setShow(false);
  const handleShow = () => setShow(true);
//...
```

We create state variables for the product data and a boolean state variable that we use to store the state of the modal with the form for the product data.

In the next step, we will create the button and form that will be displayed in the popup modal:

```js
//...
 return (
    <>
      <Button
        onClick={handleShow}
        variant="dark"
        className="rounded-pill px-0"
        style={{ width: "38px" }}
      >
        <i class="bi bi-plus"></i>
      </Button>
      <Modal show={show} onHide={handleClose} centered>
        <Modal.Header closeButton>
          <Modal.Title>New Product</Modal.Title>
        </Modal.Header>
        <Form>
          <Modal.Body>
            <FloatingLabel
              controlId="inputName"
              label="Product name"
              className="mb-3"
            >
              <Form.Control
                type="text"
                onChange={(e) => {
                  setName(e.target.value);
                }}
                placeholder="Enter name of product"
              />
            </FloatingLabel>
            <FloatingLabel
              controlId="inputUrl"
              label="Image URL"
              className="mb-3"
            >
              <Form.Control
                type="text"
                placeholder="Image URL"
                onChange={(e) => {
                  setImage(e.target.value);
                }}
              />
            </FloatingLabel>
            <FloatingLabel
              controlId="inputDescription"
              label="Description"
              className="mb-3"
            >
              <Form.Control
                as="textarea"
                placeholder="description"
                style={{ height: "80px" }}
                onChange={(e) => {
                  setDescription(e.target.value);
                }}
              />
            </FloatingLabel>
            <FloatingLabel
              controlId="inputLocation"
              label="Location"
              className="mb-3"
            >
              <Form.Control
                type="text"
                placeholder="Location"
                onChange={(e) => {
                  setLocation(e.target.value);
                }}
              />
            </FloatingLabel>
            <FloatingLabel
              controlId="inputPrice"
              label="Price"
              className="mb-3"
            >
              <Form.Control
                type="text"
                placeholder="Price"
                onChange={(e) => {
                  setPrice(e.target.value);
                }}
              />
            </FloatingLabel>
          </Modal.Body>
        </Form>
        <Modal.Footer>
          <Button variant="outline-secondary" onClick={handleClose}>
            Close
          </Button>
          <Button
            variant="dark"
            disabled={!isFormFilled()}
            onClick={() => {
              save({
                name,
                image,
                description,
                location,
                price,
              });
              handleClose();
            }}
          >
            Save product
          </Button>
        </Modal.Footer>
      </Modal>
    </>
  );
};

AddProduct.propTypes = {
  save: PropTypes.func.isRequired,
};

export default AddProduct;
```

This file has a lot of code, but it's pretty straightforward. We create a modal that will be displayed when the user clicks on the `New Product` button.
Inside the modal, we display a form with the product data fields.

If the user clicks on the `Save product` button, we call the `save` function that we passed as a prop with the product data as parameters.

We are done with the components, now we need to do a bit of cleanup in our `App.js` file and we are ready to go!

## 3. Finishing the dapp

In this last section, we will put the final touches on the dapp.

### 3.1. Update `App.js`

Let's finish the dapp by adding our `Products` and `Notification` components to our `App.js` file.

Let's start by uncommenting the import of the `Products` and `Notification` components, so it looks like this:

```js
//...
import Wallet from "./components/Wallet";
import { Notification } from "./components/utils/Notifications";
import Products from "./components/marketplace/Products";
import Cover from "./components/utils/Cover";
//...
```

Now we need to uncomment the `<Notification />` component in the JSX:

```js
//...
  return (
    <>
      <Notification />
      {account.accountId ? (
<!-- ... -->
```

And we also need to uncomment the `<Products />` component:

```js
//...
<main>
  <Products />
</main>
//...
```

Do a final test. See if your products are displayed and if you can create a new product and buy it.

The dapp should now behave like this:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/final_dapp.gif)

### 3.2. Cleanup

We can do a little bit of housekeeping in our project. In our `src` directory, we can get rid of the `logo.svg` and `setupTests.js` files.

In the `public` directory, we can add our `favicon.ico`, `logo192.png`, `logo512.png`, and `manifest.json` files that fit our project.

We should also change the `title` and `description` of our dapp in the `index.html` file in the `public` directory.

### 3.3. Deploy to GitHub Pages

In the last section, we will briely look at how to deploy our dapp to GitHub Pages.

1. First add your project to GitHub. If you don't know how to do this, check out this [GitHub guide](https://docs.github.com/en/get-started/importing-your-projects-to-github/importing-source-code-to-github/adding-an-existing-project-to-github-using-the-command-line).

2. Install the [gh-pages](https://www.npmjs.com/package/gh-pages) package. This will allow us to deploy our dapp to GitHub Pages.

```bash
npm install gh-pages
```

3. In the package.json file, add the following lines:

- At the top of the file, add the following line:
  ```
  "homepage": "https://${GithubUsername}.github.io/${RepositoryName}",
  ```
  Replace `${GithubUsername}` and `${RepositoryName}` with your GitHub username and the repository name.
- add the following lines at the bottom of the `scripts` section:
  ```
    "predeploy": "npm run build",
    "deploy": "gh-pages -d build"
  ```

4. Push your changes to GitHub.
5. Run `npm run deploy` to deploy your dapp to a new GitHub Pages branch.
6. Go to your repository on github.com and follow the instructions: [![](https://raw.githubusercontent.com/dacadeorg/near-development-101/master/content/imgs/gh-pages.png)](https://raw.githubusercontent.com/dacadeorg/near-development-101/master/content/imgs/gh-pages.png).

- Click on settings.
- Navigate to the "Pages" section.
- Select the gh-pages branch as the default branch for your new page.
- Click "Save".

Now you are done and you should be able to see your dapp on `https://${GithubUsername}.github.io/${RepositoryName}`.

Awesome! You have successfully created your first NEAR dapp. Now you can go ahead and create your own dapp in our challenge and receive feedback and earn some NEAR!
