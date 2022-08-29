В този учебен модул ще разгледаме урок за изграждане на интерфейс за “магазин”, качен в NEAR средата. Урокът предполага, че вече имате завършен [Connect a React Dapp to NEAR](/near101_chapter_2.md) учебен модул и продължавате в същия проект. 

###Предпоставки
- [Node JS](https://nodejs.org/en/download/) - Моля уверете се че имате Node.js v16 или по-висока версия инсталирана. 
- Трябва да имате основни знания за [React](https://reactjs.org/): да знаете как се борави с JSX, props, state, lifecycle methods и hooks.
- Трябва да сте преминали [Connect a React Dapp to NEAR](/near101_chapter_2.md) учебните уроци и да имате `react`, `react-scripts` v.2.1.4, `near-api-js` and `uuid` инсталирани.

### Технологии 
Ще използваме следните инструменти:
-- [React](https://reactjs.org/) - JavaScript библиотека за изграждане на потребителски интерфейс. 
-  [Bootstrap](https://getbootstrap.com/) - CSS структура.
- [near-api-js](https://docs.near.org/docs/api/javascript-library) - JavaScript/Typescript библиотека за взаимодействие с NEAR блокчейна.

### Настройка на проекта

Тъй като вече имаме важните зависимости инсталирани, сега единствено следва да добавим нашите зависимости за визуалната част и стилове.
```bash
npm install react-bootstrap bootstrap bootstrap-icons react-toastify
```
Ще използваме `react-bootstrap` , за да поддържаме `Bootstrap` стила накомпонентите. Освен това ще използваме react-toastify` за да показваме известия за потребителя, така че да не го правим собственоръчно. 

### 1.1 index.js
Нека отворим `src/index.js`  файла и да започнем да добавяме нашия начален компонент и стилове.
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

Стартираме договора като ползваме 'initializeContract` функцията от `utils/near.js`  файла, който създадохме в учебния модул [Connect a React Dapp to NEAR](/near101_chapter_2.md).

### 1.2 utils
`src/utils/`  папката следва да изглежда по този начин ако сте следвали модула[Connect a React Dapp to NEAR](/near101_chapter_2.md):
```
├── utils
│   ├── config.js
│   ├── marketplace.js
│   └── near.js
```

### 1.3 App.js
Ще настроим `App.js` файла да изобрязява нашия потребителски интерфейс. Нека отворим `src/App.js`  файла и да започнем с импортирането:
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

 
Импортираме `login`, `logout` и `accountBalance` функциите от `utils/near.js` файла. Също така импортираме `Wallet` и `Cover` компонентите и `coverImg` изображението. Всички от тях все още не са създадени.
За сега, `Notification` и `Products` компонентите ще останат коментирани, тъй като ще ги имплементираме по-късно. 
Сега, нека създадем `App`  компонентът:
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

Получаваме `account`, когато сме свързани с портфейла. Ако сме свързани, получаваме `accountId` и настройваме`balance`. Извикваме 'accountBalance`  функцията от utils/near.js`  файла, за да получим баланса.
Нека се върнем към JSX за нашия `App` компонент:
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

Ако потребителят е свързан с портфейла, показваме нашия dapp. Ако не е, изобразяваме `Cover` компонента. 
 
Задаваме `name` на нашия dapp и `coverImg` като аргументи на `Cover` компонента. Подаваме и `login`функция за влизане в портфейла. 
 
Приложението се състои от компонента `Wallet`, който показва адреса на сметката и баланса на потребителя. Показваме и компонента „Products“, който ще разпишем по-късно.
„Wallet“ се нуждае от адреса на акаунта, баланса на потребителя, символа за валутата, която показваме, и функцията „destroy“, за да излезете от портфейла, като аргументи.
 
Сега, нека създадем компонентите, които вече използвахме.

2. Компоненти

В тази секция от урока ще създадем персонализирани компоненти, които ще използваме за нашия dapp.
Папката на компонентите ще изглежда по следния начин:
```
├── components
│   ├── marketplace
│   ├── utils
│   └── Wallet.js
```

### 2.1 Cover.js
Създайте папка „components“ в директорията „src“ и създайте файл „src/components/utils/Cover.js“ със следния код:

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




Този компонент е доста елементарен. Ако получим `name`, `login` и `coverImg` като параметри, ние изобразяваме `coverImg` и `name` на dapp. Ние също така показваме бутон „Connect wallet“, който извиква функцията „Login“, когато се кликне.

#### 2.1.1 Cover Image

Тъй като използваме изображение на корицата, трябва да импортираме файла с изображение `coverImg`.
За този урок избрахме изображение „sandwich.jpg“, което можете да намерите (https://raw.githubusercontent.com/dacadeorg/near-marketplace-dapp/master/src/assets/img/sandwich.jpg ). Създаваме две нови папки в директорията “assets”и съхраняваме изображението `assets/img/sandwich.jpg`.
 
Сега нека продължим с компонента `Wallet`.


### 2.1 Wallet.js



Компонентът на портфейла ще покаже адреса на акаунта на потребителя, баланса и бутона за излизане. Създайте файл `src/components/Wallet.js` със следния код:



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





Получаваме „address“, баланса на потребителя („ammount“) и „symbol“ на валутата, която показваме като параметри. Получаваме и функция `destroy` за излизане от портфейла. Както беше описано по-рано, те се предават от компонента `App`.
 
Сега трябва да сме готови да стартираме нашето dapp приложение и да видим дали можем да влезем, да излезем и да видим нашия баланс и адрес.
 
Стартирайте dapp:
```bash
npm start
```




Сега приложението трябва да изглежда така:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/test_wallet_login.gif)

### 2.2 utils
Нека да поработим по-нататък върху някои помощни компоненти. Директорията `utils` ще изглежда така:
```
├── components
│   ├── utils
│   │   ├── Cover.js
│   │   ├── Loader.js
│   │   └── Notifications.js
```

Вече имаме компонента `Cover`. Нека създадем компонента `Loader`.

#### 2.2.1 utils/Loader.js

Компонентът `Loader` ще покаже анимация за зареждане. Създайте нов файл `components/utils/Loader.js` със следния код:
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
Този компонент е доста прост. Той просто показва компонента `Spinner` за зареждане.

#### 2.2.2 utils/Notifications.js
 
Компонентът `Notification` ще показва известия на потребителя.
Създайте нов файл `components/utils/Notifications.js` със следния код:
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



Този компонент използва библиотеката `react-toastify` за показване на известия.
Разграничаваме уведомленията за успех и грешки и иначе показваме „текста“ като низ.
Компонентът `Notification` е имплементиран в компонента `App`.

### 2.3 marketplace

Сега ще създадем нашия последен компонент, където ще изготвим потребителския интерфейс за магазина. Директорията `components/marketplace` ще изглежда по следния начин:
```
├── components
│   ├── marketplace
│   │   ├── AddProduct.js
│   │   ├── Product.js
│   │   └── Products.js
│   ├── utils
│   └── Wallet.js
```

Нека започнем с ‘Products’ компонента.

#### 2.3.1 Products.js
Компонентът „products“ ще показва списък с продукти.
Това ще бъде основният компонент на магазина, който ще съдържа компонентите `AddProducts` и `Product`.
Създайте нова папка `marketplace` в директорията `components` и създайте файл `src/components/marketplace/Products.js`.
 
Да започнем с импортирането:



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



Импортираме компонентите `AddProduct` и `Product`, които ще създадем по-късно.
Също така импортираме компонентите `Loader` и `NotificationSuccess` и `NotificationError` от директорията `utils`.
И накрая, импортираме помощните функции `getProductList`, `buyProduct` и `createProduct` от директорията `utils/marketplace`.
 
Нека създадем нашия основен компонент `Products` и функция `getProducts`, която използваме, за да извлечем списъка с продукти:



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

Вдигаме флага на „loading“, когато извличаме продуктите, и задаваме състоянието „products“ със списъка с продукти, който е извлечен. За да извлечем продуктите, използваме помощната функция `getProductList`, която импортирахме по-рано.
След като имаме продуктите, задаваме състоянието на „loading“ на „false“.
 
След това създаваме функцията `AddProduct`:




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
​​Получаваме данните за продукта като параметър и използваме помощната функция `createProduct`, за да създадем продукта. След това извличаме продуктите отново и показваме съобщение за успех или съобщение за грешка, ако продуктът не може да бъде създаден.
 
Накрая създаваме функцията `buyProduct`:



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

Имаме нужда от `id` и `price` на продукта и можем да извикаме помощната функция `buyProduct`, за да закупим продукта. След това извличаме продуктите отново и показваме съобщение за успех или съобщение за грешка, ако продуктът не може да бъде закупен.
 
Вече може да се върнем към компонента „Products“ и да напишем JSX кода:



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

В все още несъздадения компонент `AddProduct` предаваме функцията `addProduct`, която създадохме по-рано като аргумент.
 
След това свързваме списъка с „products“ към все още несъздадения компонент „Продукт“, който е компонент, който ще покаже отделния продукт като карта. Предаваме данните на `_product` като аргументи към компонента `Product`, за да изобразим продукта. Ние също предаваме функцията `buy` като аргумент.
 
Накрая експортираме компонента „Products“.
 
Нека сега създадем компонента „product“, който току-що обсъдихме.

#### 2.3.2 Product.js


Този файл ще съдържа компонента „Product“, който ще показва отделния продукт. В директорията `marketplace` създайте файл `src/components/marketplace/Product.js` със следния код:
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


Този компонент е доста прост. Получаваме данните за продукта: `id`, `price`, `name`, `description`, колко пъти е `sold`, `location`, `image` и `owner` от обекта `product` .
Във функцията `triggerBuy` извикваме функцията `buy`, която сме предали като аргумент с `id` и `price` на продукта като параметри.
След това показваме данните за продукта като начален компонент „Card“ и извикваме функцията „triggerBuy“, когато се кликне върху бутона Buy“.
В следващия раздел ще създадем компонента `AddProduct`.

#### 2.3.3 AddProduct.js

Този файл ще съдържа компонента `AddProduct`, който ще покаже формуляр за добавяне на нов продукт. В директорията `marketplace` създайте файл `src/components/marketplace/AddProduct.js`. Ще започнем с инструкциите за импортиране и компонента `AddProduct`: 
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

Създаваме променливи за данните за продукта и булева променлива на състоянието, която използваме за съхраняване на състоянието на модела с формуляра за данните за продукта.

В следващата стъпка ще създадем бутона и формуляра, които ще се показват в изскачащия диалогов прозорец:
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

Този файл има много код, но е доста ясен. Създаваме диалогов прозорец, който ще се показва, когато потребителят кликне върху бутона „Нов продукт“.
Вътре в него показваме форма с полетата с данни за продукта.
Ако потребителят кликне върху бутона „Save product“, ние извикваме функцията „save“, която сме задали като аргумент с данните за продукта като параметри.
Приключихме с компонентите, сега трябва да направим малко разчистване в нашия файл `App.js` и сме готови!

## 3. Завършване на dapp
В този последен раздел ще поставим последните щрихи върху dapp
### 3.1. Актуализация на  `App.js`

Нека завършим dapp, като добавим нашите компоненти `Products` и `Notification` към нашия `App.js` файл.

Нека започнем, като разкоментираме импортирането на компонентите `Products` и `Notification`, така че да изглежда така:
```js
//...
import Wallet from "./components/Wallet";
import { Notification } from "./components/utils/Notifications";
import Products from "./components/marketplace/Products";
import Cover from "./components/utils/Cover";
//...
```

Сега трябва да махнем коментарите за компонента `<Notification />` в JSX:
```js
//...
  return (
    <>
      <Notification />
      {account.accountId ? (
<!-- ... -->
```

Също така трябва да разкоментираме компонента „<Products />“:
```js
//...
<main>
  <Products />
</main>
//...
```

Направете последен тест. Вижте дали вашите продукти се показват и дали можете да създадете нов продукт и да го купите.

Dapp сега трябва да се държи така:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/final_dapp.gif)

### 3.2. Разчистване

Можем да направим малко разчистване в нашия проект. В нашата директория `src` можем да се отървем от файловете `logo.svg` и `setupTests.js`.

В директорията „public“ можем да добавим нашите файлове „favicon.ico“, „logo192.png“, „logo512.png“ и „manifest.json“, които отговарят на нашия проект.

Трябва също така да променим `title` и `description` на нашия dapp във файла `index.html` в директорията `public`.


### 3.3. Внедряване в GitHub Pages
В последния раздел ще разгледаме накратко как да внедрим нашия dapp в GitHub Pages.

1. Първо добавете вашия проект към GitHub. Ако не знаете как да направите това, вижте това [GitHub guide](https://docs.github.com/en/get-started/importing-your-projects-to-github/importing-source-code-to-github/adding-an-existing-project-to-github-using-the-command-line).



2. Инсталирайте пакета [gh-pages](https://www.npmjs.com/package/gh-pages). Това ще ни позволи да внедрим нашия dapp в GitHub Pages.

```bash
npm install gh-pages
```

3. Във файла package.json добавете следното:

- В горната част на файла добавете:
  ```
  "homepage": "https://${GithubUsername}.github.io/${RepositoryName}",
  ```

Заменете `${GithubUsername}` и `${RepositoryName}` с вашето потребителско име в GitHub и името на хранилището.
- добавете редове в долната част на секцията `scripts`: 
  ```
    "predeploy": "npm run build",
    "deploy": "gh-pages -d build"
  ```

4. Изпратете вашите промени в GitHub.
5. Стартирайте `npm run deploy`, за да качите вашето dapp в нов раздел на GitHub Pages.6. Отидете до вашето хранилище на github.com и следвайте инструкциите:
[![](https://raw.githubusercontent.com/dacadeorg/near-development-101/master/content/imgs/gh-pages.png)](https://raw.githubusercontent.com/dacadeorg/near-development-101/master/content/imgs/gh-pages.png).

- Кликнете върху настройките.
- Отидете до секцията „pages“.
- Изберете клона gh-pages като клон по подразбиране за вашата нова страница.
- Щракнете върху „Save “.
Сега сте готови и трябва да можете да видите вашия dapp на
`https://${GithubUsername}.github.io/${RepositoryName}`.

Страхотно! Успешно създадохте първaта си NEAR апликация. Сега можете да продължите напред и да създадете своя собствена в нашето предизвикателство и да получите обратна връзка и да спечелите NEAR!

Сега, нека публикуваме проекта ти. Ако си част от сървъра на NEAR Balkans, изпрати твоите имена (+псевдоним) и линк към товя проект на този линк за да получите NEAR сертификат:

[![](https://raw.githubusercontent.com/dacadeorg/near-development-101/master/content/imgs/button-submit-bg.png)](https://forms.gle/KPJ5ynvDdjKtuEHe9)
