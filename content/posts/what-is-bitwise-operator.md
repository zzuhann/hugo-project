---
title: "初見 Bitwise Operator：一些參考文章整理"
description: ""
date: 2024-10-08T00:54:53+08:00
draft: false
author: "zzuhann"
slug: "what-is-bitwise-operator"
categories: [技術]
tags: [JavaScript]
keywords:
    - "Bitwise Operator"
    - "位元運算子"
    - "JavaScript"
    - "位元運算"
    - "位元運算符"
    - "位元運算子應用"
    - "位元運算子情境"
    - "位元運算子用法"
    - "位元運算子好處"
    - "位元運算子優點"
    - "位元運算子概念"
    - "位元運算子初見"
    - "位元運算子初學者"
    - "位元運算子入門"
---

今天在工作上遇到了 bitwise operator 這個位元運算子，是第一次接觸到、也是第一次在實務上看到有人使用，所以看了一些文章。

因為覺得自己只是稍微看懂了一些用法和概念，還沒有辦法完全用自己的話說出來，所以想要把今天看到的文章、對於我理解 bitwise operator 概念與情境有幫助的文章放在這邊。

[JavaScript基礎篇（3）：位元運算子（Bitwise Operator）](https://blog.raven.tw/JavaScript-3-Bitwise-Operator-02c4572589fd45839be53a3dcf0b8fbe)
[一件Intent教我的事： Bitwise Operation](https://carterchen247.medium.com/%E4%B8%80%E4%BB%B6intent%E6%95%99%E6%88%91%E7%9A%84%E4%BA%8B-bitwise-operation-dd3a388ae34e)
[Javascript中的位元運算子 (1)](https://ithelp.ithome.com.tw/articles/10129361)

以及我詢問了 chatgpt 後的一些情境用法（針對《一件Intent教我的事： Bitwise Operation》這篇文章提到使用 bitwise operator的好處）

1. 用一個 field 表示多個狀態，節省記憶體空間：在某些情況下，儲存多個 boolean 值可能會佔用不必要的空間。相較於使用多個 boolean 變數，可以用位元運算符來壓縮這些狀態。

```js
// 使用 bitwise operator 儲存四個狀態
const READ = 1 << 0;    // 0001
const WRITE = 1 << 1;   // 0010
const DELETE = 1 << 2;  // 0100
const UPDATE = 1 << 3;  // 1000

let permissions = READ | WRITE;  // 使用位元運算符將兩個權限結合

console.log(permissions);  // 3 (0011 表示同時擁有讀取與寫入的權限)
```

2. 簡化狀態檢查，避免多個 boolean 值需要個別維護：當使用 bitwise 方式來儲存狀態時，可以輕鬆地檢查特定的狀態是否啟用，而不需要單獨檢查每個 boolean 變數。例如，想要檢查某個使用者是否有「讀取」權限：

```js
// 檢查是否有讀取權限
if (permissions & READ) {
  console.log("Has read permission");
} else {
  console.log("No read permission")
}

// 檢查是否有刪除權限
if (permissions & DELETE) {
  console.log("Has delete permission");
} else {
  console.log("No delete permission");
}

```

只需要一個變數就能管理所有的狀態，並且可以快速檢查某個權限是否被啟用。

以上，是我作為 bitwise operator 初入門（認識他的第一天）的一些參考文章以及 chatgpt 根據文章中提到的優點的補充，之後如果在其他情境下使用到、有更加理解的話，會發其他文章來分享！