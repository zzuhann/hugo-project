---
title: "Effective TypeScript CH6 - JavaScript 與 TypeScript 裡面的 this"
description: "這篇文章提到了在 JavaScript 和 TypeScript 中的 this。從 JavaScript 的角度出發，討論了 this 的概念，再到 TypeScript 的 this，介紹如何為 function 加上 this 參數的型別。"
date: 2024-04-07T22:49:37+08:00
draft: false
author: "zzuhann"
slug: "this-in-typescript"
categories: ["TypeScript"]
keywords:
  - "this"
  - "TypeScript"
---

### 前言

這篇的內容主要會分為兩個部分：在 JavaScript 內的 this、以及在 TypeScript 內的 this。因為先前就對 `this` 感到陌生、好像了解了又好像沒有，書上對於 `this` 的著墨我覺得也不多，所以透過 huli 大大的這篇文章 {{< a href="https://blog.techbridge.cc/2019/02/23/javascript-this/" target="_blank" >}}淺談 JavaScript 頭號難題 this：絕對不完整，但保證好懂{{< /a >}} 增加自己對於 JavaScript 內 `this` 的理解，再回頭去看書中對於 TypeScript `this` 的介紹。

因此這篇文章的前半是我對 {{< a href="https://blog.techbridge.cc/2019/02/23/javascript-this/" target="_blank" >}}淺談 JavaScript 頭號難題 this：絕對不完整，但保證好懂{{< /a >}} 這篇文章的筆記、後半才是書中的筆記。

### 這篇文會提到

- [在 JavaScript 內的 this](#在-javascript-內的-this)
  - [離開物件導向後，this 就沒有什麼太大的意義](#離開物件導向後this-就沒有什麼太大的意義)
  - [this 的值跟在哪裡宣告沒有關係，而是跟如何呼叫有關](#this-的值跟在哪裡宣告沒有關係而是跟如何呼叫有關)
  - [透過 call, apply, bind 可以改變 this 的值](#透過-call-apply-bind-可以改變-this-的值)
  - [箭頭函式（arrow function） 中，本身沒有 this](#箭頭函式arrow-function-中本身沒有-this)
- [在 TypeScript 內的 this](#在-typescript-內的-this)
  - [TypeScript 中可以為 function 加上 this 這個參數的 type](#typescript-中可以為-function-加上-this-這個參數的-type)

### 在 JavaScript 內的 this

#### 離開物件導向後，this 就沒有什麼太大的意義

在 JavaScript 中，所有地方都能存取到 `this`，而作者認為「在離開物件導向後，`this` 就沒有什麼太大的意義」。

在脫離物件導向的情況下，`this` 的值在非嚴格模式時，瀏覽器下是 `window`、nodejs 下是 `global`，而在嚴格模式下皆為 `undefined`。

```js
function hello(name) {
  console.log(this);
}
hello(); // Window
```

```js
"use strict";
function hello(name) {
  console.log(this);
}
hello(); // undefined
```

#### this 的值跟在哪裡宣告沒有關係，而是跟如何呼叫有關

`this` 的值跟在哪裡宣告沒有關係，而是跟如何呼叫有關。

要知道 `this` 的值為何，作者提供的小撇步是在 function 後面加 `.call()` 並將 function 之前的對象填入 `.call(對象)` 中，就能知道當時的 `this` 的值為何。

```js
const person = {
  age: 35,
  hello: function () {
    console.log(this.age);
  },
  child: {
    age: 12,
    hello: function () {
      console.log(this.age);
    },
  },
};

person.hello();
// function 之前的對象為 person：person.hello.call(person) -> 35
const personHello = person.hello;
personHello();
// function 之前沒有對象：personHello.call() -> undefined
const child = person.child;
child.hello();
// function 之前的對象為 child：child.hello.call(child) -> 12
const childHello = child.hello;
childHello();
// function 之前沒有對象：childHello.call() -> undefined
```

#### 透過 call, apply, bind 可以改變 this 的值

雖然 `this` 有本身的預設值（前面兩點提到的），但也可以透過三種方式來改變。

1. `call`

```js
"use strict";
function hello(name) {
  console.log(this);
}
hello.call(123, "john"); // 123
```

2. `apply`：需要用 array 傳參數

```js
"use strict";
function hello(name) {
  console.log(this);
}
hello.apply(123, ["john"]); // 123
```

3. `bind`：在 `bind` 之後，`this` 的值就不能夠再改變
   使用 `bind` 會回傳新的 function

```js
"use strict";
function hello(name) {
  console.log(this);
}
const newHello = hello.bind(123);
newHello("john"); // 123
```

#### 箭頭函式（arrow function） 中，本身沒有 this

在 arrow function 中，他自己本身沒有 `this`。
arrow function 裡的 `this` 值來自宣告 arrow function 時的 `this` 是什麼值，那個值就是 arrow function 內 `this` 的值。

```js
const person = {
  age: 35,
  hello: function () {
    const innerHello = () => {
      console.log(this.age);
    };
    innerHello();
  },
};

person.hello(); // person.hello.call(person) -> 35
const personHello = person.hello;
personHello(); // personHello.call() -> undefined
```

### 在 TypeScript 內的 this

#### TypeScript 中可以為 function 加上 this 這個參數的 type

在 TypeScript 中可以為 function 加上 `this` 這個參數的 type，但實際上 `this` 參數在執行期中並不會被視為一個真正的參數，如下面的例子（取自書）：

```ts
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener("keydown", (e) => {
    fn(el, e); // ~~ error: Expected 1 arguments, but got 2.
  });
}
```

`this` 是由 JavaScript 設定的，所以要用 `bind` `apply` `call` 的方式傳入，不能用一般傳參數的方式。

```ts
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener("keydown", (e) => {
    fn.call(el, e);
  });
}
```

當宣告 this 的參數，TypeScript 會強迫你用正確的 `this` 來呼叫 function，透過這種方式可以確保型態安全。

如果是使用 arrow function，也和上面提到的 「箭頭函式（arrow function） 中，本身沒有 this」 概念一樣，在宣告 arrow function 時 `this` 是什麼值，arrow function 的 `this` 就是什麼值。下方的 `this` 指的是 `Foo` 這個類別，而 `Foo` 沒有 `innerHTML` 屬性，所以會報錯。

```ts
class Foo {
  registerHandler(el: HTMLElement) {
    addKeyListener(el, (e) => {
      this.innerHTML;
      // ~~~~~~~~~ Property 'innerHTML' does not exist on type 'Foo'
    });
  }
}
```

### 總結

先認識了在 JavaScript 內的 `this`，再去了解 TypeScript 內的 `this`，我覺得就能更好地去理解。這篇整理了一些 `this` 在 JavaScript 內的特性：

- 離開物件導向後，this 就沒有什麼太大的意義
- this 的值跟在哪裡宣告沒有關係，而是跟如何呼叫有關
- 透過 call, apply, bind 可以改變 this 的值
- 箭頭函式（arrow function） 中，本身沒有 this
  再去了解在 TypeScript 裡的 `this`：TypeScript 中可以為 function 加上 this 這個參數的 type

之後對 `this` 有更多了解時會再回到這邊擴充的～！

### REF 參考

- {{< a href="https://blog.techbridge.cc/2019/02/23/javascript-this/" target="_blank" >}}淺談 JavaScript 頭號難題 this：絕對不完整，但保證好懂{{< /a >}}
- {{< a href="https://www.books.com.tw/products/0010858053" target="_blank" >}}Effective TypeScript 中文版{{< /a >}}
