---
title: "使用 flex box 來防止字體被動態調整？"
description: ""
date: 2024-10-02T23:21:56+08:00
draft: false
author: "zzuhann"
slug: "use-flex-to-limit-font-size"
categories: [技術]
tags: [CSS]
keywords:
    - "CSS"
    - "font-size"
    - "flex"
    - "android"
    - "字體大小"
    - "手機"
    - "設備"
    - "調整"
    - "dynamic"
    - "adjust"
    - "font"
    - "size"
    - "flex"
    - "box"
    - "android"
    - "device"
    - "mobile"
    - "phone"
    - "prevent"
    - "limit"
    - "dynamic"
    - "adjust"
    - "font"
    - "size"
    - "flex"
    - "box"
    - "android"
    - "device"
    - "mobile"
    - "phone"
---

最近在公司專案上遇到一個有趣的點，是在 android 的手機調整系統字體大小時，即使前端頁面有針對字體設定固定大小，例如 `font-size: 14px` ，字也會隨之變大變小。

有趣的點在於並不是所有字都會跟著動態調整，而且也會因為不同的 android 設備型號不同，有不同的結果。

例如
```jsx
const Item = styled.div`
	font-size: 14px;
`

const Component = () => {
	return (
		<Item>這是第一行字</Item>
		<Item>這是第二行字</Item>
		<Item>這是第三行字</Item>
	)
}

```

有些 android 設備，這三行字可能全部都是 `14px`，有些是只有第一行被動態調整，而第二三行則維持 `14px`。

對於這個現象我也還是很疑惑，因為 component 都一樣的情況下，為什麼會有不同的結果呢？

ChatGPT 有提供我一個想法是，也許是因為某個「字」的原因，不過這個我就沒有去實測了。

我又去看專案中其他有被動態調整的文字，發現似乎加上 `display: flex` 會讓字體大小被固定為指定的 px。

```jsx
const Item = styled.div`
	font-size: 14px;
`

const ItemWithFlex = styled.div`
	font-size: 14px;
	display: flex;
`

const Component = () => {
	return (
		<Item>這是第一行字</Item>
		<ItemWithFlex>這是第二行字</ItemWithFlex>
		<Item>這是第三行字</Item>
	)
}

```

使用 `ItemWithFlex` 的文字，以目前看到的設備來說（使用了三台 samsung 測試），文字大小都固定為 `14px`。

把 `display: flex` 加到那些有機率被動態調整的文字的 component 上，似乎都能夠變為正常的指定大小。

不過以這個方向來找相關資料，資料不多、或是沒有什麼官方說法來驗證這件事。

相關的 stack overflow 問題：{{< a href="https://stackoverflow.com/questions/38346494/why-is-flex-affecting-font-size-on-ios/38347131#" target="_blank" >}}Why is flex affecting font size on iOS?{{< /a >}}
提到 `flex` 和 `float` 有可能導致字體大小的問題，不知道有沒有人有遇過類似的問題，以及怎麼解決～

目前在工作上我先以 `display: flex` 作為短解，觀察看看這個解法是否能夠達成固定字體大小的目的。

註解：這篇文只探討 android 而沒有討論 iOS

{{< a href="https://css-tricks.com/your-css-reset-needs-text-size-adjust-probably/" target="_blank" >}}Your CSS reset needs text-size-adjust (probably){{< /a >}} 這篇文章提到在 css reset 時可以加上 `text-size-adjust` 屬性，公司專案原本就有 `-webkit-text-size-adjust: 100%;` 的設定，所以 iOS 是沒有字體大小會被動態調整的問題，所以只聚焦在 android 上