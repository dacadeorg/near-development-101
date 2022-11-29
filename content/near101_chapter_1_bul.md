Ще използваме AssemblyScript, за да напишем нашите NEAR смарт договори. Писането на AssemblyScript код е много подобно на писането на TypeScript код. Но има няколко неща, специфични за писането на смарт договори NEAR в AssemblyScript, които ще разгледаме в следващите раздели.

### Предпоставки

- Имате основни познания в Blockchain технологиите и смарт договорите.
- Имате основни познания по JavaScript.
- Можете да работите с терминал.

### Технически стек

Ще използваме следните инструменти:

- [near-cli](https://www.npmjs.com/package/near-cli) - CLI инструмент за NEAR, който предлага API за взаимодействие със смарт договори.
- [assemblyscript](https://www.npmjs.com/package/assemblyscript) - Език за WebAssembly, подобен на TypeScript.
- [asbuild](https://www.npmjs.com/package/asbuild) - Build tool за AssemblyScript.


## 1. Подготовка

В този първи раздел на урока ще подготвим нашата среда за разработка и проект.

Ще трябва да имаме инсталирана версия на node.js, по-висока или равна на 12.
Ще използваме и yarn-а за управление на пакети, така че се уверете, че е инсталиран.

### 1.1 Инсталирайте CLI инструменти

Можете да инсталирате най-новите версии на CLI инструменти, като изпълните следните команди:
```bash
yarn global add near-cli assemblyscript asbuild
```

Препоръчваме да използвате редактор, който поддържа code completion и syntax highlighting за TypeScript, за да ви помогне да пишете код на AssemblyScript, като Visual Studio Code или Atom(EOL). За този учебен модул ще използваме Visual Studio Code.

### 1.2 Подготовка на проекта

Сега ще подготвим нашия проект. Създайте нова главна директория за нашия проект и я кръстете нещо от сорта на  `near-marketplace-contract`.

#### 1.2.1 `asconfig.json`

В главната ни директория трябва да създадем файл, наречен `asconfig.json`. Този конфигурационен файл предоставя CLI опции и конфигурации за AssemblyScript.

За да компилираме с `near-sdk-as`, ще трябва да добавим следното към файла `asconfig.json`:
```javascript
{
    "extends": "near-sdk-as/asconfig.json"
}

#### 1.2.2 `tsconfig.json`

Създаваме директория „assembly“ и файла „assembly/tsconfig.json“ в нея. Този файл описва настройките на компилатора и задава основни файлове, които са необходими за компилиране на TypeScript проекти.

Файлът `tsconfig.json` трябва да бъде в основната директория на проекта TypeScript.
```javascript
{
  "extends": "../node_modules/assemblyscript/std/assembly.json",
  "include": [
    "./**/*.ts"
  ]
}

#### 1.2.3 `as_types.d.ts`

Създаваме файла `assembly/as_types.d.ts` в нашата `assembly` директория. Този файл се използва за дефиниране на типове, които се използват в нашия код на AssemblyScript.
В този случай ние използваме типовете от `near-sdk-as`.

```javascript
/// <reference types="near-sdk-as/assembly/as_types" />
```

### 1.3 Инициализирайте проекта

Изпълнете следната команда, за да инициализирате проекта в главната му директория.:

```bash
yarn init
```


Това ще създаде файл - package.json, където са изброени зависимостите.

Изпълнете следната команда, за да добавите `near-sdk-as` към проекта:

```bash
yarn add -D near-sdk-as
```

Потребителите на Mac M1 може да имат грешката `Error: Unsupported platform: Darwin arm64 при опит за инсталиране.
За да решите проблема изпълнете предишната команда с опцията `--ignore-scripts`:

yarn add -D --ignore-scripts near-sdk-as
```



В последната стъпка от процеса на инициализация ще трябва да създадем основен файл за смарт договора. Създайте файл, наречен `index.ts` в директорията `assembly`.

Крайната структура на проекта трябва да изглежда така:

```
```
├── asconfig.json
├── assembly
│   ├── as_types.d.ts
│   ├── index.ts
│   └── tsconfig.json
├── package.json
└── yarn.lock
```

## 2. Договорно съхранение

В този раздел ще научим как да съхраняваме данни в смарт договора.

### 2.1 Съхранение

Нашите смарт договори ще трябва да съхраняват данни на блокчейна. NEAR предлага различни опции за съхранение в зависимост от начина за употреба и типа на данните.

Ще използваме класа `Storage` от `near-sdk-as`, който предлага страхотен интерфейс за съхраняване и извличане на данни от блокчейна. Можем да избираме между следните видове колекции:

- `PersistentMap` - структура за ключ-стойност данни.
- `PersistentVector` - подобна на масив структура от данни.
- `PersistentDeque` - двупосочна опашка.
- `PersistentUnorderedMap` - подобно на `PersistentMap`, но с полезни допълнителни функции, които ни позволяват да обхождаме ключове, стойности, записи.

Можете да научите повече за съхранението и колекциите в [документацията NEAR](https://docs.near.org/docs/develop/contracts/as/intro#state-and-data).

### 2.2 Четене и запис

Има два типа извикващи функции, `view` и `change`. Извикванията, `view`, ще четат само данни от блокчейна, докато извиванията `change`, ще записват данни на блокчейна и ще променят състоянието му. Извикванията  „view“, са безплатни, докато ние трябва да платим gas fee за извикванията  „change“.

Например извикването `PersistentUnorderedMap#getSome(key: K)` и извличането на данни за съответния ключ няма да струва gas. Докато извикването `PersistentUnorderedMap#setSome(key: K, value: V)` и добавянето на нова двойка ключ-стойност ще струва gas.

## 3. Прочетете и напишете приложение

В този раздел на урока ще напишем прост смарт договор, който ще съхранява и извлича данни от блокчейна.

Ще напишем това приложение в нашия файл `assembly/index.ts`.

Нека започнем с импортиране на класа `PersistentUnorderedMap` от `near-sdk-as` в горната част на файла:

```javascript
import { PersistentUnorderedMap } from "near-sdk-as";
```

След това създаваме променлива от тип „Persistent Unordered_Map“, за да съхраняваме нашите продукти. Създаваме постоянна променлива, с име  „products“, която е „PersistentUnorderedMap“ която ще съпостави ids на продукти от тип „string“ към имена на продукти от тип „string“.

String-ът  „PRODUCTS“ в конструктора на „PersistentUnorderedMap“ е уникалният префикс, който да се използва за всеки ключ.

```javascript
export const products = new PersistentUnorderedMap<string, string>("PRODUCTS");
```



### 3.1 Функция за запис

Нека създадем функция за добавяне на нов продукт към колекцията  „products“:

```javascript
export function setProduct(id: string, productName: string): void {
    products.set(id, productName);
}
``


Функциите, които трябва да бъдат извикани извън смарт договора, трябва да бъдат експортирани чрез добавяне на ключовата дума „export“.

Нашата функция `setProduct` се нуждае от два параметъра: `id` и `productName`. Параметърът `id` е ключът на продукта, а параметърът `productName` е стойността на продукта.

Трябва да посочим типа на връщане на функцията, който е `void`, тъй като не е необходимо да връщаме нищо.

За да създадем нов запис в колекцията-а на нашите продукти, трябва просто да извикаме функцията `set` на променливата (`products`) и да дадем ключа и стойността като параметри.

### 3.2 Функция за четене

За да завършим нашия първи смарт договор, трябва да създадем функция за извличане на продукт от колекцията  „продукти“.

```javascript
export function getProduct(id: string): string | null {
    return products.get(id);
}
```


Експортираме функцията `getProduct` със само един параметър `id`, който е ключът на продукта, който искаме да извлечем.

Стойността на връщане на функцията е `string | null`, тъй като можем да върнем или име на продукт, или `null`, ако продуктът не съществува.

В тялото (body-то) на функцията извикваме функцията `get` на колекцията  `products`, подаваме ключа като параметър и връщаме стойността.

Крайният код за този раздел изглежда така:

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

**Предупреждение**: Ключът за постоянната колекция трябва да бъде възможно най-кратък, за да се намали мястото  за съхранение, тъй като този ключ ще се повтаря за всеки запис в колекцията. Тук използвахме само по-дългия ключ `PRODUCTS`, за да добавим по-голяма яснота на новите NEAR разработчици.

## 4. Създайте акаунти

За да тестваме нашите смарт договори в тестовата мрежа NEAR, трябва да създадем два акаунта.

Ще създадем един акаунт, който ще използваме за взаимодействие със смарт договора, и друг акаунт, към който ще закачим смарт договора.
Вторият акаунт ще бъде подакаунт на първия акаунт и ще изглежда като поддомейн.

За този учебен модул ще използваме `myaccount.testnet` като първи акаунт и ще създадем негов подакаунт, наречен `mycontract.myaccount.testnet`, където ще внедрим (deploy-нем) нашият смарт договор.

### 4.1 Създайте акаунт от първо ниво

Отидете на страницата [NEAR Testnet wallet](https://wallet.testnet.near.org/) и създайте нов акаунт, като следвате тези стъпки:

1. Отворете портфейла.
2. Изберете име за вашия акаунт.
3. Изберете метод за сигурност, ние ще изберем парола за този учебен модул.
4. Съхранете паролата на сигурно място.
5. Въведете отново паролата, за да потвърдите.

Ето GIF, показващ горните стъпки:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/create_account.gif)

Сега вашият тестов акаунт е създаден и трябва да можете да го използвате. Вашият акаунт вече ще има баланс от NEAR testnet токени, така че не е необходимо да използвате faucet.

След това ще създадем подакаунт с помощта на `near-cli`.

### 4.2 Влезте в NEAR CLI

За да влезете в новия си акаунт, отворете терминал и изпълнете следната команда `near-cli`:

```bash
near login
```

Тази команда трябва да отвори нов подпрозорец във вашия браузър и да ви помоли да влезете във вашия NEAR акаунт. Ще бъдете помолени да предоставите разрешения на `near-cli` за достъп до вашия акаунт.

Вече сте удостоверени за тази сесия и можете да извършвате транзакции, като внедряване (deployment) и извиквания на методите на договора чрез „near-cli“.

Ето GIF, показващ горните стъпки:
![](https://github.com/dacadeorg/near-development-101/raw/master/content/gifs/login_to_shell.gif)

### 4.3 Създайте подакаунт

За да създадете подакаунт за вашия акаунт, изпълнете следната команда:

```bash
near create-account ${SUBACCOUNT_ID}.${ACCOUNT_ID} --masterAccount ${ACCOUNT_ID} --initialBalance ${INITIAL_BALANCE}
```
- `SUBACCOUNT_ID` - id на подакаунта.
- `ACCOUNT_ID` - id на акаунта от първо ниво.
- `INITIAL_BALANCE` - начален баланс за подакаунта в NEAR токени.

Както беше посочено по-рано, искаме да използваме подакаунта, за да внедрим (deploy-нем) нашия смарт договор. Така че в нашия случай едно примерно извикване за създаване на подакаунт може да изглежда така:
```bash
near create-account mycontract.myaccount.testnet --masterAccount myaccount.testnet --initialBalance 5
```
## 5. Компилирайте и качете

В този раздел ще съставим нашия прост смарт договор и ще го внедрим в тестовата мрежа NEAR.

### 5.1 Съставяне на договора

Преди да можем да внедрим нашия смарт договор в тестовата мрежа NEAR, трябва да го компилираме до wasm код. За да съставим нашия договор, трябва да изпълним следната команда в главната директория на проекта:

```bash
yarn asb
```


Компилираният wasm код ще се съхранява във файл, наречен `${CONTRACT_NAME}.wasm` в следната директория:

```bash
${PROJECT_ROOT}/build/release/${CONTRACT_NAME}.wasm
```

`${CONTRACT_NAME}` се отнася до името на вашия проект, което можете да намерите във вашия файл package.json.

### 5.2 Внедряване на приложението

За да внедрим нашия смарт договор в тестовата мрежа NEAR, трябва да изпълним следната команда:

```bash
near deploy --accountId=${ACCOUNT_ID} --wasmFile=${PATH_TO_WASM}
```

- `${ACCOUNT_ID}` - идентификационният номер на акаунта, към който ще се внедри смарт договор.
- `${PATH_TO_WASM}` - пътят до `.wasm` файла, който съдържа компилирания смарт договор.

В нашия случай командата за внедряване може да изглежда така:

```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=build/release/near-marketplace-contract.wasm
```

Сега нашият договор е внедрен в тестовата мрежа NEAR и можем да го използваме.

## 6. Извиквания към приложението

В този раздел на учебния модул ще извикаме функциите на внедрения смарт договор.

Както беше посочено в началото на учебния модул, има два типа извиквания на функции, които можем да направим: `view` и `change`.

### 6.1 Извикване на change функция

Първо, ще извикаме change функция.

Извикването на договора `change` в `near-cli` изглежда така:
```bash
near call ${CONTRACT_ACCOUNT_ID} ${METHOD_NAME} ${PAYLOAD} --accountId=${ACCOUNT_ID}
```

- `${CONTRACT_ACCOUNT_ID}` - идентификаторът на акаунта, който съдържа смарт договора.
- `${METHOD_NAME}` - името на функцията, която искаме да извикаме.
- `${PAYLOAD}` -  аргументите, нужни за изпълнението на функцията.
- `${ACCOUNT_ID}` - идентификаторът на акаунта, който ще направи извикването.

Ако искате да добавите нов продукт към договора, който току-що внедрихме, извикването може да изглежда така:
```bash
near call mycontract.myaccount.testnet setProduct '{"id": "0", "productName": "tea"}' --accountId=myaccount.testnet
```
Ако използвате PowerShell или CMD в Windows, може да ви се наложи да избегнете двойните кавички в json-a, което може да изглежда така:
```bash
near call mycontract.myaccount.testnet setProduct "{\"id\": \"0\", \"productName\": \"tea\"}" --accountId=myaccount.testnet

Имайте това предвид за следващите раздели, когато имаме двойни кавички в аргументите, които подавате.

Ако извикването на вашия договор е било успешно, ще видите нещо подобно на следния изход:

```
Scheduling a call: mycontract.myaccount.testnet.setProduct({"id": "0", "productName": "tea"})
Doing account.functionCall()
Transaction Id ${TRANSACTION_ID}
To see the transaction in the transaction explorer, please open this url in your browser
https://explorer.testnet.near.org/transactions/${TRANSACTION_ID}
''

### 6.2 Извикване на функция за четене

Сега, след като добавихме продукт към договора, можем да извикаме функцията `view`, за да извлечем продукта.

Извикването на договора `view` в `near-cli` изглежда така:
```bash
near view ${CONTRACT_ACCOUNT_ID} ${METHOD_NAME} ${PAYLOAD}
```
Тъй като не е необходимо да плащаме gas, можем да пропуснем идентификатора на акаунта в края.

Ако искаме да извлечем продукта, който току-що добавихме, извикването може да изглежда така:
```bash
near view mycontract.myaccount.testnet getProduct '{"id": "0"}'
```
Ако изпълнението на функцията на вашия договор е било успешно, ще видите нещо подобно на следния изход:
```
View call: mycontract.myaccount.testnet.getProduct({"id": "0"})
'tea'
```

Сега можете да компилирате и внедрите договора си в тестовата мрежа NEAR и да взаимодействате с него.

## 7. Договор смодел на продукт

В този раздел ще напишем втора версия на нашия договор, която може да съхранява повече от просто string.

### 7.1 Създайте модел на продукти

Ще използваме модел, наречен „Product“, за да представим нашите продукти, защото искаме да можем да съхраняваме повече от името на продукта. Моделът е дефинирана от програмиста структура от данни.

В директорията assembly ще създадем нов файл, наречен `model.ts`.

Съдържанието на файла трябва да изглежда така:
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

Използваме атрибута ‘@nearBindgen за да можем да съхраним класа и данните му на блокчейна.’

Нашият клас `Product` се състои от полетата `id`, `name`, `description`, `image`, `location` и `owner` на продукта, които са низ от символи. Имаме и поле „price“, което е 128-битово цяло положително число и поле „sold“, което е 32-битово цяло положително число. Тези цифрови типове данни са специфични за NEAR и се препоръчва да се използват вместо числовите типове TypeScript, за да се избегнат проблеми с преобразуването на типове данни.

Полето `u128` `price` ни позволява да съхраняваме NEAR цената в [yocto](https://www.nanotech-now.com/metric-prefix-table.htm). 1 yokto-NEAR = 10<sup>-24</sup> NEAR, което е най-малката единица на NEAR.

Нашият клас също така се състои от статичен метод, наречен `fromPayload`, който приема параметър от същия тип и връща нов обект `Product`, и метод, наречен `incrementSoldAmount`, който ще използваме по-късно, за да увеличим стойността на `sold`, след като продуктът е бил е продаден.

Обектът `context` съдържа допълнителна информация за транзакцията. В този случай ние използваме `context.sender`, за да извлечем идентификатора на акаунта, който извиква функцията.

Също така създаваме нова колекция от тип ключ-стойност, наречена „listedProducts“, която ще замени променливата  “products“, която преди това сме използвали в нашия файл „index.ts“. Можем също така да мигрираме данните от “ към „listedProducts“, но това е тема за напреднали, която е извън обхвата на този учебен модул.

Както беше обяснено по-рано, ще използваме по-четима версия на ключ за `LISTED_PRODUCTS`, за да осигурим по-голяма четливост на нашия код. За да намалите мястото за съхранение, трябва да изберете по-къс ключ.

### 7.2 Актуализирайте `index.ts`

Трябва да актуализираме нашия файл `assembly/index.ts` до p:
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

Първо импортираме класа „Product“ и колекцията  „listedProducts“, която е нова структура от данни, която добавихме към файла „model.ts“, след като премахнахме променливата „products“.

След това модифицираме функцията `setProduct`, за да я коригираме да използва новия клас `Product` и`listedProducts`. Първо проверяваме дали идентификаторът на продукта вече съществува в картата. Ако го направи, извеждаме грешка. В противен случай извикваме метода `fromPayload`, за да създадем нов обект `Product` от данните и да го съхраним в колекцията `listedProducts`.

Също така променяме функцията `getProduct`,да използва новия клас `Product` и променливата `listedProducts`.

Накрая създаваме функцията `getProducts` за връщане на всички продукти в колекцията.

### 7.3 Заменяне на договора с нова версия

NEAR ни позволява да актуализираме кода на нашия договор на блокчейна. Можем да направим това, като качим договора отново.

Първо трябва да съставим нашия нов договор:

```bash
yarn asb
```
След това можем да актуализираме договора към същия идентификатор на акаунта, както преди:
```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=build/release/${WASM_FILE_NAME}
```
Нека добавим нов продукт към договора, като извикаме функцията `setProduct`. Тъй като използваме класа „Product“, трябва да подадем данни, от тип „Product“, като изглежда така:
```bash
near call mycontract.myaccount.testnet setProduct '{"product": {"id": "0", "name": "BBQ", "description": "Grilled chicken and beef served with vegetables and chips.", "location": "Berlin, Germany", "price": "1000000000000000000000000", "image": "https://i.imgur.com/yPreV19.png"}}' --accountId=myaccount.testnet
```

След успешно извикване на `setProduct`, можем да извикаме функцията `getProduct`, за да извлечем продукта, който току-що добавихме:

```bash
near view mycontract.myaccount.testnet getProduct '{"id": "0"}'
```

Трябва да получите изход, подобен на следния:
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

Това е! Успешно добавихме нов продукт към нашия договор.

## 8. Договор с функция за покупка

За последния раздел на този урок ще създадем функцията `buyProduct`, за да позволим на потребителя да закупи продукт.

Библиотеката `near-sdk-as` предоставя клас `ContractPromiseBatch`, който ни позволява да групираме действия в рамките на договор на AssemblyScript. Ще използваме този клас за прехвърляне на токени от извикващия функцията към собственика на продукта.

Кодът за прехвърляне на токени изглежда така:
```javascript
ContractPromiseBatch.create(${RECEIVING_ACCOUNT}).transfer(${DEPOSIT});
```

По-рано споменахме, че обектът „context“ съдържа информация за транзакцията. За нашата функция `buyProduct` ще използваме `context.attchedDeposit`, за да получим количеството токени, които извикващият функцията е прикачил към транзакцията.

Нека напишем нашата функция `buyProduct` в нашия файл `index.ts`.

Първо импортираме `ContractPromiseBatch` и `context` от библиотеката `near-sdk-as` в горната част на файла:
```javascript
import { Product, listedProducts } from './model';
import { ContractPromiseBatch, context } from 'near-sdk-as';
```
Сега можем да напишем нашата функция `buyProduct` в долната част на файла:
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
Първо извличаме продукта с посочения идентификатор чрез функцията `getProduct`.

След това проверяваме дали продуктът съществува. Ако не стане, извеждаме грешка ("product not found"). В противен случай проверяваме дали изпратените токени са колкото е цената на продукта. Ако не е, извеждаме грешка ("attached deposit should equal to the product's price").

След това създаваме нов обект `ContractPromiseBatch` и извикваме метода `transfer` върху него. Този метод взема количеството токени, които извикващият функцията е добавил към транзакцията, и ги прехвърля на собственика на продукта. Получаваме акаунта на собственика на продукта чрез достъп до член-променливата `owner`на продукта, който сме извлекли.

И накрая, увеличаваме полето „sold“ на продукта, като извикваме функцията „incrementSoldAmount“ и актуализираме продукта в колекцията „listedProducts“.

Това е! Финалният договор трябва да изглежда така:
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
Сега трябва да съставим нашия договор за последен път:
```bash
yarn asb
```

След това можем да превнедрим договора към същия идентификатор на акаунта, както преди:
```bash
near deploy --accountId=mycontract.myaccount.testnet --wasmFile=build/release/${WASM_FILE_NAME}
```
Нека тестваме нашия договор, като извикаме функцията `buyProduct`. За да направим това, ще създадем нов подакаунт, който ще действа като купувач и ще прехвърли някои токени към него:

```bash
near create-account buyeraccount.myaccount.testnet --masterAccount myaccount.testnet --initialBalance 6
```

Вече сме готови да закупим продукт с акаунта. Ето как изглежда кодът за закупуване на продукт:
```bash
near call mycontract.myaccount.testnet buyProduct '{"productId": "0"}' --depositYocto=1000000000000000000000000 --accountId=buyeraccount.myaccount.testnet
```

Новото в това извикване е параметърът `--depositYocto`. Този параметър определя количеството токени, които купувачът ще прикачи към транзакцията. В този случай ние прикачваме 1 NEAR токен в Yocto-NEAR. Ние изпълняваме това извикване от акаунта на купувача, разбира се.

Ако нямаме никакви грешки, трябва да видим следния резултат:
```
Scheduling a call: mycontract.myaccount.testnet.buyProduct({"productId": "0"}) with attached 1 NEAR
Doing account.functionCall()
Transaction Id ${TRANSACTION_ID}
To see the transaction in the transaction explorer, please open this URL in your browser
https://explorer.testnet.near.org/transactions/${TRANSACTION_ID}
''
```
Да видим дали транзакцията е преминала правилно. Копирайте връзката към block explorer-a на тестовата мрежа и я отворете във вашия браузър. Трябва да видите транзакцията в explorer-a на транзакциите, да проверите дали транзакцията е преминала правилно и сумата на трансфера на токена е 1 NEAR.

След това ще проверим дали продуктът е закупен правилно. Ще използваме функцията `getProduct`, за да извлечем продукта и да проверим дали полето `sold` е равно на 1.
```bash
near view mycontract.myaccount.testnet getProduct '{"id": "0"}'
```
Резултатът сега трябва да изглежда така:
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
Това е! Успешно написахме договор за децентрализиран пазар.

Можете да намерите кода за този проект в [GitHub](https://github.com/dacadeorg/near-marketplace-dapp/tree/master/smartcontract).

След това можете да разгледате нашите учебни модули, които обясняват как да изградите интерфейса за пазара.