U ovom modulu učenja slijedit ćemo vodič za izgradnju frontenda za pametni ugovor. Ovaj vodič pretpostavlja da ste već dovršili [Spojite React Dapp s NEAR-om](/near101_chapter_2.md) modul učenja i nastavljate u istom projektu.

### Preduvjeti

- [Node JS](https://nodejs.org/en/download/) - Molimo provjerite imate li instaliran Node.js v16 ili noviju verziju.
- Trebali biste imati osnovno razumijevanje [React-a](https://reactjs.org/): znati kako koristiti JSX, props, stanja, lifecycle metode, i hooks.
- Trebali ste slijediti modul učenja [Spojite React Dapp sa NEAR-om](/near101_chapter_2.md) i imati instalirano `react`, `react-scripts` v.2.1.4, `near-api-js` i `uuid`.

### Tech Stack

Koristiti ćemo sljedeće tehnologije:

- [React](https://reactjs.org/) - JavaScript biblioteka za izgradnju korisničkih sučelja.
- [Bootstrap](https://getbootstrap.com/) - CSS okvir (framework).
- [near-api-js](https://docs.near.org/docs/api/javascript-library) - JavaScript/Typescript biblioteka za interakciju s NEAR-ovim blockchainom.

## 1. Postavljanje projekta

Budući da već imamo važne dependencies instalirane, sada samo trebamo dodati naše dependencies za frontend i stil:

```bash
npm install react-bootstrap bootstrap bootstrap-icons react-toastify
```

Koristit ćemo `react-bootstrap` za rukovanje `Bootstrap` stilom naših react komponenti. Također ćemo koristiti `react-toastify` za prikaz obavijesti korisniku, tako da to ne moramo sami rješavati.


### 1.1 index.js

Otvorimo `src/index.js` datoteku i počnimo dodavati naše bootstrap komponente i stilove:

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

Započinjemo ugovor pozivanjem funkcije `initializeContract` iz datoteke `utils/near.js` koju smo kreirali u [Spojite React Dapp sa NEAR-om](/near101_chapter_2.md) modulu učenja.


### 1.2 utils

`src/utils/` direktorij bi trebao izgledati ovako ako ste slijedili [Spojite React Dapp sa NEAR-om](/near101_chapter_2.md) modul učenja:

```
├── utils
│   ├── config.js
│   ├── marketplace.js
│   └── near.js
```

### 1.3 App.js

Postavit ćemo datoteku `App.js` za renderiranje našeg korisničkog sučelja. Otvorimo datoteku `src/App.js` i počnimo s uvozom:

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

Uvozimo funkcije prijavi se (`login`), odjavi se (`logout`) i stanje računa (`accountBalance`) iz datoteke `utils/near.js`. Također uvozimo komponente `Wallet``Cover`te datoteku sa slikama `coverImg`. Sve to još nije stvoreno.

Za sada će komponente (‘Notification’ i ‘Products’ ostati bez komentara jer ćemo ih implementirati kasnije.

Kreirajmo sada našu komponentu Aplikacija (`App`):

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

Dobivamo `account` kada smo spojeni na novčanik. Ako smo spojeni, dobivamo `accountId` i postavljamo stanje (`balance`). Pozivamo `accountBalance`funkciju iz `utils/near.js` datoteke da bismo dobili saldo.

Vratimo JSX za našu komponentu `App`:

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

Ako je korisnik povezan s novčanikom, prikazujemo naš dapp (decentraliziranu aplikaciju). Ako nisu, renderiramo komponentu `Cover`.

Prosljeđujemo`name`za naš dapp i `coverImg` kao props komponenti `Cover`. Također prosljeđujemo funkciju `login` za prijavu u novčanik.

Dapp se sastoji od komponente `Wallet`, koja prikazuje adresu i saldo korisničkog računa. Također prikazujemo komponentu `Products` koju ćemo kasnije implementirati.
`Wallet` treba adresu računa, saldo korisnika, simbol za valutu koju prikazujemo i funkciju `destroy` za odjavu iz novčanika kao prop.

Sada kreirajmo komponente koje smo već koristili.

## 2. Komponente

U ovom dijelu vodiča izradit ćemo prilagođene komponente koje ćemo koristiti u našem dapp-u.

Direktorij komponenti će izgledati ovako:


```
├── components
│   ├── marketplace
│   ├── utils
│   └── Wallet.js
```

Počet ćemo s komponentom `Cover`.

### 2.1 Cover.js

Napravite mapu komponente (`components`) u direktoriju `src` i kreirajte datoteku `src/components/utils/Cover.js` sa sljedećim kodom:

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

Ova komponenta je prilično jednostavna. Ako primi `name`, `login` i `coverImg` kao propertije, renderiramo `coverImg` i `name`) dapp-a. Također prikazujemo gumb `Connect Wallet` koji poziva funkciju `login` kada se klikne.

#### 2.1.1 Naslovna slika

Budući da koristimo naslovnu sliku, moramo uvesti datoteku sa slikama `coverImg`.
Za ovaj vodič odabrali smo sliku `sandwich.jpg` koju možete pronaći [ovdje](https://raw.githubusercontent.com/dacadeorg/near-marketplace-dapp/master/src/assets/img/sandwich.jpg ). Kreiramo dvije nove spremljene mape u direktoriju ‘assets’ i spremamo sliku `assets/img/sandwich.jpg`.


Sada nastavimo sa komponentom `Wallet`.

### 2.1 Wallet.js

Komponenta ‘Wallet’ će prikazati adresu korisničkog računa, saldo i gumb za odjavu. Napravite datoteku `src/components/Wallet.js` sa sljedećim kodom:

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

Primamo `address`, `amount` i `symbol` valute koje prikazujemo kao propertije. Također primamo funkciju `destroy` za odjavu iz novčanika. Kao što je ranije opisano, oni se prosljeđuju iz komponente `App`.

Sada bismo trebali biti spremni pokrenuti naš dapp i vidjeti možemo li se prijaviti, odjaviti i vidjeti svoj saldo i adresu.

Pokrenite dapp:

```bash
npm start
```

Aplikacija bi se sada trebala ponašati ovako:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/test_wallet_login.gif)

### 2.2 utils

Idemo sada raditi na nekim utility komponentama. Direktorij `utils` će izgledati ovako:

```
├── components
│   ├── utils
│   │   ├── Cover.js
│   │   ├── Loader.js
│   │   └── Notifications.js
```

Već imamo naslovnu komponentu (`Cover`). Kreirajmo komponentu za učitavanje, ( `Loader`).

#### 2.2.1 utils/Loader.js

Komponenta `Loader` će prikazati animaciju učitavanja. Napravite novu datoteku `components/utils/Loader.js` sa sljedećim kodom:

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

Ova komponenta je prilično jednostavna. Samo prikazuje bootstrap komponentu `Spinner`.

#### 2.2.2 utils/Notifications.js

Komponenta `Notification’ će prikazati obavijesti korisniku.
Izradite novu datoteku `components/utils/Notifications.js` sa sljedećim kodom:


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

Ova komponenta koristi biblioteku `react-toastify` za prikaz obavijesti.
Razlikujemo obavijesti o uspjehu i obavijesti o pogreškama, a inače prikazujemo "text" kao string.
Komponenta `Notification` implementirana je u komponentu `App`.

### 2.3 Tržnica

Sada kreiramo našu završnu komponentu, gdje ćemo kreirati korisničko sučelje za tržište. Direktorij `components/marketplace` izgledat će ovako:

```
├── components
│   ├── marketplace
│   │   ├── AddProduct.js
│   │   ├── Product.js
│   │   └── Products.js
│   ├── utils
│   └── Wallet.js
```

Počnimo s komponentom ‘Products’.

#### 2.3.1 Products.js

Komponenta‘Products’ prikazat će popis proizvoda.
To će biti glavna komponenta tržnice koja će sadržavati komponente `AddProducts` i `Product`.

Napravite novu mapu `marketplace` u direktoriju `components` i kreirajte datoteku `src/components/marketplace/Products.js`.

Počnimo s importanjem:

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

Importamo komponente `AddProducts i `Product` koje ćemo kreirati kasnije.
Također importamo komponente `Loader`, `NotificationSuccess` i `NotificationError` iz direktorija `utils`.
Konačno, importamo utility funkcije `getProductList`, `buyProduct` i `createProduct` iz direktorija `utils/marketplace`.

Kreirajmo našu glavnu komponentu `Products` i funkciju `getProducts` koju koristimo za dohvaćanje popisa proizvoda:

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

Stanje `loading` postavljamo na `true` kada dohvaćamo proizvode i postavljamo stanje `Products` na popis proizvoda. Za dohvaćanje proizvoda koristimo utility funkciju `getProductList` koji smo ranije uvezli.
Nakon što imamo proizvode, postavljamo stanje `loading` na `false`.

Zatim stvaramo funkciju `AddProduct`:

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

Podatke o proizvodu primamo kao parametar i koristimo utility funkciju `createProduct` za izradu proizvoda. Zatim ponovno preuzimamo proizvode i prikazujemo poruku o uspjehu ili poruku o pogrešci ako se proizvod ne može kreirati.

Konačno, kreiramo funkciju `buyProduct`:

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

Trebamo `id` i `price` proizvoda, i onda možemo pozvati utility funkciju `buyProduct` da bismo kupili proizvod. Zatim ponovno preuzimamo proizvode i prikazujemo poruku o uspjehu ili poruku o pogrešci ako se proizvod ne može kupiti.

Sada možete vratiti komponentu `Products` i napisati JSX kod:

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

U komponenti `AddProduct`, koja još nije stvorena, prosljeđujemo funkciju `addProduct` koju smo ranije kreirali kao prop.

Zatim mapiramo listu ‘products’ u komponentu `Product` koja još nije kreirana, a ona će prikazati pojedinačni proizvod kao karticu. Podatke `_product` prosljeđujemo kao props komponenti `Product` za renderiranje proizvoda. Također prosljeđujemo funkciju `buy` kao prop.

Konačno, exportamo komponentu `Products`.

Kreirajmo sada komponentu ‘Product’ o kojoj smo upravo govorili.

#### 2.3.2 Product.js

Ova datoteka će sadržavati komponentu ‘Product’, koja će prikazati pojedinačni proizvod kao karticu. U direktoriju `marketplace` kreirajte datoteku `src/components/marketplace/Product.js` sa sljedećim kodom:

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

Ova komponenta je prilično jednostavna. Dobivamo podatke o proizvodu: `id`, ‘price’, ‘name’, `description`, koliko je puta prodan (‘sold’), `location`, `image` i `owner` iz propa ‘Product’ .

U funkciji `triggerBuy` pozivamo funkciju `buy` koju smo proslijedili kao prop s `id` i `price` kao parametrima.

Zatim prikazujemo podatke o proizvodu kao bootstrap komponentu `Card` i pozivamo funkciju `triggerBuy` kada se klikne gumb za kupovinu (‘buy’).

U sljedećem dijelu kreirat ćemo komponentu ‘AddProduct’.

#### 2.3.3 AddProduct.js

Ova datoteka će sadržavati komponentu ‘AddProduct’, koja će prikazati obrazac za dodavanje novog proizvoda. U ‘marketplace’ kreirajte datoteku ‘src/components/marketplace/AddProduct.js’. Počet ćemo s naredbama za uvoz i komponentom AddProduct’:

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

Izrađujemo varijable stanja za podatke o proizvodu i booleovu varijablu stanja koju koristimo za pohranu stanja modala s obrascem za podatke o proizvodu.

U sljedećem koraku kreirat ćemo gumb i obrazac koji će se prikazati u skočnom modalnom prozoru:

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

Ova datoteka ima puno koda, ali je prilično jednostavna. Izrađujemo modal koji će se prikazati kada korisnik klikne na gumb ‘New Product’.
Unutar modala prikazujemo obrazac s poljima podataka o proizvodu.


Ako korisnik klikne na gumb ‘Save Product’, pozivamo funkciju ‘save’ koju smo proslijedili kao prop s podacima o proizvodu kao parametrima.

Završili smo s komponentama, sada moramo malo očistiti našu datoteku `App.js` i spremni smo za rad!


## 3. Završavanje dapp-a

U ovom posljednjem odjeljku postavit ćemo završne detalje na dapp.

### 3.1. Ažurirajte `App.js`

Završimo dapp dodavanjem komponenti ‘Products’ i Notification’ u našu datoteku `App.js`.

Počnimo s poništavanjem komentara uvoza komponenti `Products` i `Notification`, tako da to izgleda ovako:

```js
//...
import Wallet from "./components/Wallet";
import { Notification } from "./components/utils/Notifications";
import Products from "./components/marketplace/Products";
import Cover from "./components/utils/Cover";
//...
```

Sada moramo poništiti komentare komponente (‘<Notification />’) u JSX-u:

```js
//...
  return (
    <>
      <Notification />
      {account.accountId ? (
<!-- ... -->
```

Također moramo poništiti komentare komponente (‘<Products />’):

```js
//...
<main>
  <Products />
</main>
//...
```

Napravite završni test. Provjerite jesu li vaši proizvodi prikazani i možete li stvoriti novi proizvod i kupiti ga.

Dapp bi se sada trebao ponašati ovako:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/final_dapp.gif)

### 3.2. Čišćenje

Možemo malo i počistiti naš projekt. U našem direktoriju `src` možemo se riješiti datoteka `logo.svg` i `setupTests.js`.

U direktorij `public` možemo dodati naše datoteke `favicon.ico`, `logo192.png`, `logo512.png` i `manifest.json` koje odgovaraju našem projektu.

Također bismo trebali promijeniti ‘title’ i ‘description’ našeg dappa u datoteci `index.html` u direktoriju `public`.

### 3.3. Postavite na GitHub Pages

U posljednjem odjeljku ukratko ćemo pogledati kako postaviti naš dapp na GitHub Pages.

1. Prvo dodajte svoj projekt na GitHub. Ako ne znate kako to učiniti, pogledajte [GitHub vodič](https://docs.github.com/en/get-started/importing-your-projects-to-github/importing-source-code-to-github/adding-an-existing-project-to-github-using-the-command-line).

2.Instalirajte paket [gh-pages](https://www.npmjs.com/package/gh-pages). To će nam omogućiti da implementiramo našu dapp na GitHub Pages.

```bash
npm install gh-pages
```

3.U datoteci package.json dodajte sljedeće retke:

- Na vrh datoteke dodajte sljedeći redak:
  ```
  "homepage": "https://${GithubUsername}.github.io/${RepositoryName}",
  ```
 Zamijenite `${GithubUsername}` i `${RepositoryName}` svojim GitHub korisničkim imenom i imenom spremišta.
- dodajte sljedeće linije koda na dno odjeljka ‘scripts’:
  ```
    "predeploy": "npm run build",
    "deploy": "gh-pages -d build"
  ```

4. Prenesite svoje promjene na GitHub.
5. Pokrenite `npm run deploy` da implementirate svoj dapp na novu granu GitHub Pages.
6. Idite na svoje spremište na github.com i slijedite upute: [![](https://raw.githubusercontent.com/dacadeorg/near-development-101/master/content/imgs/gh-pages.png)](https://raw.githubusercontent.com/dacadeorg/near-development-101/master/content/imgs/gh-pages.png).

- Kliknite na postavke.
- Idite na odjeljak ‘Pages’.
- Odaberite granu gh-pages kao zadanu granu za svoju novu stranicu.
- Kliknite "Save".

Sada ste gotovi i trebali biste moći vidjeti svoju dapp na `https://${GithubUsername}.github.io/${RepositoryName}`.

Super! Uspješno ste stvorili svoj prvi NEAR dapp. Sada možete ići korak naprijed i stvoriti svoj vlastiti dapp u našem izazovu, primati povratne informacije i zaraditi NEAR!

A sada, idemo objaviti vaš projekt! Ako ste iz NEAR Balkans zajednice, molimo upišite informacije i repozitorij svog projekta putem linka da biste dobili NEAR certifikat:

[![](https://raw.githubusercontent.com/dacadeorg/near-development-101/master/content/imgs/button-submit-hr.png)](https://9ybz5j7dvgu.typeform.com/to/Senm7LQw)
