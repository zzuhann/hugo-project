---
title: "Effective TypeScript CH6 - 撰寫 TypeScript 型態宣告時的注意事項"
description: "這篇文章探討了在撰寫 TypeScript 型態宣告時需要注意的事項，包括優先使用 structural typing、將所有用到的型態都 export、使用 JSDoc 或 TSDoc 格式的註釋來說明函式、類別與型態，以及優先考慮使用條件型態而不是多載的型態宣告。透過這些技巧，可以提高程式碼的可讀性和易用性。"
date: 2024-04-06T20:31:14+08:00
draft: false
author: "zzuhann"
slug: "type-declaration-tips"
categories: ["前端"]
keywords:
  - "TypeScript"
  - "type declaration"
  - "型態宣告"
  - "TSDoc"
  - "structural typing"
---

### 前情提要

延續上一篇{{< a href="https://www.zzuhann.work/posts/typescript-types/" target="_blank" >}}Effective TypeScript CH6 - 使用 TypeScript 後要注意的版本管理、介紹 @types{{< /a >}}，這篇會提到的內容當自己嘗試要開始寫型態宣告、並且開放給其他人使用時，要注意哪些事情。

### 這篇文會提到

- [在寫型態宣告，並且預計開放給其他人使用，要注意什麼？](#在寫型態宣告並且預計開放給其他人使用要注意什麼)
  - [優先使用 structural typing 來提供非必要的依賴項目](#優先使用-structural-typing-來提供非必要的依賴項目)
  - [設計了 public 的 api，只要有用到的型別就把它 export 出去](#設計了-public-的-api只要有用到的型別就把它-export-出去)
  - [使用 JSDoc 或 TSDoc 格式的註釋來說明匯出的函式、類別與型態](#使用-jsdoc-或-tsdoc-格式的註釋來說明匯出的函式類別與型態)
  - [優先使用條件型態，而不是多載的型態宣告](#優先使用條件型態而不是多載的型態宣告)

### 在寫型態宣告，並且預計開放給其他人使用，要注意什麼？

#### 優先使用 structural typing 來提供非必要的依賴項目

當開始寫型態宣告時，可能會有需要依賴其他 `@types` 的型別，為了讓其他使用者 install 時也能成功下載依賴的型別，所以將依賴的 `@types` 安裝在 devDependency。

書上的舉例是：

```ts
/**
 * 寫了一個解析 CSV 檔案的程式庫
 * input: CSV 檔案內容（為了方便 NodeJS 用戶/瀏覽器用戶，可以是 Buffer 或字串）
 * output: 物件串列：將欄位名稱作為 key，欄位值作為 value
 */
function parseCSV(input: string | Buffer): { [column: string]: string }[] {
  if (typeof input === "object") {
    return parseCSV(input.toString("utf-8"));
  }
  // ...
}
```

而 `Buffer` 是來自 nodejs 的型態宣告，所以 install 了 `@types/node`

```
npm install --save-dev @types/node
```

這時候會產生的問題是：對瀏覽器開發者來說，它們並不需要這個型別、而對非 TypeScript 的 nodejs 開發者來說，他們也不需要。
作者建議可以優先使用 structural typing，以減少對外部依賴的需要，只要符合所需功能的型別即可。

```ts
// 透過 structural typing，判斷是否存在特定的 method 來決定是否為 Buffer
interface CSVBuffer {
  toString(encoding: string): string;
}
function parseCSV2(input: string | CSVBuffer): { [column: string]: string }[] {
  if (typeof input === "object") {
    return parseCSV2(input.toString("utf-8"));
  }
  // ...
}
// 在 nodejs 中就可以透過以下方式使用
parseCSV2(new Buffer("a,b,c\n1,2,3\n"));
```

能不依賴其他套件的型別就盡量不依賴，先嘗試看看透過 structural typing 是否能解決。
作者下了這樣的總結：不要強迫 JavaScript 開發者依賴 `@types`、不要強迫 Web 開發者依賴 nodejs。

#### 設計了 public 的 api，只要有用到的型別就把它 export 出去

當設計了 public 的 api，應該把有用到的型別都 export，確保使用者可以存取到所有用到的型別，不需要東拼西湊，只為了找出類似的型別或是重新進行型態宣告。

```ts
interface Name {
  firstName: string;
  lastName: string;
}

export function getDeposit(name: Name) {
  // ...
}
```

在上面的例子來說，雖然使用者 import 的對象是 `getDeposit` 這個 function，看起來不需要 import `Name` 這個型別，但其實為了正確執行 `getDeposit` 這個 function，他仍需要 `Name` 來驗證參數的型別。

所以，只要有用到的型別就 export 出去，讓使用者更方便、也更彈性的使用你設計的 api。

#### 使用 JSDoc 或 TSDoc 格式的註釋來說明匯出的函式、類別與型態

你經常寫的是行內註釋 `//` 還是 JSDoc/TSDoc 風格的註釋 `/** */` 呢？
編輯器會透過提示工具把 JSDoc/TSDoc 的註釋顯示出來，行內註釋則不會。

![jsdoc](/gif/jsdoc.gif)
![jsdoc-import](/gif/jsdoc-import.gif)
使用 JSDoc 時，import 也能看到 JSDoc 對該 function 的註釋。

在定義型態時使用 TSDoc
![tsdoc](/gif/new-tsdoc.gif)

#### 優先使用條件型態，而不是多載的型態宣告

雖然 TypeScript 支援多載的方式宣告 function 的型別，作者建議應該優先使用條件型態，而不是多載的型態宣告。
條件型態可讓你的宣告式支援聯集型態。

以下例子為 Effective TypeScript 書中程式碼

```ts
/**
 * 需要實作一個可以接收 string | number 的 function
 * 剛開始，可能會使用聯集類型
 */
function double(x: number | string): number | string;
function double(x: any) {
  return x + x;
}

// 這樣的寫法，會讓 return 的 type 都變成 string | number
const num = double(12); // 但我想要這個的 return type 是 number
const str = double("x"); // 我想要這個的 return type 是 string

/**
 * 接下來，也許可以嘗試看看泛型
 */
function double2<T extends number | string>(x: T): T;
function double2(x: any) {
  return x + x;
}

// 傳入字面常值，因為是 const，所以傳給 T 的型別是 12 | 'x'
// 接著 return 的型別就是 12 | 'x'
// 但實際上，得到的結果應該是 24 | 'xx'
const num2 = double2(12); // return type 12, 但實際應該是 24
const str2 = double2("x"); // return type 'x', 但實際應該是 'xx'

/**
 * 試試看多載
 */
function double3(x: number): number;
function double3(x: string): string;
function double3(x: any) {
  return x + x;
}

// 看起來還不錯？
const num3 = double3(12); // number
const str3 = double3("x"); // string

/**
 * 此時如果有一個 function 嘗試傳入 string | number
 */
function f(x: number | string) {
  return double3(x);
}
// 進行多載型態宣告時，TypeScript 會一個一個檢查，直到找到相符的為止
// 先檢查是否符合 function double3(x: number): number;
// 再檢查是否符合 function double3(x: string): string;
// 都不符合，所以會在最後一個 string type 報錯

/**
 * 使用條件型態
 * 看起來像剛剛提到的泛型，但他有更縝密的 return type
 */
function double4<T extends number | string>(
  x: T
): T extends string ? string : number;
function double4(x: any) {
  return x + x;
}

const number4 = double4(12); // number
const string4 = double4("x"); // string

function f2(x: number | string) {
  return double4(x);
}
```

上述的例子，多載對於一次傳入一種 type 可以很好的處理，但沒辦法處理會有多種 type 傳入的可能性
而透過條件型態，可以判斷本次傳入的 type 為何者，靈活的調整 return type 的結果
因此在上述的情境下，條件型態會是更好的選擇。

### 總結

雖然希望可以在這篇就把上篇預計剩下還沒提到的東西寫完，但一驚！內容好像還是太多了～ 所以預計會再分為兩篇：如何測試型態宣告、以及原來也可以在 TypeScript 中宣告 this 的型別！

做個總結，這篇主要提到的是如果自己想要寫型態宣告，並且開放給其他人使用，要注意哪些呢

1. 優先使用 structural typing 來提供非必要的依賴項目
   - 盡量不去做其他套件的型態去做依賴，因為可能會讓瀏覽器開發者或是 JavaScript 開發者增加不需要的依賴
2. 設計了 public 的 api，只要有用到的型別就把它 export 出去
3. 使用 JSDoc 或 TSDoc 格式的註釋來說明匯出的函式、類別與型態
4. 優先使用條件型態，而不是多載的型態宣告

### REF 參考

- {{< a href="https://www.books.com.tw/products/0010858053" target="_blank" >}}Effective TypeScript 中文版{{< /a >}}
