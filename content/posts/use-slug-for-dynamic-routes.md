---
title: "使用 Next.js 的 Dynamic Routes 來處理多層結構的頁面"
description: ""
date: 2024-10-05T00:00:01+08:00
draft: false
author: "zzuhann"
slug: "use-slug-for-dynamic-routes"
categories: [技術]
tags: [Next.js]
keywords:
    - "Next.js"
    - "Dynamic Routes"
    - "多層結構"
    - "網址"
    - "網頁"
---
實作「常見問題」頁面的時候，常見問題可能會是多層結構

以{{< a href="https://support.rhinoshield.io/hc/zh-tw/" target="_blank" >}}犀牛盾的常見問題頁{{< /a >}}來說：

1. 最外層是 Categories：iPhone 16 專區｜產品使用｜訂單常見問題｜會員常見問題 ...
2. 點擊任何一個 Category 後，會看到該 Category 對應到的 Sections
3. 再點擊任一個 Section 後，會看到該 Section 對應到的 Articles
4. 點擊任一個 Article，會進入特定的文章頁面

最近接觸到的常見問題頁，在點擊切換 Category/Section/Article 時，網址是不會有變化的，而是透過 state 來控制當前顯示的 component

隨著需求改變，希望可以調整成 Category/Section/Article 都能夠有自己的相對應網址，這樣客服在回答問題時，可以直接將網址傳給對方、或是在某些頁面將用戶導到相對應的文章網址，用戶就不用自己從入口開始漫無目的的尋找。

專案是使用 next.js page router，所以我最直接的想法就是建新的資料夾來設定對應的 page

```
pages/
└── rhinoshield/
    ├── faq/
    │   ├── index.tsx                => /rhinoshield/faq
    │   ├── [categoryId]/
    │   │   ├── index.tsx            => /rhinoshield/faq/[categoryId]
    │   │   └── [sectionId]/
    │   │       └── index.tsx        => /rhinoshield/faq/[categoryId]/[sectionId]
    │   │       └── [articleId]/
    │   │           └── index.tsx    => /rhinoshield/faq/[categoryId]/[sectionId]/[articleId]

```

然後都吃同一個 component，所以等於寫了好幾次重複的東西，只為了單獨設定 page

同事提供給我一個 next.js 的 {{< a href="https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes" target="_blank" >}}Dynamic Routes{{< /a >}} 的 Optional Catch-all Segments


之前我使用過的 dynamic routes 是像 `[id]` 這種，這個方式讓 `/a`、`/b`、`/123` 等都包含在裡面，透過 `router.query` 就能取得 `a`、`b`、`123` 這些值。

如果需要更多 query，可以使用 `[...slug]`，如下面這張圖

![nextjs-dynamic-routes](/img/use-slug-for-dynamic-routes-1.png)

以上面這個例子來說，如果需要讓 `/shop` 也在這個可能裡面呢？就可以使用 `[[...slug]]`（Optional Catch-all Segments）

使用這個方式，不用新增過多 page folder，就可以更好的達成我想要的結果，根據我的需求，我將資料夾結構調整成以下：

```
pages/
└── rhinoshield/
    ├── faq/
    │   ├── [[...slug]].tsx
```

調整成這樣之後，以下網址都會 render `[[...slug]].tsx` 裡面的 component
1. `/rhinoshield/faq`
2. `/rhinoshield/faq/123`
3. `/rhinoshield/faq/123/123`
4. `/rhinoshield/faq/123/123/123`

不過這個的小缺點是允許被無限增長的，想要限制 query 的量的話需要靠自己來 handle，我自己是在 `[[...slug]].tsx` 內再去判斷 `slug` 長度是否大於 3，如果是的話則 return 404 page。

拿取 slug 的方式，我目前是透過 `getServerSideProps` 取得 slug 後傳給 component

```tsx
export const getServerSideProps = async ({query}) => {
	const { slug = [] } = query

	return {
		props: {
			slug
		}
	}
};
```


（感覺寫得很茫，太想睡了但還是要來日更！今天先發出來，日後再來回顧要怎麼修改可以更好）