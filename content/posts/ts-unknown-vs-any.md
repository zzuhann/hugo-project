---
title: "Effective TypeScript: unknown vs any"
description: ""
date: 2024-10-07T00:20:27+08:00
draft: false
author: "zzuhann"
slug: "ts-unknown-vs-any"
categories: [技術]
tags: [TypeScript]
keywords:
    - "TypeScript"
    - "unknown"
    - "any"
    - "型別"
    - "型態"
    - "型別安全"
    - "型別檢查"
    - "型別推斷"
    - "型別推論"
---
在 TypeScript 中使用 `unknown` 及 `any` 時，雖然看起來都是因為不知道型別時會使用的型別，兩者之中還是有一些區別。

之前讀書會的時候有同學分享過一些差異，把筆記整理一下分享出來，之後有更多相關的補充資料也會繼續調整這篇文章。

### `unknown` 型態是 `any` 型態的安全替代品

在不知道型別時，比起馬上使用 `any`，可以考慮使用 `unknown` 來作為替代品，有提到幾個原因

1. 針對 `unknown` 型別的變數做任何操作都會產生警告

```tsx
let unknownVar: unknown;

unknownVar.b // 'unknownVar' is of type 'unknown'
unknownVar() // 'unknownVar' is of type 'unknown'
```

如果你很肯定你當下在使用時，會有 `b` 這個屬性可以使用、或是肯定 `unkownVar` 是 function 的話，也許可以在使用時進行一些調整：進行斷言 `as` 或是透過 type guard 進行更安全的型別檢查

- 斷言 `as`

```tsx
let unknownVar: unknown;

(unknownVar as { b: string }).b;
(unknownVar as VoidFunction)();
```

使用斷言的感覺有點像是在跟 TypeScript 說「我比你更懂！」，所以如果不是超級篤定的話，使用 type guard 後再進行斷言或許也是不錯的方式

- type guard
```tsx
let unknownVar: unknown;

// 確認 unknownVar 是物件、不是 null 且有 b 這個屬性
if (typeof unknownVar === "object" && unknownVar !== null && "b" in unknownVar) {
  console.log((unknownVar as { b: string }).b);
}

if (typeof unknownVar === "function") {
  (unknownVar as VoidFunction)();
}
```

2. 任何型態都可以指派給 `unkown`， `unknown` 不能指派給其他型態
```tsx

let unknownVar: unknown;

// 可以指派任何型態給 unknown
unknownVar = "string value";
unknownVar = 42;
unknownVar = true;
unknownVar = { key: "value" };
unknownVar = [1, 2, 3];
unknownVar = () => console.log("Hello");

let stringVar: string;

// unknown 不能指派給其他型態
stringVar = unknownVar // Type 'unknown' is not assignable to type 'string'.
```

相比 `any` 呢？

1. 任何型態都可以指派給 `any`，`any` 可以指派給其他型態（除了 `never`）

```tsx
let anyVar: any;

// 任何型態都可以指派給 any
anyVar = "string value";
anyVar = 42;
anyVar = true;
anyVar = { key: "value" };
anyVar = [1, 2, 3];
anyVar = () => console.log("Hello");

// any 可以指派給其他型態
let anyVar: any;
let stringVar: string;
let numberVar: number;
let booleanVar: boolean;

anyVar = "Hello";

stringVar = anyVar;
numberVar = anyVar;
booleanVar = anyVar;
```

型態是 `any` ，就等於 typescript 不再對這個變數進行型別檢查，也減少了型別的安全性，且 any 有蔓延性（下次再來寫這個！），一不注意就會讓更多變數也變成 any。

```tsx

let anyVar: any;
let neverVar: never;

// any 不能指派給 never
neverVar = anyVar; // Type 'any' is not assignable to type 'never'. 
```

`never` 是「永遠不會發生的值」，所以他也不應該等於任何值，`any` 就也沒有辦法指派給 `never`。

Ref: Effective TypeScript 3,4,5 章 讀書會