---
title: "Effective TypeScript CH6 - 測試型別宣告"
description: "探索如何測試 TypeScript 中的型別宣告，這篇文章深入探討了兩種主要策略：程式碼內部和型別檢查器外部的測試方法。從確保 function 正常執行到驗證返回值的型別，這篇文章提供了幾個技巧和工具，幫助你確保 TypeScript 型別的準確性和可靠性。"
date: 2024-04-07T11:20:02+08:00
draft: false
author: "zzuhann"
slug: "testing-type-declarations"
categories: ["TypeScript"]
keywords:
  - "測試型態宣告"
  - "TypeScript"
  - "Testing Type Declaration"
---

### 前情提要

上一篇{{< a href="https://www.zzuhann.work/posts/type-declaration-tips/" target="_blank" >}}Effective TypeScript CH6 - 撰寫 TypeScript 型態宣告時的注意事項{{< /a >}}提到的，在撰寫型態宣告時要注意的事項，而這一篇會提到撰寫完之後，要如何測試型態宣告。

書中提到的方法主要分為兩種：一種在程式碼內測試型態宣告、一種則是在型態檢查器外面。

### 這篇文會提到

- [在程式碼內，常見的做法](#在程式碼內常見的做法)
  - [確保 function 可以正常執行，不會報錯](#確保-function-可以正常執行不會報錯)
  - [將結果指派給特定型態的變數](#將結果指派給特定型態的變數)
  - [定義一個協助函式](#定義一個協助函式)
- [在型態檢查器外面進行測試](#在型態檢查器外面進行測試)

### 在程式碼內，常見的做法

#### 確保 function 可以正常執行，不會報錯

有一個 function

```ts
map(["2017", "2018", "2019"], (v) => Number(v));
```

透過測試來確保這個 function 可以正常執行

```ts
// 測試 map 函式是否正確轉換字串陣列為數字陣列
test("map function converts string array to number array", () => {
  const input = ["2017", "2018", "2019"];
  const expected = [2017, 2018, 2019];
  const result = map(input, (v) => Number(v));
  expect(result).toEqual(expected);
});
```

這個方式可以確保這個 function 可以正常執行，不會報錯。問題在於沒有測試到 return 的 type 是否正確，所以接下來的方式會去檢查 return 的 type 是否正確。

#### 將結果指派給特定型態的變數

```ts
const lengths: number[] = map(["john", "may"], (name) => name.length);
```

這個方式確實可以確保 return 的 type 是我們所期望的，但同時也會帶來一個問題：為了賦予結果特定的 type，必須多宣告一個變數，而這個變數除了用來檢查 return 的 type 以外沒有其他用途。

這樣的情況下，eslint 可能會有未使用變數的 warning 或 error，需要再額外處理。

![ts-eslint-ignore](/gif/ts-eslint-ignore.gif)

#### 定義一個協助函式

```ts
function assertType<T>(x: T) {}
assertType<number[]>(map(["john", "paul"], (name) => name.length));
```

這個方式的問題在於

1. 檢查的是兩種型態的可賦值性，而不是相等性

```ts
const n = 12; // n 的 type 為 12
assertType<number>(n); // OK，因為 12 是 number 的子型態
```

在這個例子中 `n` 的型別為 12，透過 `assertType` 預期它的型態為 `number`，因為 `12` 為 `number` 的子型態，所以是會通過型態檢查的，因為 `12` 也是 `number` 的一種。

2. 物件型態檢查為 structural typing，只要有符合我需要的 type 即可，如果有多的 key 不會檢查

```ts
const beatles = ["john", "paul", "george", "ringo"];
assertType<{ name: string }[]>(
  map(beatles, (name) => ({
    name,
    inYellowSubmarine: name === "ringo",
  }))
); // OK
```

在這個例子中預期結果型態為 `{name: string}[]`，然而實際上 map 出來的結果為 `{name: string, inYellowSubmarine: boolean}[]`，因為只預期需要 `name` 這個 key，所以 `{name: string, inYellowSubmarine: boolean}[]` 他只會去檢查是否有 `name`、符合 `string`、且為陣列型態，即使有多的 key 也不會檢查，因為已經符合了我所需要的 type。

3. function 參數可以比型態宣告得多

```ts
const add = (a: number, b: number) => a + b;
assertType<(a: number, b: number) => number>(add); // OK

const double: (a: number, b: number) => number = (x: number) => 2 * x;
assertType<(a: number, b: number) => number>(double); // OK
```

這是常見的 JavaScript pattern：function 接受多個參數，在呼叫時要傳入幾個參數（可以是 0 個，當然，不能傳超過指定參數的數量）由呼叫者決定。為了和 JavaScript 中常見的 pattern 保持一致，TypeScript 也會容許這個行為。

可以參考這篇文章，這是來自 TypeScript 的 FAQ：{{< a href="https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-with-fewer-parameters-assignable-to-functions-that-take-more-parameters" target="_blank" >}}Why are functions with fewer parameters assignable to functions that take more parameters?{{< /a >}}

如果想要確保參數的數量及型別，可以把 function 的參數及 return type 分開來測試。

```ts
const double = (x: number) => 2 * x;
let p: Parameters<typeof double> = null!;
assertType<[number, number]>(p);
//                           ~ Argument of type '[number]' is not
//                             assignable to parameter of type [number, number]
let r: ReturnType<typeof double> = null!;
assertType<number>(r); // OK
```

### 在型態檢查器外面進行測試

因為有可能透過一些方式，把 `any` 指派給整個模組，讓 TypeScript 不再對整個模組進行型態檢查，即使你對各種型態宣告做了嚴格的測試。
（作者在書中提到 `declare module 'overbar'`的方式會造成這個結果，但是我試不出來，所以提供給大家可能有這樣的情形 QQ ～～）

因此作者認為在型態檢查器外面進行檢查是比較好的做法，以 DefinitelyTyped 存放的型態宣告而言(之前提到的 `@types`)，是透過 `dtslint` 在型態檢查器外面進行檢查。

語法來說可能會類似這樣（取自書中程式碼）

```ts
const beatles = ["john", "paul", "george", "ringo"];
map(
  beatles,
  function (
    name, // $ExpectType string
    i, // $ExpectType number
    array // $ExpectType string[]
  ) {
    this; // $ExpectType string[]
    return name.length;
  }
); // $ExpectType number[]
```

作者有提到，`dtslint` 不是檢查可賦值性，而是檢查各個代號的型態，也會進行字面比較，符合在編輯器中手動測試型態宣告式的做法：`dtslint` 是將這個程序自動化。

不過這一塊我沒有涉獵過，所以先記錄起來～！往後接觸到的話應該會更有心得。

### 總結

這篇提到了如何測試撰寫好的型態宣告
主要分為在程式碼內測試、以及在型態檢查器外測試

- 程式碼內測試，常見的做法
  - 確保 function 可以正常執行，不會報錯
  - 將結果指派給特定型態的變數
  - 定義一個協助函式
- 在型態檢查器外面進行測試

### REF 參考

- {{< a href="https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-with-fewer-parameters-assignable-to-functions-that-take-more-parameters" target="_blank" >}}Why are functions with fewer parameters assignable to functions that take more parameters?{{< /a >}}
- {{< a href="https://www.books.com.tw/products/0010858053" target="_blank" >}}Effective TypeScript 中文版{{< /a >}}
