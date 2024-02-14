---
title: "使用 structuredClone() 來進行深拷貝"
description: "在工作中，深拷貝是一項常見的任務，以避免對原始物件的影響。傳統的方法是使用JSON.parse(JSON.stringify(x))，但它有一些限制。最近發現了structuredClone函數，它可以彌補這些缺點，並且不需要額外的套件。這篇筆記介紹了深淺拷貝的常見方法，評估了structuredClone的優缺點，並提供了手寫深拷貝的方法。"
date: 2024-02-14T14:29:05+08:00
draft: false
author: "zzuhann"
slug: "deep-copy-structuredClone"
categories: ["前端"]
keywords:
- "deep copy"
- "structuredClone"
---
### 動機
在工作上有時會使用到深拷貝，避免影響到原有的物件，起初都是使用 `JSON.parse(JSON.stringify(x))` 的方式來做深拷貝，因為不用再額外引入套件、也是比較常見的做法。

有一次看到在介紹 `structuredClone` 這個 function 的文章，可以解決 `JSON.parse(JSON.stringify(x))` 會遇到的缺點，同樣也不用額外引入套件，就能達到深拷貝的需求。因為是第一次看到 `structuredClone` 這個做法，所以想做成筆記，同時介紹常見的深淺拷貝方式、使用 structuredClone 的優缺點，以及最後如何實作手寫深拷貝。

