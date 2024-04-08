---
title: "Effective TypeScript 系列：使用 TypeScript 後要注意的版本管理、介紹 @types"
description: "本篇文章提到在 TypeScript 開發中需要注意的地方：包括套件版本控制、綑綁套件、以及介紹 @types。"
date: 2024-04-06T14:05:08+08:00
draft: false
author: "zzuhann"
slug: "typescript-types"
categories: [TypeScript]
tags: [Effective TypeScript]
keywords:
  - "TypeScript"
  - "types"
  - "type version"
---

### 動機

和轉職班的同學在近期一起讀了 Effective TypeScript 這本書，讀到第六章：型態宣告與 `@types`，提到了使用 TypeScript 之後要注意的點、使用 TypeScript 後常見的 `@types` 從何而來、自己在寫型態宣告時需要注意的點，以及如何去測試型態宣告。

我覺得這個章節是個平常在開發時可能常見，但比較少去意識到的部分，所以將這些重點整理起來，也讓之後的自己更方便回顧～
本篇會先提及的部分是使用 TypeScript 之後要注意的點、使用 TypeScript 後常見的 `@types` 從何而來，避免啪拉啪拉讓文章拉的太長！接著也會持續把內容在下一篇補齊～

### 這篇文會提到

- [決定要使用 TypeScript 後，要注意的地方變多了](#決定要使用-typescript-後要注意的地方變多了)
- [綑綁套件與型態宣告的版本](#綑綁套件與型態宣告的版本)
- [上面常常提到的 @types 是什麼呢？](#上面常常提到的-types-是什麼呢)

### 決定要使用 TypeScript 後，要注意的地方變多了

#### 決定要使用 TypeScript 之後，要在哪裡安裝 TypeScript 及 TypeScript 相關的套件呢？

TypeScript 作為一種開發工具，且 type 在執行期並不存在，因此作者建議把 TypeScript 及 `@types` 相關的東西放在 devDependencies。

相較於 dependencies 主要是用在 production 環境，上線後如果沒有這個套件就會報錯、無法執行，devDependencies 主要用於開發、測試程式碼，上線後就不需要用到。

上述提到 TypeScript 主要是在開發時使用，type 在上線後也不存在，ts 相關的程式碼會被轉為 JavaScript 來執行，所以作者認為把 TypeScript 及 `@types` 相關的東西放在 devDependencies 會是比較合適的。

```
npm install --save-dev @types/react
```

#### 使用 TypeScript 需要注意三個依賴項目的版本

1. TypeScript 的版本
2. 套件的版本
3. 套件 `@types` 的版本

這三個版本需要注意什麼呢？

- 套件與其對應的 `@types` 版本是否同步
  不同步的情況下，不論是套件的版本比 `@types` 更新、或是 `@types` 的版本比套件更新，都可能導致 type 對應不上，導致 ts 報錯。
- 套件可能使用比專案更新的 TypeScript 版本

1. 在這個狀況下，可以去更新 TypeScript 版本、使用舊的 `@types` 版本，如果前面這兩個方法你都不想或無法做到的話，在遇到型別錯誤的情況下，可以使用 declare module 手動宣告

根據 ChatGPT，declare module 實作的方式為
如果你使用了一個名為 `foo` 的套件，且它的型態有問題，可以建立一個 `foo-extension.d.ts` 的檔案（.d.ts 前面是什麼都可以）

```ts
// foo-extension.d.ts
// declare module '套件名稱'

declare module "foo" {
  export interface Person {
    name: string;
  }
}
```

```ts
import * as foo from "foo-extension";
const person: foo.Person = {
  name: "John",
};
```

2. 為特定版本 ts 安裝 `@types` `npm install --save-dev @types/lodash@ts3.1`

### 綑綁套件與型態宣告的版本

有些套件會透過綑綁的方式，將套件本身和套件的型態宣告綁起來，確實可以解決套件和 type 版本不相符的問題。但同時也會遇到一些其他的問題

綑綁 types 會寫在 package.json 裡面，指向 `.d.ts` 檔案的 types 欄位

```json
{
  "name": "left-pad",
  "version": "1.3.0",
  "description": "String left pad",
  "main": "index.js",
  "types": "index.d.ts"
}
```

1. 可能無法透過 declare module 的方式進行擴增
2. 可能發布的時候沒問題，但新版的 TypeScript 有問題 -> 可能為了這個原因，只能使用舊版 ts
3. 如果綑綁的型態依賴另一個套件的型態宣告（通常依賴的 `@types` 在它的 devDependencies 內），安裝這個套件 `npm install A` 時，A 套件依賴的 devDependencies 不會被下載，只會下載 A 套件的 dependencies {{< a href="https://juejin.cn/post/7077520444332441630" target="_blank" >}}（什么包放在 devDependencies？什么包放在 dependencies？）{{< /a >}}，但如果將依賴的型態宣告安裝在 dependency 內，JavaScript 用戶也會安裝到，但 JavaScript 使用者其實是不需要 `@types` 套件的（畢竟他們沒有用 TypeScript）

作者建議除非程式庫是用 TypeScript 寫的，否則不要綑綁型態宣告。

### 上面常常提到的 @types 是什麼呢？

`@types` 是套件的開發者在維護的嗎？
`@types` 來自於 DefinitelyTyped：是由社群維護的 JavaScript 套件的型態定義集。
{{< a href="https://github.com/DefinitelyTyped/DefinitelyTyped" target="_blank" >}}DefinitelyTyped Github{{< /a >}}

如果套件本身沒有 TypeScript 的型態宣告，通常比較熱門的套件都可以在 DefinitelyTyped 找到型態，例如使用 `lodash` 套件，可以安裝 `@types/lodash`。

```
npm install --save-dev @types/lodash
```

### 總結

1. 使用 TypeScript 時應該注意 TypeScript 本身的版本、套件以及相對應的 `@types` 的版本
2. 綑綁套件與型態宣告的版本可能會遇到一些問題
3. 使用 TypeScript 後經常搭配套件下載的 `@types` 為 DefinitelyTyped 社群維護

### REF 參考

- {{< a href="https://www.books.com.tw/products/0010858053" target="_blank" >}}Effective TypeScript 中文版{{< /a >}}
- {{< a href="https://juejin.cn/post/7077520444332441630" target="_blank" >}}什么包放在 devDependencies？什么包放在 dependencies？{{< /a >}}
- {{< a href="https://github.com/DefinitelyTyped/DefinitelyTyped" target="_blank" >}}DefinitelyTyped Github{{< /a >}}
