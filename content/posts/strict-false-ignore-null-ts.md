---
title: "tsconfig 設定 strict: false 會忽略 null 的型別判定"
description: "tsconfig 將 strict 設定為 false 會讓 TypeScript 忽略 null 和 undefined 的型別判定，可能會導致沒有 handle 好 falsy 值的情況，導致非預期的 crash 發生～！"
date: 2024-09-25T01:08:16+08:00
draft: false
author: "zzuhann"
slug: "strict-false-ignore-null-ts"
categories: [技術]
tags: [TypeScript]
keywords:
    - "TypeScript"
    - "strict"
    - "null"
    - "型別判定"
---

今天在重構一些邏輯的時候

遇到了以下這個狀況（非當事程式碼，這是用來 demo 用的）

```tsx
const memoTest = useMemo(() => {
    if (isSuccess) {
      return { isSuccess: true };
    }
    return null;
  }, [isSuccess]);
```

我預期 `memoTest` 的型別應該被判定為 `{ isSuccess: boolean } | null`

但實際上是這樣：沒有 `null` 的型別

{{< figure src="/img/strict-false-ignore-null-ts-1.png" width="350" >}}


即使把程式碼改為

```tsx
const memoTest = useMemo<{ isSuccess: boolean } | null>(() => {
    if (isSuccess) {
      return { isSuccess: true };
    }
    return null;
  }, [isSuccess]);
```
也會是同樣的型別判定

同事發現是因為 `tsconfig.json` 的 `strict` 設為 `false`

`strict` 設為 `false` 會讓 `null` 和 `undefined` 可以賦值給其他型別

例如
```tsx
let a: string = '123';
a = null
```
這樣是 valid 的，typescript 不會報錯

typescript 的文件有提到（{{< a href="https://www.typescriptlang.org/tsconfig/#strictNullChecks" target="_blank" >}}strictNullChecks{{< /a >}}）

> When `strictNullChecks` is `false`, `null` and `undefined` are effectively ignored by the language.

> 當 `strictNullChecks` 為 `false`（`strict` 設為 false 時，`strictNullChecks` 即使沒有設定，也會是 `false`），`null` 和 `undefined` 會完全被忽略。

`strict` 設為 `false` 的情況下

TypeScript 會忽略 `null` 和 `undefined` 的可能性

如果沒有做好 falsy 值的處理的話，可能會導致非預期的結果（壞去～～）