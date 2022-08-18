U ovom modulu učenja slijedit ćemo vodič za povezivanje s ugovorom koji predstavlja tržište na NEAR testnetu.

### Preduvjeti

- Do sada ste trebali izraditi i implementirati NEAR pametni ugovor za tržište kao što je opisano u našem modulu za učenje [AssemblyScript contract development](/near101_chapter_1.md).
- [Node JS](https://nodejs.org/en/download/) - Molimo provjerite imate li instaliranu Node.js v16 ili noviju verziju.
- Trebali biste imati osnovno razumijevanje React-a [React](https://reactjs.org/): te znati koristiti JSX, props, state, lifecycle metode i hooks.

### Tech Stack

Koristit ćemo sljedeći tehnološki skup:

- [React](https://reactjs.org/) - JavaScript library za izgradnju korisničkih sučelja.
- [near-api-js](https://docs.near.org/docs/api/javascript-library) - JavaScript/Typescript library za interakciju s NEAR-ovim blockchainom.

## 1. Postavljanje projekta

U prvom dijelu ovog vodića postaviti ćemo projekt i instalirati potrebne ovisnosti.

Molimo provjerite imate li instaliranu Node.js v16 ili noviju verziju.


```bash
node -v
```


Koristit ćemo utility komandu `create-react-app` za izradu novog react projekta:


```bash
npx create-react-app near-marketplace
```

Putem komande cd navigirajte do kreiranog projekta:

```bash
cd near-marketplace
```


Nažalost, "react-scripts" verzije 5 možda neće raditi s najnovijom verzijom node-a, pa bismo trebali koristiti "react-scripts" verzije 4.0.3:

```bash
npm install react-scripts@4.0.3
```


Takoder moramo instalirati i `near-api-js` library:

```bash
npm install near-api-js
```


Konačno, instalirat ćemo library `uuid` koji se koristi za generiranje jedinstvenih ID-ova za naše proizvode:

```bash
npm install uuid
```

To je to! Sada možemo pokrenuti projekt i provjeriti radi li sve kako bi trebalo:

```bash
npm start
```

## 2. Povezivanje na NEAR

Sada kada imamo projekt, možemo postaviti našu vezu s NEAR mrežom i s našim pametnim ugovorom.

### 2.1 Konfiguracija

Izrađujemo mapu `utils` u direktoriju `src` s datotekom `src/utils/config.js` kako bismo definirali konfiguraciju za našu vezu s NEAR mrežom:

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


U prvom retku definiramo naziv pametnog ugovora s kojim želimo komunicirati. To je zapravo naziv računa na koji je pametni ugovor postavljen.
Koristimo kreiranje ugovora u našem modulu za učenje [AssemblyScript ugovor](/near101_chapter_1.md), tako da bi varijabla `${CONTRACT_NAME}` bila `mycontract.myaccount.testnet`. Zamijenite je s ID-om računa na kojem je vaš pametni ugovor postavljen.

U redovima 5 i 14 definiramo različita okruženja na koja se možemo povezati. U ovom vodiču koristit ćemo samo testnet.

### 2.2 Povezivanje na NEAR

U ovom odjeljku spojit ćemo se na NEAR mrežu i naš pametni ugovor.
Napravite datoteku `src/utils/near.js`. Prvo napravimo naš import i definirajmo okruženje:

```js
import environment from "./config";
import { connect, Contract, keyStores, WalletConnection } from "near-api-js";
import { formatNearAmount } from "near-api-js/lib/utils/format";

const nearEnv = environment("testnet");
//...
```

Sada kreiramo funkciju za inicijalizaciju našeg ugovora:

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

Stvaramo `near` objekt koji ćemo koristiti za interakciju s NEAR mrežom. Taj objekt sadrži drugi objekt `keyStore` koji pohranjuje podatke o novčaniku, ti podatci su pohranjeni u lokalnoj memoriji internet preglednika.

Zatim kreiramo objekt `WalletConnection` koji ćemo koristiti za interakciju s novčanikom. Kako bismo mogli prijaviti se, odjaviti se, dohvatiti ID računa i dohvatiti stanje računa.

Izrađujemo objekt `Contract` koji ćemo koristiti za interakciju s pametnim ugovorom. Konstruktoru prosljeđujemo račun, naziv pametnog ugovora i metode koje želimo koristiti.

Kada razvijamo s NEAR-om, nije nam potreban ABI, kao što nam je potreban za Ethereum ugovore. Možemo samo koristiti svojstva `viewMethods` i `changeMethods` objekta `Contract` da definiramo metode koje želimo koristiti. U ovom slučaju imamo niz `viewMethods` koji ne mijenjaju stanje (`["getProduct", "getProducts"]`) i niz `changeMethods` koji mijenjaju stanje (`["buyProduct", "setProduct"]`).

Konačno, kreirat ćemo neke funkcije za interakciju s novčanikom:

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

U ovim funkcijama koristimo `walletConnection` za dobivanje stanja na računu i ID računa te povezujemo i odspajamo naš dapp sa novčanikom.

## 3. Implementacija tržišta

Sada kada smo postavili našu vezu s NEAR blockchainom i našim pametnim ugovorom, možemo implementirati funkcionalnost tržišta.

### 3.1 Funkcionalnost tržišta

Izrađujemo datoteku `src/utils/marketplace.js` koja će sadržavati funkcije koje ćemo koristiti za interakciju s našim pametnim ugovorom:




























S funkcijom `createProduct` stvaramo novi proizvod. Koristimo funkciju `uuid4` za generiranje jedinstvenog ID-a za proizvod i funkciju `parseNearAmount` za pretvaranje cijene u ispravan format.
Zatim koristimo funkciju `setProduct` za stvaranje proizvoda s objektom `product`.

Funkcija `getProducts` vraća sve proizvode pohranjene u pametnom ugovoru.

Konačno, za kupnju proizvoda koristimo funkciju `buyProduct`. Mi prosljeđujemo ID proizvoda, količinu gasa za korištenje i cijenu proizvoda.

Za konstantu `GAS` koristimo tvrdo kodiranu vrijednost `100000000000000`, što je 100 TGas-a (terra gas). Ako se ne potroši sav gas, ostatak će biti vraćen na račun. Ako želite znati kako možete izračunati približnu cijenu gasa za transakciju, možete pogledati  [Near documentation](https://docs.near.org/docs/concepts/gas#the-cost-of-common-actions).

### 3.2 Dodavanje tržišta u aplikaciju

Našoj ćemo aplikaciji dodati funkcionalnost tržišta. Otvorite datoteku `src/App.js` i promijenite kod na sljedeći:

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

Ovaj kod služi samo u svrhu testiranja. Pokušavamo se spojiti na novčanik, te ako smo povezani, preuzimamo proizvode. Zatim prikazujemo proizvode u konzoli, koristeći funkciju `getProducts` koju smo upravo stvorili.
Ako nismo povezani, prikazujemo gumb koji će se povezati s novčanikom. Za to koristimo uslužnu funkciju `login` iz datoteke `utils/near.js`.

### 3.2 Ažuriranje datoteke index.js

Također moramo ažurirati datoteku `src/index.js` kako bismo inicijalizirali ugovor:

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

Ovdje koristimo utility funkciju `initializeContract` iz datoteke `utils/near.js` za inicijalizaciju ugovora. Koristimo varijablu `window.nearInitPromise` kako bismo bili sigurni da se aplikacija ne generira dok se ugovor ne inicijalizira.

Sada možete pokrenuti aplikaciju:

```bash
npm start
```

Trebali biste vidjeti nešto poput ovoga:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/login_and_print_products_to_console.gif)

Sjajno! Sada možete vidjeti proizvode u konzoli. Nastavite sljedeći modulu učenja kako biste naučili kako izgraditi frontend komponente za svoju decentraliziranu aplikaciju tržišta (dapp).