### 這篇文會提到
- [簡介淺拷貝與深拷貝](#簡介淺拷貝與深拷貝)
- [常見的淺拷貝作法](#常見的淺拷貝作法)
- [常見的深拷貝作法](#常見的深拷貝作法)
- [使用 structuredClone() 來進行深拷貝](#使用-structuredclone-來進行深拷貝)
- [實作手寫深拷貝](#實作手寫深拷貝)

### 簡介淺拷貝與深拷貝
- **淺拷貝**：在拷貝時，只針對原始型別進行拷貝，創建新的記憶體位址；但對於物件類（object, array 等）不會建立新的記憶體位址，而是共同指向相同的位址。

如果 result 針對 `result.array` 做了 `push` 的動作，會讓 `result.array` 的陣列多一個 `4` 元素，但因為和 `被拷貝對象.array` 共用記憶體位址，實際上是更改到同一個 array。
```
const 被拷貝對象 = {
    string: 'hi',
    array: [1, 2, 3],
}

const result = { ...被拷貝對象 }
result.array.push(4)

console.log(result.array) // [1, 2, 3, 4]
console.log(被拷貝對象.array) // [1, 2, 3, 4]
```

- **深拷貝**：拷貝對象與被拷貝對象各自指向不同的記憶體位址。

### 常見的淺拷貝作法
要達到淺拷貝，有以下方式
- Object spread: `{ ...被拷貝對象 }`
```
const a = {
    nestedArr: [1, 2, 3],
    string: 'hi'
}

const b = { ...a }
```

- `Object.assign({}, 被淺拷貝的對象)`
```
const a = {
    nestedArr: [1, 2, 3],
    string: 'hi'
}

const b = Object.assign({}, a)
```

- `Object.create(被拷貝對象)`
```
const a = {
    nestedArr: [1, 2, 3],
    string: 'hi'
}

const b = Object.create(a)
```

### 常見的深拷貝作法
要達到深拷貝，通常會有兩種方式
- `JSON.parse(JSON.stringify(被拷貝對象))`
```
const a = {
    nestedArr: [1, 2, 3],
    string: 'hi'
}

const b = JSON.parse(JSON.stringify(a))
```
這個方法主要會碰到的缺點是，`JSON.stringify` 無法處理複雜的物件，只能處理簡單的物件、陣列以及原始型別的值
1. 如果他遇到 `Date` 則會將他轉為字串
2. 如果遇到 `Set`, `Map`, `Regex`, `File`, `Error`, `DOM節點`，會把它們轉為空物件
3. 遇到 `undefined`, `function` 型別，甚至會直接忽略
```
const exampleObj = {
    dateString: new Date(),
    set: new Set([1, 2, 3]),
    map: new Map([[1, 'one'], [2, 'two'], [3, 'three']]),
    regex: /hello/,
    file: new File(['file contents'], 'example.txt'),
    error: new Error('example error'),
    domNode: document.createElement('div'),
    undefinedValue: undefined,
    func: function() {
        console.log('example function');
    }
};

const result = JSON.parse(JSON.stringify(exampleObj));
console.log(result)

/*
{
    "dateString": "2024-02-14T08:48:25.118Z",
    "set": {},
    "map": {},
    "regex": {},
    "file": {},
    "error": {},
    "domNode": {}
}
*/
```
- `_.cloneDeep(被拷貝對象)`
可以解決 `JSON.parse(JSON.stringify(被拷貝對象))` 遇到的問題，上述型別可以被拷貝，而主要考慮的點在於要使用 `_.cloneDeep()` 需要額外 import 這個 function 或是 lodash，可能導致檔案變大、加載時間變長，但如果原本在專案裡面就有使用 lodash，`_.cloneDeep()` 也是一個方式。

### 使用 structuredClone() 來進行深拷貝
`structuredClone()` 可以解決 `JSON.parse(JSON.stringify(被拷貝對象))` 大部分的問題，可以被拷貝的型別包括 `Date`, `Set`, `Map`, `Error`, `RegExp`, `ArrayBuffer`, `Blob`, `File`, `ImageData` 等，且為 Web API 提供的方法，所以不用再額外 import 套件，可以直接使用（Node.js 17 以上的版本也支援 structuredClone 這個方法了）。

而 `structuredClone()` 也有一些不能被深拷貝的型別：`function`、`DOM 節點`、`原型鏈`
- **拷貝 function**: 會報錯
```
structuredClone({ fn: () => { } })
// VM901:1 Uncaught DOMException: Failed to execute 'structuredClone' on 'Window': () => { } could not be cloned.
```

- **拷貝 DOM 節點**：會報錯
```
structuredClone({ el: document.body })
// VM930:1 Uncaught DOMException: Failed to execute 'structuredClone' on 'Window': HTMLBodyElement object could not be cloned.
```

- **原型鏈**：去深拷貝一個實例的時候，拷貝出來的結果只會有該實例的屬性，不會拷貝到實例的原型鏈
```
class MyClass {
  constructor(value) {
    this.value = value;
  }
  
  getValue() {
    return this.value;
  }
}

let obj1 = new MyClass(10);
let obj2 = structuredClone(obj1);

// obj2 為 { value: 10 }
```

### 實作手寫深拷貝
如果不用上述提到的三種方式來進行深拷貝，而選擇用手寫的方式

1. 建立一個 function，傳入兩個參數：obj（要被拷貝的對象）、weakMap（記錄是否已經處理過該對象）
2. 先檢查是否為物件，如果不是物件則代表為原始型別，可以直接回傳結果
3. 查看 weakMap 是否已經存在這個 obj 鍵對，如果存在則直接回傳 weakMap 該鍵對的值
4. 檢查是否為 `Date` 或是 `RegExp` 型別，如果是的話則用 `new Date()` 或 `new RegExp()` 創建新的實例
5. 以上過渡之後，剩下可能的型別剩下物件（可能是陣列或物件）
6. 建立一個新的物件：如果 obj 是陣列的話，則建立空陣列、如果 obj 不是陣列的話，則建立空物件
7. 將這個新物件存入 weakMap 
8. 使用 `Object.entries` 進行遞迴，直到所有對象被處理完

```
function deepClone(obj, map = new WeakMap()) {
	if(obj === null || typeof obj === 'object') return obj;
	
	if(map.has(obj)) return map.get(obj);

	if(obj instanceof Date) return new Date(obj);
	if(obj instanceof RegExp) return new RegExp(obj);

	let result = Array.isArray(obj) ? [] : {};
	map.set(obj, result);
	for(const [key, value] of Object.entries(obj)) {
		result[key] = deepClone(value)
	}

	return result;
}
```

### 結語
以上是最近看了幾篇文章後整理的深淺拷貝的方式，以及近期在開發上常用的 `structuredClone()`，也因為看到幾篇文章有提到如何手寫深拷貝，搭配 ChatGPT 也把這個 function 整理在文章裡面。

如果有任何想法或錯誤，再麻煩跟我說聲！非常感謝觀看的您～
可以透過信箱(zzuhanlin@gmail.com)或 [linkedin](https://www.linkedin.com/in/zzuhanlin/) 找到我

### REF 參考文章
[Deep Cloning Objects in JavaScript, the Modern Way](https://www.builder.io/blog/structured-clone#property-descriptors-setters-and-getters)
[請實踐 JavaScript 淺拷貝 (shallow copy) 和深拷貝 (deep copy)](https://www.explainthis.io/zh-hant/swe/shallow-copy-and-deep-copy)
[structuredClone() global function](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone)
[How to Fix the "structuredClone is not defined" Error in Node.js](https://www.codingbeautydev.com/blog/javascript-structuredclone-is-not-defined)
