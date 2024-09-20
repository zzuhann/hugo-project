---
title: "在 Hugo 裡安裝 Disqus 啟用留言板功能！"
description: "這個部落格正式啟用留言功能啦～！目前是留言板是採用 Disqus，也想紀錄自己從設定到安裝的過程，希望也可以幫助到想要在 Hugo 中加留言板的人！"
date: 2024-09-13T01:05:07+08:00
draft: false
author: "zzuhann"
slug: "add-disqus-in-hugo"
categories: [技術]
tags: [Hugo]
keywords:
    - "hugo 留言"
    - "disqus hugo"
---

### 一些前言

前天跟了一場{{< a href="https://www.facebook.com/smallgoldfishblog/?locale=zh_TW" target="_blank" >}}小金魚的人生實驗島{{< /a >}}的分享（預計明天或後天可以來簡單分享），開始想把一些東西分享在這個部落格，而不單單只是工作與技術相關的，希望這裡可以成為紀錄我各個面向的地方，畢竟人不只是在工作而已，是由很多不同的面貌組成的。

小金魚提到，要「公開」而且是可以收到回饋的地方，這個部落格目前並沒有留言板讓有其他想法的人留言，只能透過寄信或是在社群聯絡我的方式來交流一些想法，所以我決定在這個部落格加一個留言板。

先說結論：我使用的是 Disqus，我沒有針對各個留言板做太多功課，但有一些基本的需求：可以放到 hugo 裡面使用、可以不登入就留言，因為預計之後文章的主題不限於技術相關，所以也不是所有人都有 github 帳號可以使用，其他的我就沒關係～ 可以先試用看看研究一下有沒有遇到痛點再來跳槽。優缺點分析我是參考這篇：{{< a href="https://ivonblog.com/posts/replace-giscus-with-disqus/" target="_blank" >}}為什麼我要用Disqus取代Giscus當作Hugo留言板{{< /a >}}

### 先在 Disqus 註冊與設定
#### 先去{{< a href="https://disqus.com/" target="_blank" >}}官網{{< /a >}}註冊帳號

到右上角 `Get Started` 進行註冊或 `Login` 登入
![disqus官網](/img/add-disqus-in-hugo-1.png)

#### I want to install Disqus on my site

![choose-service](/img/add-disqus-in-hugo-2.png)

#### Create a new site
1. 輸入你的 Website Name，他會幫你轉換成下面的網址
2. Category 都可以
3. Language 就照你想要的語言
4. 輸入完之後就 Create Site

![enter-info](/img/add-disqus-in-hugo-3.png)

#### 選擇免費的 Basic 專案，點擊 Subscribe Now
#### 選擇平台
因為 Hugo 不在這些選項裡面，所以選擇下面的按鈕
`I don't see my platform listed, install manually with Universal Code`

![choose-manually-universal-code](/img/add-disqus-in-hugo-4.png)

#### Universal Code install instructions
接下來會到這個頁面（最上面是一個 How to Install Disqus Using the Universal Code 的影片）
直接滑下來到最底，右下角有一個 `Configure` 的按鈕
按下去，然後不用更改，再按 `Next` 按鈕
會到這邊
照自己的需求去選就好，之後也還可以更改

![choose-preference](/img/add-disqus-in-hugo-5.png)

選完之後就按右下角的 `Complete Setup`
設定就差不多完成哩～ 接下來可以到 `Setting` 頁面，原本的位置應該是在 `Start`

![remember-short-name](/img/add-disqus-in-hugo-6.png)

點擊上面選單的 `Setting` 進到設定頁面，左邊選擇 `General` ，等等會需要用到 `Shortname`

### Hugo 安裝 Disqus

#### 在設定檔案中 `config.toml` 加入 `disqusShortname`
```
disqusShortname: <你的 Shortname>
```
位置不要放錯～ 我覺得可以盡量往上面放，放在 `title` 附近


#### 在 `layouts/partials` 新增 `disqus.html`
記得不要加到 theme 資料夾裡面的 partials 了（我不小心放在這邊，結果想說為什麼正式環境都沒有）
`disqus.html` 放上這段

```html
<div id="disqus_thread"></div>
<script type="text/javascript">

(function() {
    // Don't ever inject Disqus on localhost--it creates unwanted
    // discussions from 'localhost:1313' on your Disqus account...
    if (window.location.hostname == "localhost") return;

    var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
    var disqus_shortname = '{{ .Site.Config.Services.Disqus.Shortname }}';
    dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
    (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
<a href="https://disqus.com/" class="dsq-brlink">comments powered by <span class="logo-disqus">Disqus</span></a>
```

如果想在本地開發環境看到留言板的話，可以把下面這段註解
```js
if (window.location.hostname == "localhost") return;
```

#### 將 `disqus.html` 引入到適合的位置
觀察一下如果要放東西在你的文章最下面，哪個檔案、哪個地方最適合
就把下面這段放進去那邊

```html
<div class="disqus markdown" style="width: 100%;">
	{{ partial "disqus.html" . }}
</div>
```

我加上 `style="width: 100%;"` 是因為 iframe 不知道為什麼會有 `width: 1px !important` 導致畫面上留言板只有 1px，加上這個 style 之後就可以了
也可以嘗試看看把 `width: 100%` 拿掉看會不會有影響，也許不一定要加

以上～ 把這段過程保留下來，之後自己需要再用的時候可以再來複習一次
希望可以幫助到想要加留言板的你們～～！如果有卡關可以在下面留言，測試看看這個留言板是不是有正常運作

### 參考資料
{{< a href="https://dizang.io/posts/hugo-install-disqus/" target="_blank" >}}Hugo 安裝 Disqus 留言板{{< /a >}}

(2/365)