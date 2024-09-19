---
title: "實作 | 把 codepen 嵌入 Hugo 文章"
description: "透過 Hugo 的 shortcodes，可以在文章內插入 codepen 的預覽，讓文章更美觀、也可以達到創作者的需求，把 HTML 內容放到 Markdown 內。一起來實作看看吧～～！"
date: 2024-09-20T01:11:31+08:00
draft: false
author: "zzuhann"
slug: "insert-codepen-in-hugo"
categories: [Hugo]
tags: [Hugo]
keywords:
    - "Hugo"
    - "shortcodes"
    - "codepen"
    - "embed codepen in hugo"
    - "Hugo codepen"
    - "Hugo shortcodes"
    - "Hugo embed codepen"
    - "Hugo codepen shortcode"
    - "Hugo codepen"
---

為了在 Hugo 文章內插入 codepen 的預覽，找到了 Hugo 的 `shortcodes` 的方式， Markdown 有時候無法達成一些需求，導致創作者會需要把 HTML 放到 Markdown 裡面，讓整體不是很美觀。透過 `shortcodes` 可以兼顧美觀、也可以達到創作者的需求，把 HTML 內容放到 Markdown 內。

以上簡介以及用法可以參考 Hugo 的這篇文章：{{< a href="https://gohugo.io/content-management/shortcodes/" target="_blank" >}}Shortcodes{{< /a >}}

實作的部分，以插入 codepen 來說
1. 打開你想要 embed 入文章的 codepen 作品
2. 按右下角的 `Embed`，會出現一個彈窗
3. 右下角會有 `HTML(Recommended)` 這段程式碼，把它複製起來
4. 回到你的 Hugo 專案，在 `layouts/shortcodes` 新增一個檔案，名字隨意，我是取 `codepen.html`
5. 把剛剛複製的程式碼貼到這個檔案
6. 因為是範本，所以我有把一些比較針對單一作品、但又不想每次都傳參數進來的欄位拔掉，應該不會有影響
7. 主要改動是把中間的 `<span>` 拿掉、`data-slug-hash` 的值將以參數傳入，是作品的 id

data-user="yourname" 要改成你的 username
```
<p class="codepen" data-height="500" data-default-tab="result" data-slug-hash="{{ .Get "id" }}" data-user="yourname" style="height: 500px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;"></p>

<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>
```

接著在你想要插入的文章內文中，找好適當位置貼上下面這段程式碼（因為直接貼上來的話，會吃到 shortcodes，所以我用圖片）

{{< figure src="/img/insert-codepen-in-hugo-1.png" >}}

說明：
1. `codepen`：如果你剛剛新增的 html 檔案名稱不是 `codepen.html` 而是 `XXX.html` 就把這邊的 `codepen` 改成 `XXX`
2. `id` 為你的 codepen 作品 id，你的 codepen 作品網址應該會是 `https://codepen.io/<yourname>/pen/<這邊是 id>`
3. 如果你想要動態調整高度的話，也可以新增 `height` 參數，再調整一下範本的內容

希望有成功！