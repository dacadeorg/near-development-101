Ovaj modul sastoji se od vodiča kroz koji ćete naučiti kako napraviti pametan ugovor za decentralizirano tržište na NEAR blockchainu.

Koristit ćemo AssemblyScript za pisanje naših NEAR pametnih ugovora. Pisanje  AssemblyScript koda je veoma slično pisanju TypeScript koda; međutim, postoje određene specifičnosti vezane isključivo za pisanje NEAR pametnih ugovora u AssemblyScript-u, koje ćemo proći u narednim sekcijama. 


### Preduvjeti

- Imati određeno osnovno znanje o Blockchain tehnologiji I pametnim ugovorima. 
- Imati određeno osnovno znanje u JavaScriptu.  
- Biti spreman na korištenje terminala.

### Tech Stack

Koristit ćemo sljedeći tech stack:

- [near-cli](https://www.npmjs.com/package/near-cli) - CLI alat za NEAR koji nudi API s pametnim ugovorima. 
- [assemblyscript](https://www.npmjs.com/package/assemblyscript) – Jezik sličan TypeScript-u za WebAssembly.
- [asbuild](https://www.npmjs.com/package/asbuild) – Alat za izgradnju na AssemblyScript-u. 

## 1. Postavke

U ovoj prvoj sekciji tutorijala, podesit ćemo razvojno okruženje i projekt.

Neophodno je imati instaliranu verziju node.js koja je viša ili jednaka 12.
Također koristit ćemo package manager yarn, pa provjerite imate li ga instaliranog. 

### 1.1 Instaliranje CLI Alata

Možete globalno instalirati najnoviju verziju CLI alata pokretanjem sljedećih naredbi: 

```bash
yarn global add near-cli
yarn global add assemblyscript
yarn global add asbuild
```

Predlažemo korištenje editora koji podržava code completion i syntax highlighting za TypeScript kako bi vam pomogao napisati AssemblyScript kod, kao Visual Studio Code ili Atom. Koristit ćemo Visual Studio Code za ovaj modul učenja. 

### 1.2 Postavka projekta

Sada ćemo postaviti naš projekt. Molim vas kreirajte novi root direktorij i dajte mu naziv sličan sljedećem: `near-marketplace-contract`.

#### 1.2.1 `asconfig.json`

U nasem root direktoriju, treba napraviti datoteku pod nazivom `asconfig.json`. Ova config datoteka pruža CLI mogućnosti I konfiguraciju za AssemblyScript.

Za prevodenje s `near-sdk-as`, potrebno je dodati sljedeće u `asconfig.json` datoteku:

```javascript
{
    "extends": "near-sdk-as/asconfig.json"
}
```

#### 1.2.2 `tsconfig.json`

Kreiramo `assembly` direktorij i `assembly/tsconfig.json` datoteku unutar njega. Cilj ove datoteke je navesti “compiler” mogućnosti I “root level” datoteke koji su neophodni za prevodenje TypeScript projekata.

`tsconfig.json` datoteka treba biti u root direktoriju TypeScript projekta.

```javascript
{
  "extends": "../node_modules/assemblyscript/std/assembly.json",
  "include": [
    "./**/*.ts"
  ]
}
```

#### 1.2.3 `as_types.d.ts`

Kreiramo `assembly/as_types.d.ts` datoteku u nasem `assembly` direktoriju. Ova datoteka se koristi da bi se definirali tipovi  koji su korišteni u našem AssemblyScript kodu. 
U ovom slučaju, “importamo” tipove iz `near-sdk-as`.

```javascript
/// <reference types="near-sdk-as/assembly/as_types" />
```

### 1.3 Inicijaliziranje projekta

Pokrenuti sljedeću naredbu za inicijaliziranje projekta u root direktoriju:

```bash
yarn init
```

To će kreirati package.json datoteku gdje su popisani dependencies.
Pokrenuti sljedeću naredbu da bi dodali `near-sdk-as` projektu:

```bash
yarn add -D near-sdk-as
```

Korisnici Mac M1 računala mogu se susresti s `Error: Unsupported platform: Darwin arm64` greškom prilikom pokušaja instaliranja.


Drugo rješenje bi bilo da se pokrene prethodna naredba s`--ignore-scripts` opcijom:
```
yarn add -D --ignore-scripts near-sdk-as
```

U posljednjem koraku procesa inicijaliziranja, treba kreirati ulaznu datoteku za pametni ugovor. Kreirati datoteku pod nazivom `index.ts` u `assembly` directory.

Konačna struktura projekta bi trebalo ovako izgledati:


```
├── asconfig.json
├── assembly
│   ├── as_types.d.ts
│   ├── index.ts
│   └── tsconfig.json
├── package.json
└── yarn.lock
```

## 2. Memorija pametnog ugovora

U ovoj sekciji naučit ćemo kako pametni ugovor sprema podatke. 

### 2.1 Spremanje

Naši pametni ugovori trebaju spremati podatke na blockchain. NEAR pruža različite opcije za spremanje, ovisno o načinu upotrebe i tipa podataka.

Koristit ćemo `Storage` klasu iz `near-sdk-as`, koji nudi sjajan API za spremanje i povlačenje podataka s blockchain-a. Možemo birati između sljedećih tipova kolekcije:  

- `PersistentMap` - ključ-vrijednost struktura podataka. 
- `PersistentVector` - array-like struktura podataka. 
- `PersistentDeque` - dvosmjerni queue.
- `PersistentUnorderedMap` - slično `PersistentMap` ali s korisnim dodatnim funkcijama koje nam omogućavaju da iteriramo po ključevima, vrijednostima, ulaznim parametrima.  


O spremanju I kolekcijama možete saznati više u [NEAR dokumentaciji](https://docs.near.org/docs/develop/contracts/as/intro#state-and-data).

### 2.2 Čitanje i pisanje stanja

Postoje dva tipa poziva funkcija, `view` i `change`. Pozivi koji su `view` će samo čitati podatke s blockchain-a, dok pozivi koji su `change` će pisati podatke na isti i mjenjati njegovo stanje. Pozivi koji su `view` su besplatni, dok treba plaćati gas za `change` pozive.

Na primjer, pozivanje `PersistentUnorderedMap#getSome(key: K)` I povlačenje podataka iza odgovarajući ključ neće imati trošak gasa. Dok s druge strane, pozivanje `PersistentUnorderedMap#setSome(ključ: K, vrijednost: V)` I dodavanje novog ključ-vrijednost para će imati trošak goriva. 

## 3. Read and Write ugovori

U ovoj sekciji tutorijala, pisat ćemo jednostavan pametni ugovor koji će spremati i povlačiti podatke s blockchain-a. 

Pisat ćemo ovaj ugovor u našoj `assembly/index.ts` datoteci.

Počinjemo s unošenjem `PersistentUnorderedMap` klase sa `near-sdk-as` na vrhu datoteke:

```javascript
import { PersistentUnorderedMap } from "near-sdk-as";
```

Nakon toga, kreiramo `PersistentUnorderedMap` instancu za spremanje naših proizvoda:

```javascript
export const products = new PersistentUnorderedMap<string, string>("PRODUCTS");
```

Kreiramo konstantnu varijablu zvanu `products` koja je `PersistentUnorderedMap` za mapiranje id-eva proizvoda tipa `string` s imenima proizvoda tipa `string`. 

String `PRODUCTS` iz `PersistentUnorderedMap` konstruktora je jedinstveni prefiks koji se koristi za svaki ključ.

### 3.1 Write funkcija

Napravit ćemo funkciju da dodamo novi proizvod `products` mapi:

```javascript
export function setProduct(id: string, productName: string): void {
    products.set(id, productName);
}
```

Funkcije koje moraju biti pozvane izvan pametnog ugovora moraju biti izvedene dodavanjem `export` keyword-a.

Našoj `setProduct` funkciji su neophodna dva parametra: `id` i `productName`.`id` parametar je ključ proizvoda, a `productName` parametar je vrijednost proizvoda.

Potrebno je navesti tip povrata funkcije, koji je u ovom slučaju `void` iz razloga što se ne vraca nista. 

Da bismo napravili novi unos za mapiranje proizvoda, treba samo pozvati `set` funkciju na instanci mape(`products`) I proslijediti ključ I vrijednost kao parametre. 

### 3.2 Read funkcija

Kako bismo završili naš prvi pametni ugovor, potrebno je kreirati funkciju koja će vratiti proizvod iz `products` mape.

```javascript
export function getProduct(id: string): string | null {
    return products.get(id);
}
```

Izvodimo `getProduct` funkciju i u ovom slučaju imamo samo jedan parametar `id`, koji je ključ proizvoda koji želimo vratiti. 

Povratni tip funkcije je `string | null`, a razlog je to što možemo vratiti ili naziv proizvoda ili `null` ako proizvod ne postoji. 

U tijelu funkcije pozivamo `get` funkciju na `products` mapiranju, pridajemo ključ kao parametar I vraćamo vrijednost. 

Konačni kod za ovaj odlomak izgleda ovako:

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

**Upozorenje**: Ključ trajne kolekcije treba biti što kraći kako bi se smanjila uporaba memorije. Na taj način taj ključ će biti ponovljen za svaki zapis u kolekciji. Ovdje smo koristili samo duži `PRODUCTS` ključ kako bismo povećali čitljivost za first-time NEAR developere.

## 4. Kreiranje računa

Kako bismo testirali naše pametne ugovore na NEAR testnet-u, potrebno je napraviti dva računa. 

Napravit ćemo jedan račun koji ćemo koristiti za interakciju s pametnim ugovorom, i drugi račun na koji ćemo razviti pametni ugovor. 
Drugi račun će biti podračun prvog računa, I izgledat će kao poddomena. 

Za ovaj modul koristit ćemo `myaccount.testnet` kao prvi račun, I napravit ćemo podračun istom pod nazivom `mycontract.myaccount.testnet`, na kojem ćemo razviti pametni ugovor. 

### 4.1 Kreiranje Top-Level računa

Otići na [NEAR Testnet wallet](https://wallet.testnet.near.org/) stranicu I napraviti novi račun prateći sljedeće korake:

1. Otvoriti wallet. 
2. Odabrati ime za račun.
3. Odabrati sigurnosnu metodu, odabrat ćemo lozinku za ovaj modul. 
4. Spremit cemo lozinku na sigurno mjesto.
5. Ponovno ćemo ukucati lozinku za potvrdu.

Ovo je GIF koji pokazuje prethodne korake: 
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/create_account.gif)

Sada je vaš testni račun kreiran I trebali biste ga moći koristiti. Vaš račun će već imati iznos NEAR testnet tokena, tako da nije nužno koristiti faucet.

Nakon toga, kreiramo podračun koristeći `near-cli`.

### 4.2 Login na NEAR CLI

Kako bismo se ulogirali na novi račun, otvaramo terminal i pokrećemo sljedeću `near-cli` naredbu:

```bash
near login
```

Ova naredba bi trebala otvoriti novi tab u browser-u I zatražiti login na naš NEAR račun. Zatražit će vas da se odobri dozvola `near-cli` za pristup našem računu.

Sada ste dokazali autentičnost za ovu sesiju.

Sada ste verificirani za ovu sesiju i možete obavljati transakcije kao što su deployment I interakcija s pametnim ugovorimaai preko `near-cli` alata.

Ovo je GIF koji prikazuje korake iznad:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/login_to_shell.gif)

### 4.3 Kreiranje podračuna

Da biste kreirali podračun za svoj nalog, pokrenite sljedeću naredbu: 

```bash
near create-account ${SUBACCOUNT_ID}.${ACCOUNT_ID} --masterAccount ${ACCOUNT_ID} --initialBalance ${INITIAL_BALANCE}
```

- `SUBACCOUNT_ID` - id podračuna.
- `ACCOUNT_ID` - id top-level računa.
- `INITIAL_BALANCE` - inicijalni iznos za podračun u NEAR tokenima. 

Kao što je prethodno najavljeno, želimo koristiti podračun za deploy pametnog ugovora. Dakle, u našem slučaju primjer poziva za kreiranje podnaloga bi izgledao ovako:

```bash
near create-account mycontract.myaccount.testnet --masterAccount myaccount.testnet --initialBalance 5
```

## 5. Sastaviti I postaviti

U ovoj sekciji, sastavit ćemo jednostavan pametni ugovor I postavit ćemo ga na NEAR testnet.

### 5.1 Sastaviti Contract

Prije nego što možemo postaviti pametni ugovor na NEAR testnet, potrebno je prevesti ga u wasm kod. Da bismo preveli naš ugovor trebamo pokrenuti sljedeću naredbu u korijenu projekta: 

```bash
yarn asb
```

Preveden wasm kod će biti spremljen u datoteci koja nosi naziv `${CONTRACT_NAME}.wasm` u sljedećem direktoriju:

```bash
${PROJECT_ROOT}/build/release/${CONTRACT_NAME}.wasm
```

`${CONTRACT_NAME}` se odnosi na ime vašeg projekta, kojeg možete naći u vašem package.json fajlu.

### 5.2 Deployanje ugovora

Da bismo deployali naš pametni ugovor na NEAR testnet, potrebno je pokrenuti sljedeću naredbu: 

```bash
near deploy --accountId=${ACCOUNT_ID} --wasmFile=${PATH_TO_WASM}
```

- `${ACCOUNT_ID}` - id naloga na koji će biti deployan pametni ugovor.
- `${PATH_TO_WASM}` - putanja do `.wasm` datoteka koja čini preveden pametni ugovor.

U našem slučaju, postavljena naredba bi izgledala ovako: 

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=build/release/near-marketplace-contract.wasm
```

Sada je naš ugovor deployan na NEAR testnet I možemo ga koristiti. 

## 6. Pozivi funkcija na pametnom ugovoru

U ovom dijelu, pozvat ćemo funkcije na deployanom pametnom ugovoru.

Kao što je najavljeno na početku modula, postoje dva tipa poziva funkcija koje možemo napraviti: `view` i `change`.

### 6.1 Pozivanje Change Funkcije

Prvo ćemo pozvati `change` funkciju.

`Change` poziv funkcije u `near-cli` izgleda ovako:

```bash
near call ${CONTRACT_ACCOUNT_ID} ${METHOD_NAME} ${PAYLOAD} --accountId=${ACCOUNT_ID}
```

- `${CONTRACT_ACCOUNT_ID}` - id računa koji sadrži pametni ugovor.
- `${METHOD_NAME}` - naziv funkcije koju želimo pozvati. 
- `${PAYLOAD}` - payload koji će biti poslan funkciji. 
- `${ACCOUNT_ID}` - id računa koji će napraviti poziv. 

Ako želimo dodati  novi proizvod na ugovor koji smo tek deployali, poziv bi izgledao ovako: 

```bash
near call mycontract.myaccount.testnet setProduct '{"id": "0", "productName": "tea"}' --accountId=myaccount.testnet
```

Ako koristite PowerShell ili CMD na Windows-u, možda ćete morati izbjeći dvostruke navodne znakove u payloadu, što bi izgledalo ovako: 

```bash
near call mycontract.myaccount.testnet setProduct "{\"id\": \"0\", \"productName\": \"tea\"}" --accountId=myaccount.testnet
```

Imajte to na umu za naredne cjeline kada ćemo imati dvostruke navodne znakove u payloadu.

Ako je vaš poziv funkcijel bio uspješan, vidjet ćete nešto slično narednom ispisu:

```
Scheduling a call: mycontract.myaccount.testnet.setProduct({"id": "0", "productName": "tea"})
Doing account.functionCall()
Transaction Id ${TRANSACTION_ID}
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.testnet.near.org/transactions/${TRANSACTION_ID}
''
```

### 6.2 Pozivanje View Funkcije

Sada kada smo dodali proizvod ugovoru, možemo pozvati `view` funkciju da dohvatimo proizvod.

`View` contract poziv u `near-cli` izgleda ovako:

```bash
near view ${CONTRACT_ACCOUNT_ID} ${METHOD_NAME} ${PAYLOAD}
```

S obzirom na to da ne trebamo platiti gas možemo izostaviti id računa na kraju.

Ako bismo htjeli dohvatiti proizvod koji smo tek dodali, poziv bi izgledao ovako: 

```bash
near view mycontract.myaccount.testnet getProduct '{"id": "0"}'
```

Ako je vaš contract poziv bio uspješan, vidjet ćete nešto nalik sljedećem outputu:

```
View call: mycontract.myaccount.testnet.getProduct({"id": "0"})
'tea'
```

Sada možete prevoditi I deployati svoje ugovore na NEAR testnetu i koristiti ih. 

## 7. Ugovor s modelom proizvoda

U ovoj cjelini, pisat ćemo drugu iteraciju našeg ugovora koja može pohraniti više od samog stringa.

### 7.1 Kreiranje modela proizvoda

Koristit ćemo model koji se zove `Product` za predstavljanje našeg proizvoda, zato što želimo biti u mogućnosti pohraniti više od samog naziva proizvoda. Model je prilagođen kontejneru podatka koji definira novi tip i sastoji se od AssemblyScript klase.

U assembly direktoriju kreirat ćemo novu datoteku pod nazivom `assembly/model.ts`.

Sadržaj datoteke bi trebao izgledati ovako: 

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

Koristimo `@nearBindgen` dekorator da serijalizira našu prilagođenu klasu prije nego što ga pohranimo na blockchain.

Naša `Product` klasa se sastoji od polja `id`, `name`, `description`, `image`, `location` i `owner` proizvoda, koji su sve stringovi. Imamo također `price` polje koje je 128 bitni unsigned integer, I jedno `sold` polje koje je 32 bit unsigned integer. Ovi tipovi numeričkih podataka su specifični za NEAR I preporučuje se da se koriste umjesto TypeScript numeričkih tipova kako bi se izbjegli problemi s konverzijom tipova podataka.

`u128` `price` polje nam omogućava da pohranimo NEAR cijenu u  [yocto](https://www.nanotech-now.com/metric-prefix-table.htm). 1 yocto-NEAR = 10<sup>-24</sup> NEAR, koja je najmanja jedinica NEAR.

Naša klasa se također sastoji od statične metode koja se zove `fromPayload` i koja uzima  payload I vraća novi `Product` objekt, kao I metode koja se zove `incrementSoldAmount`, koju ćemo koristiti kasnije da povećamo `sold` vrijednost nakon što je neki proizvod prodan 

`context` objekt sadrži dodatnu informaciju o transakciji. U ovom slučaju, koristimo `context.sender` da dohvatimo id računa koji poziva funkciju. 

Također kreiramo novu mapu pod nazivom `listedProducts` koja je perzistentna nesortirana mapa I koja će zamijeniti `products` mapu koju smo prethodno pohranili u našoj `index.ts` datoteci. Možemo također migrirati podatke s `products` mape na `listedProducts` mapu, ali to je već napredna tema koja je van okvira ovog modula. 

Kao što je prethodno objašnjeno, koristit ćemo verziju ključa koja je čitljivija za `LISTED_PRODUCTS` kako bismo omogućili lakše čitanje za naš kod. Da bi se smanjio prostor za pohranu, treba birati kraći ključ.

### 7.2 Nadogradnja `index.ts` datoteke

Potrebno je ažurirati našu `assembly/index.ts` datoteku da bismo obuhvatili novi model I mapu: 

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

Prvo, importamo `Product` klasu I `listedProducts` mapu, koja je nova struktura podataka koju smo dodali `model.ts` datoteci nakon što smo otklonili `products` mapu.

Nakon toga, modificiramo`setProduct` funkciju da koristi novu `Product` klasu i `listedProducts` mapu. Prvo provjeravamo postoji li id proizvoda već u mapi. Ako postoji, šaljemo grešku. Ako je drugačija situacija, `fromPayload` metode kreiramo novi `Product` objekt iz payloada I spremimo ga u `listedProducts` mapu.

Također modificiramo `getProduct` funkciju da bi je prilagodili da koristi novu `Product` klasu I `listedProducts` mapu.

Za kraj, kreiramo `getProducts` funkciju kako bismo vratili sve proizvode u mapu. 

### 7.3 Ažuriranje pametnog ugovora

NEAR nam dozvoljava ažuriranje koda ugovora na blockchain-u. Ovo možemo napraviti redeploymentom ugovora.

Prvo trebamo prevesti naš novi contract:

```bash
yarn asb
```

Nakon toga možemo deployati ugovor na isti id naloga kao prethodno:

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=build/release/${WASM_FILE_NAME}
```

Dodajmo novi proizvod ugovoru pozivanjem `setProduct` funkcije. Kako koristimo `Product` class, potrebno je priložiti `Product` objekt kao payload, koji bi trebao izgledati ovako: 

```bash
near call mycontract.myaccount.testnet setProduct '{"product": {"id": "0", "name": "BBQ", "description": "Grilled chicken and beef served with vegetables and chips.", "location": "Berlin, Germany", "price": "1000000000000000000000000", "image": "https://i.imgur.com/yPreV19.png"}}' --accountId=myaccount.testnet
```

Nakon uspješnog `setProduct` poziva, možemo pozvati `getProduct` funkciju da dohvati proizvod koji smo tek dodali: 

```bash
near view mycontract.myaccount.testnet getProduct '{"id": "0"}'
```

Trebali biste dobiti output sličan sljedećem: 

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

To je to! Uspešno smo dodali novi proizvod našem ugovoru.

## 8. Ugovor sa Buy Funkcijom

Za finalnu cjelinu ovog tutorijala, kreirat ćemo `buyProduct` funkciju da omogućimo korisniku kupnju proizvoda. 

`near-sdk-as` biblioteka omogućava `ContractPromiseBatch` klasu koja nam dozvoljava da skupne akcije unutar AssemblyScript ugovora. Koristit ćemo ovaj za obavljanje transfera tokena od pozivatelja funkcije do vlasnika proizvoda. 

Kod za transfer tokena izgleda ovako:

```javascript
ContractPromiseBatch.create(${RECEIVING_ACCOUNT}).transfer(${DEPOSIT});
```

Prethodno smo spomenuli da `context` objekt sadrži informacije o transakciji. Za našu `buyProduct` funkciju koristit ćemo `context.attchedDeposit` da dobijemo količinu tokena koju je pozivatelj funkcije priložio na transakciju. 

Napišimo našu `buyProduct` funkciju u `index.ts` datoteku.

Prvo, importamo`ContractPromiseBatch` i `context` iz `near-sdk-as` biblioteke na samom vrhu datoteke:

```javascript
import { Product, listedProducts } from './model';
import { ContractPromiseBatch, context } from 'near-sdk-as';
```

Sada možemo pisati `buyProduct` funkciju na dnu datoteke:

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

Prvo dohvaćamo proizvod sa specificiranim id-em kroz `getProduct` funkciju.

Nakon toga provjeravamo postoji li proizvod. Ako ne postoji, šaljemo grešku ("product not found"). Ako je situacija drugačija, provjeravamo je li priložen depozit jednak cijeni proizvoda. Ako nije, šaljemo grešku ("attached deposit should equal to the product's price").

Nakon toga kreiramo novi `ContractPromiseBatch` objekt I pozivamo `transfer` metodu na njega. Ova metoda uzima količinu tokena koju je pozivatelj funkcije priložio na transakciju I vrši transfer istih do vlasnika proizvoda. Dobijemo račun vlasnika proizvoda tako što pristupamo `owner` propertiju proizvoda koji smo dohvatili. 

Konačno, povećavamo `sold` polje proizvoda pozivanjem `incrementSoldAmount` funkcije i ažuriramo proizvod u `listedProducts` mapi.

To je to! Finalni ugovor bi trebao izgledati ovako:

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

Sada treba prevesti ugovor posljednji puta: 

```bash
yarn asb
```

Onda možemo redeployati ugovor na isti id nalog kao prethodno: 

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=build/release/${WASM_FILE_NAME}
```

Ajmo testirati naš ugovor pozivanjem `buyProduct` funkcije. Da bismo to učinili, kreiramo novi podračun koji će služiti kao kupac I vršit će transfer tokena na njega:  

```bash
near create-account buyeraccount.myaccount.testnet --masterAccount myaccount.testnet --initialBalance 6
```

Sada smo spremni kupiti proizvod s računom. Ovako bi kod za kupovinu proizvoda trebao izgledati: 

```bash
near call mycontract.myaccount.testnet buyProduct '{"productId": "0"}' --depositYocto=1000000000000000000000000 --accountId=buyeraccount.myaccount.testnet
```

Novo u ovom pozivu je `--depositYocto` parametar. Ovaj parameter specificira količinu tokena koje kupac stavlja na transakciju. U ovom slučaju, mi stavljamo 1 NEAR token u Yocto-NEAR. Izvršavamo ovaj poziv s računa kupca naravno. 

Ako nemamo nikakve greške, trebali bismo vidjeti sljedeći output:

```
Scheduling a call: mycontract.myaccount.testnet.buyProduct({"productId": "0"}) with attached 1 NEAR
Doing account.functionCall()
Transaction Id ${TRANSACTION_ID}
To see the transaction in the transaction explorer, please open this URL in your browser
https://explorer.testnet.near.org/transactions/${TRANSACTION_ID}
''
```

Pogledajmo je li transakcija prošla kako treba. Kopirajte link na block explorer testnet-a I otvorite ga u svom browser-u. Trebali biste vidjeti transakciju u transaction explorer-u, provjeriti je li transakcija prošla kako treba i da je količina token transfera 1 NEAR.

Nakon toga, provjerit ćemo je li proizvod kupljen kako treba. Koristit ćemo `getProduct` funkciju da dohvatimo proizvod I provjerimo je li `sold` polje jednako 1.

```bash
near view mycontract.myaccount.testnet getProduct '{"id": "0"}'
```

Output bi trebao izgledati ovako: 

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

To je to! Uspješno smo napisali ugovor za decentralizirano tržište.. 

Možete naći kod za ovaj projekt na [GitHub](https://github.com/dacadeorg/near-marketplace-dapp/tree/master/smartcontract).

Nakon toga, možete pogledati naše module koji objašnjavaju kako napraviti frontend za tržište.