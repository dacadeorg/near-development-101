В този модул ще продължим с урок как да се свържем със приложение, качено в тестовата мрежа на NEAR.

### Предпоставки

- Трябва да сте създали и качили смарт договор “магазин”, както е описано в нашия учебен модул [разработване на договор на AssemblyScript](/near101_chapter_1.md).
- [Node JS](https://nodejs.org/en/download/) - Моля, уверете се, че имате инсталиран Node.js v16 или по-нова версия.
- Трябва да имате основни познания за [React](https://reactjs.org/): да знаете как да използвате JSX, props,state, lifecycle methods и hooks.

### Технологии

Ще използваме следните технологии:

- [React](https://reactjs.org/) - JavaScript библиотека за изграждане на потребителски интерфейси.
- [near-api-js](https://docs.near.org/docs/api/javascript-library) - JavaScript/Typescript библиотека за взаимодействие с блокчейна на NEAR.

## 1. Настройка на проекта

В първия раздел на този урок ще настроим проекта и ще инсталираме необходимите зависимости.

Уверете се, че имате инсталиран `nodejs` v16 или по-нова версия:
```bash
node -v
```
Ще използваме помощната програма `create-react-app`, за да създадем нов проект на react:
```bash
npx create-react-app near-marketplace
```
Нека влезем в новосъздадения проект:
```bash
cd near-marketplace
```

За съжаление, `react-scripts` от версия 5 може да не работят с най-новата node версия, така че трябва да използваме `react-scripts` от версия 4.0.3:
```bash
yarn add react-scripts@4.0.3
```

Също така трябва да инсталираме библиотеката `near-api-js`:
```bash
yarn add near-api-js
```

Накрая ще инсталираме библиотеката `uuid`, която се използва за генериране на уникални идентификатори за нашите продукти:

```bash
yarn add uuid
```

Това е! Сега можем да започнем проекта и да видим дали всичко работи:

```bash
yarn start
```

Ако се сблъскате с някакъв проблем, свързан с надпис `Uncaught (in promise) ReferenceError: Buffer is not defined` в браузъра, моля, следвайте стъпките по-долу

```
yarn add -D buffer
```

**Сега импортирайте `Buffer` от буфер и добавете този импортиран `Buffer` към глобалния обект на браузъра.

```js
import {Buffer} from 'buffer'
```

Добавете обекта `Buffer` към глобалния обхват на браузъра

```js
global.Buffer = Buffer
```

## 2. Свързване с NEAR

След като имаме проект, можем да се свържем с мрежата NEAR и нашия смарт договор.

### 2.1 Config

Създаваме папка `utils` в директорията `src` с файл `src/utils/config.js`, за да дефинираме конфигурацията за нашата връзка към мрежата NEAR:

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

На ред 1 дефинираме името на смарт договора, с който искаме да взаимодействаме. Това е името на акаунта, в който е каченсмарт договорът.
Ние използваме договора, създаден в нашия модул за обучение [AssemblyScript contract development](/near101_chapter_1.md), така че променливата `${CONTRACT_NAME}` ще бъде `mycontract.myaccount.testnet`. Заменете го с accountID на акаунта, в който е качен вашият смарт договор.

В редове 5 и 14 ние дефинираме различните среди, към които можем да се свържем. В този урок ще използваме само тестовата мрежа.

### 2.2 Свързване към NEAR

В този раздел ще се свържем с мрежата NEAR и нашия смарт договор.
Създайте файл `src/utils/near.js`. Нека първо направим нашите импорти и дефинираме средата:

```js
import environment from "./config";
import { connect, Contract, keyStores, WalletConnection } from "near-api-js";
import { formatNearAmount } from "near-api-js/lib/utils/format";

const nearEnv = environment("testnet");
//...
```

Сега създаваме функция за инициализиране на нашия договор:

```js
//...
export async function initializeContract() {
  const near = await connect(
    Object.assign(
      { keyStore: new keyStores.BrowserLocalStorageKeyStore() },
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

Ние създаваме „near“ обект, който ще използваме за взаимодействие с мрежата NEAR. Той съдържа обект „keyStore“, който съхранява информацията за портфейла, която се съхранява в локалната памет на браузъра.

След това създаваме обект `WalletConnection`, който ще използваме за взаимодействие с портфейла. За да направите автентикация, вземете ID на акаунта и балансът на акаунта.

Създаваме обект „Contract“, който ще използваме за взаимодействие със смарт договора. Подаваме акаунта, името на смарт договора и методите, които искаме да използваме на конструктора.

Когато работим с NEAR, не се нуждаем от ABI, както се нуждаем от договорите на Ethereum. Можем просто да използваме свойствата `viewMethods` и `changeMethods` на обекта `Contract`, за да дефинираме методите, които искаме да използваме. В този случай имаме масив от `viewMethods`, който не променя състоянието (`["getProduct", "getProducts"]`) и масив от `changeMethods`, който променя състоянието (`["buyProduct", "setProduct"]`).

И накрая, ще създадем някои функции за взаимодействие с портфейла:

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
  return window.walletConnection.requestSignIn({
    contractId: env.contractName,
  });
}

export function logout() {
  window.walletConnection.signOut();
  window.location.reload();
}
```

В тези функции използваме „walletConnection“, за да получим балансът на акаунта и идентификатора на акаунта, и свързваме и изключваме нашия dapp от портфейла.

## 3. Внедряване на приложение

Сега, след като се свързахме с блокчейна NEAR и нашия смарт договор, можем да внедрим функционалността на “магазина”.

### 3.1 Функционалност на “магазина”

Създаваме файл `src/utils/marketplace.js`, който ще съдържа функциите, които ще използваме за взаимодействие с нашия смарт договор:

```js
import { v4 as uuid4 } from "uuid";
import { parseNearAmount } from "near-api-js/lib/utils/format";

const GAS = 100000000000000;

export function createProduct(product) {
  product.id = uuid4();
  product.price = parseNearAmount(product.price + "");
  return window.contract.setProduct({ product });
}

export function getProducts() {
  return window.contract.getProducts();
}

export async function buyProduct({ id, price }) {
  await window.contract.buyProduct({ productId: id }, GAS, price);
}
```

С функцията `createProduct` създаваме нов продукт. Използваме функцията `uuid4`, за да генерираме уникален идентификатор за продукта, и функцията `parseNearAmount`, за да конвертираме цената в правилния формат.
След това използваме функцията `setProduct`, за да създадем продукта с обекта `product`.

Функцията getProducts връща всички продукти, съхранени в смарт договора.

И накрая, използваме функцията `buyProduct`, за да купим продукт. Изпращаме идентификатора на продукта, количеството gas и цената на продукта.

За константата „GAS“ използваме дефинираната константа  „100000000000000“, която е 100 TGas (терра газ). Ако не е използван целият газ, остатъкът ще бъде възстановен по сметката. Ако искате да знаете как можете да изчислите приблизителните разходи за газ за дадена транзакция, можете да погледнете [Near documentation](https://docs.near.org/docs/concepts/gas#the-cost-of-common-actions).


### 3.2 Добавяне на “магазин” към приложението

Ще добавим функционалността на магазина към нашето приложение. Отворете файла `src/App.js` и променете кода на следното:

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
Този код е само за тестови цели. Опитваме се да се свържем с портфейла и ако сме свързани, извличаме продуктите. След това показваме продуктите в конзолата, като използваме функцията `getProducts`, която току-що създадохме.
Ако не сме свързани, показваме бутон, който ще се свърже с портфейла. За целта използваме помощната функция `login` от файла `utils/near.js`.

### 3.2 Актуализиране на файла index.js

Също така трябва да актуализираме файла `src/index.js`, за да инициализираме договора:
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

Тук използваме помощната функция `initializeContract` от файла `utils/near.js` за инициализиране на договора. Използваме променливата `window.nearInitPromise`, за да сме сигурни, че приложението няма да се покаже, докато договорът не бъде инициализиран.

Сега можете да стартирате приложението:

```bash
yarn start
```

Трябва да видите нещо подобно:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/login_and_print_products_to_console.gif)

Страхотно! Вече можете да видите продуктите в конзолата. Продължете със следващия учебен модул, за да научите как да създавате fronted components за вашия пазарен dapp.


