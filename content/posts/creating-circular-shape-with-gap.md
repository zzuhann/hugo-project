---
title: "CSS Challenge Day 6 | 做一個有缺口的圓形"
description: "有缺口的圓形要怎麼做呢？最近開始做 css 的 100 天 Challenge，Day 6 有一個這樣的圖形需求，我先是朝了 clip path、svg 等方向來思考、找相關資料，但似乎沒有符合我需求的，於是直接找了網路上做 Day6 的人的分享，原來用做圓形的方式來思考就可以～"
date: 2024-09-18T23:56:27+08:00
draft: false
author: "zzuhann"
slug: "creating-circular-shape-with-gap"
categories: [技術]
tags: [CSS Challenge]
keywords:
  - "CSS Challenge"
  - "CSS"
  - "CSS Circle"
---


{{< figure src="/img/creating-circular-shape-with-gap-1.png" width="350" >}}


這樣類似圓形的圖形會怎麼做呢？

最近開始做 css 的 100 天 Challenge，Day 6 有一個這樣的圖形需求，我先是朝了 clip path、svg 等方向來思考、找相關資料，但似乎沒有符合我需求的，於是直接找了網路上做 Day6 的人的分享（{{< a href="https://ithelp.ithome.com.tw/articles/10290513?sc=rss.qu" target="_blank" >}}[Day 7] Profile: 來製作個簡單樸實的個人檔案吧!{{< /a >}}）

發現只要像平常做圓形一樣去做就可以： `width` 、 `height` 、 `border-radius` 以及 `border`

關鍵在於想要製造缺口的那一邊的 border 顏色設為 `transparent`

又是把圖形想得太複雜的一天！

今天 CSS Challenge 的成果：
{{< codepen id="vYqqqqy">}}


（推坑一下 BOYNEXTDOOR！）

（哇光是研究 hugo 要怎麼嵌入 codepen 預覽又另外花了一些時間，明天來分享！）
