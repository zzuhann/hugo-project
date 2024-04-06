---
title: "實作 react-i18next：使用 i18-ally、google sheet 維護多國語系"
description: "公司導入了多國語系 i18n-next 後，遇到的問題是如何更快速的維護需要被翻譯的文字、如何減少人力但能達到相同的效果？這篇文章會分享 i18n-ally 的使用，以及如何在 google sheet 維護多國語系。"
date: 2024-01-29T16:42:45+08:00
draft: false
author: "zzuhann"
slug: "i18n-google-sheet"
categories: ["前端"]
keywords:
  - "i18n"
  - "多國語系"
  - "i18n-next"
---

### 前情提要

公司在約半年前為了做多國語系，導入了 i18n-next。前期僅限於測試階段，所以需要被翻譯的文字並不算多，後來正式導入之後，需要維護網站上所有文字，以及三種語言，加上每個人手上都有不少的工作量，沒有太多的時間去人工維護翻譯的整理與更新。

因此我希望可以讓這段多國語系翻譯的過程盡量自動化，減少人力的維護，在這篇文章主要會提及如何使用 google sheet 來與 PM、翻譯人員進行合作，以及如何將程式碼的變數更新到 google sheet 上，又是如何將在 google sheet 上翻譯完成的文字更新到程式碼內，以及如何透過套件及 function 來讓維護多國語系更為沒有壓力。

### 基本設定

大致設定，是參考這篇文章：{{< a href="https://medium.com/@a401120174/%E4%BD%BF%E7%94%A8-react-i18next-%E4%BE%86%E6%89%93%E9%80%A0%E5%A4%9A%E5%9C%8B%E8%AA%9E%E8%A8%80%E7%B6%B2%E7%AB%99%E5%90%A7-i18n-5b9c70c05d24" target="_blank" >}}使用 react-i18next 來打造多國語言網站(i18n){{< /a >}}

稍微不同的地方是，我們在專案中使用的翻譯檔案是照語言區分：zh-TW.json, en.json, vi.json

### 延伸做出的客製化

除了上述基本的設定，公司專案也做了一些客製化：

1. VS Code Extension: {{< a href="https://github.com/lokalise/i18n-ally" target="_blank" >}}i18n-ally{{< /a >}}
   在還沒下載 i18n-ally 之前，使用 i18n-next 在程式碼裡面會長的像下面這樣：
   ![Imgur](https://i.imgur.com/Z67hoUO.png)
   _(圖片取自 https://github.com/lokalise/i18n-ally)_
   對於 key 越來越長的情況下，基本上很難閱讀。

    <br/>
   下載之後，會變成以下：

   ![Imgur](https://i.imgur.com/vtpAfK7.png)
   _(圖片取自 https://github.com/lokalise/i18n-ally)_

   設定可以參考這篇文章：{{< a href="https://wualnz.com/Vue-i18n-%E7%A5%9E%E5%99%A8-i18n-ally/" target="_blank" >}}[Vue] i18n 神器 i18n-ally{{< /a >}}
   <br/>
   除了可以在程式碼內顯示特定語言目前的翻譯文字，我覺得更為方便的地方是
   ![Imgur](https://github.com/lokalise/i18n-ally/raw/screenshots/hover.png?raw=true)
   _(圖片取自 https://github.com/lokalise/i18n-ally)_
   hover 到 key 上面，i18n-ally 就會顯示目前這個 key 對應到各語言的文字，點擊右邊數過來第二個的筆 ✏️，輸入這個 key 在該語言對應到的文字，他就會自動更新到該語言的 json 檔，不用辛苦去 json 檔找到位置後再新增。
   <br/>

2. 讓 i18n-next 的 t function 不只是傳入 key，也能快速帶入預設值
   僅帶入 key 的情況下，如果想要針對文字去做修改，即使是全域搜尋也不容易一次找到要修改的地方，因此針對 i18n-next 提供的 t function 再寫了一個 function，讓團隊成員除了傳入 key 以外，還需帶入預設值。
   <br/>

   ```ts
   import { t as trans } from "i18next";

   export const t = (key, defaultValue, params = {}) => {
     return trans(key, { ...params, defaultValue }) || "";
   };
   ```

   之後團隊要新增 i18n 的 key 時，就可以這樣使用，如果其他語言尚未補上翻譯，則就會顯示預設值。

   ```ts
   import { t } from "utils/i18n";

   <p>t( 'customer.email', '電子信箱', )</p>;
   ```

3. 使用 google sheet 來維護越來越多的 key 與需要翻譯的內容
   隨著專案越來越大，需要翻譯的文字開始變多、以及變更需求的同時，文字也可能需要隨之調整，因此使用 google sheet 來維護翻譯。

   <br/>

   初期，需要翻譯的文字還沒那麼多、也還沒有樹立規則該如何維護這份文件，基本上是工程師在做完需求、更新完語言 json 檔之後，再手動從 json 檔複製 key 及文字到 google sheet。隨著需求與工作量都變多，光是複製貼上去維護 google sheet 的時間就要花上半小時以上，於是我和同事有初步討論之後，當時有以下痛點：

   1. 從 json 檔複製貼上到 google sheet 的這一段過程，雖然很直覺，但是很耗時
   2. 如果要變更需求、調整文字，語意也隨之不同，則需要重新翻譯，需要讓翻譯人員知道哪裡被調整
   3. 新增新的 key 到 google sheet 上，需要讓翻譯人員知道哪些是新增的
   4. 翻譯人員翻譯完成後，要如何快速更新 json 檔

   <br/>

   針對上述痛點，思考的解法如下：

   1. 工程師僅需完成更新程式碼、json 檔的部分，從 json 檔更新到 google sheet 的這一段過程，我會寫一段 node.js 程式碼去跑排程，每半個月執行一次更新 google sheet

   <br/>

   這是我們在 google sheet 上的格式 key - 中文 value - 英文 value - 越南文 value：
   ![Imgur](https://i.imgur.com/jnFoR7K.png)

   <br/>

   舉例來說，json 檔會是這樣設定（其他語言則為將蘋果、香蕉替換為對應的語言翻譯）：
   ![Imgur](https://i.imgur.com/qPF6AvZ.png)

   <br/>

   並且在專案中去執行以下程式碼，將 json 檔內容更新至 google sheet

   (亦可到 github gist 查看更好看的排版：{{< a href="https://gist.github.com/zzuhann/617c216ac4ddf94ff871732337e4095a" target="_blank" >}}語言 json 檔 conver to google sheet{{< /a >}})

   <br/>

   ```js
   const fs = require("fs-extra");
   const path = require("path");
   const { google } = require("googleapis");

   const auth = new google.auth.GoogleAuth({
     // 設定您的 Google Sheets API 金鑰憑證
     keyFile: "./script/googleSheetConfig.json", // 更改為 Google Sheets API 金鑰憑證放置的位置
     scopes: ["https://www.googleapis.com/auth/spreadsheets"],
   });
   const sheets = google.sheets({ version: "v4", auth });

   /**
    * spreadsheetId: Google Sheets 的 ID
    * sheetName: 工作表名稱
    */
   const spreadsheetId = { spreadsheetId };
   const sheetName = { sheetName };

   /**
    * JSON 檔轉換為 ['key', 'value', 'value', 'value'] 格式
    */
   const data = [];

   const files = [
     path.resolve(__dirname, "../src/locales", `zh-TW.json`),
     path.resolve(__dirname, "../src/locales", `en.json`),
     path.resolve(__dirname, "../src/locales", `vi.json`),
   ];
   const zhTWData = JSON.parse(fs.readFileSync(files[0], "utf8"));
   const enData = JSON.parse(fs.readFileSync(files[1], "utf8"));
   const viData = JSON.parse(fs.readFileSync(files[2], "utf8"));

   const translationData = {}; // 用來存放翻譯資料的物件

   const translations = [
     {
       language: "zhTW",
       data: zhTWData,
     },
     {
       language: "en",
       data: enData,
     },
     {
       language: "vi",
       data: viData,
     },
   ];

   async function jsonToSheet() {
     for (let i = 0; i < translations.length; i++) {
       const language = translations[i].language;
       processObject(translations[i].data, "", translationData, i);
       if (language === "vi") {
         // 將翻譯資料轉換為陣列格式並放入 data 中
         Object.entries(translationData).forEach(([key, value]) => {
           data.push([key, ...value]);
         });
       }
     }

     /**
      * 存取 Google Sheets 資料
      * 若 Google Sheets 無資料，則直接寫入資料
      * 若 Google Sheets 有資料，則比對資料是否有異動
      * 異動情況：
      * 1. 三個語言中只要有其中一個是空值，則該列多一欄為 need to translate
      * 2. 三個語言中只要有其中一個有異動，則在該欄 +4 欄放置異動後的值方便比對
      * 3. 若 JSON 檔的 key 不存在於 Google Sheets，則該列多一欄為 added
      */
     await sheets.spreadsheets.values
       .get({
         spreadsheetId,
         range: `${sheetName}!A2:D`,
       })
       .then((res) => {
         const rows = res.data.values || [];
         if (rows.length === 0) {
           // 如果此表無內容
           try {
             // 寫入數據到 Google Sheets
             sheets.spreadsheets.values.update({
               spreadsheetId,
               range: `${sheetName}!A2:D`,
               valueInputOption: "USER_ENTERED",
               requestBody: { values: [...data] },
             });
             console.log("JSON data imported to Google Sheets successfully.");
           } catch (error) {
             console.error(
               "Error importing JSON data to Google Sheets:",
               error
             );
           }
         } else {
           const header = rows.map((row) => row[0]);
           const length = data.length;
           for (let i = 0; i < length; i++) {
             const key = data[i][0];
             const rowIndex = header.indexOf(key);
             if (rowIndex > -1) {
               // 如果此 key 已經存在於表
               for (let j = 1; j < 4; j++) {
                 if (!rows[rowIndex][j]) {
                   rows[rowIndex][4] = "need to translate";
                 } // 如果中文、英文、越南文有任何一格為空，最後一格會註記 'need to translate'
                 if (rows[rowIndex][j] && rows[rowIndex][j] !== data[i][j]) {
                   rows[rowIndex][j + 4] = data[i][j];
                   rows[rowIndex][4] = "updated";
                 } // 如果任何一格有進行變動，則最後一格會註記 'updated'
               }
             } else {
               // 如果此 key 並不存在於表
               const newRow = [
                 key,
                 data[i][1],
                 data[i][2],
                 data[i][3],
                 "added",
               ];
               for (let j = 1; j < 4; j++) {
                 if (!newRow[j]) {
                   newRow[4] = "need to translate";
                 }
               }
               rows.push(newRow);
             }
           }
           sheets.spreadsheets.values.update({
             spreadsheetId,
             range: `${sheetName}!A2:H`,
             valueInputOption: "USER_ENTERED",
             requestBody: { values: [...rows] },
           });
         }
       })
       .catch((err) => {
         console.log(err);
       });
   }

   function processObject(obj, prefix, translationData, languageOrder) {
     for (const key in obj) {
       const value = obj[key];
       if (typeof value === "object") {
         const newPrefix = prefix ? `${prefix}.${key}` : key;
         processObject(value, newPrefix, translationData, languageOrder);
       } else {
         const columnName = prefix ? `${prefix}.${key}` : key;
         const translations = translationData[columnName] ?? [];
         if (translations.length < languageOrder) {
           translations[languageOrder] = value;
         } else {
           translations.push(value);
         }
         translationData[columnName] = translations;
       }
     }
   }
   ```

   2. 在從 json -> google sheet 的過程 1. 會去比對各個語言當前在 json 的文字以及對應到 google sheet 的文字是否相同，如果不同，會註記為 _updated_ 2. 如果在 google sheet 中有需要被翻譯的地方（例如英文處是空格，尚未被翻譯），會註記為 _need to translate_ 3. 如果是本次新增，而各個語言都已經被填寫，會註記為 _added_ ，讓翻譯人員 double check 是否需要修改 4. 善用 google sheet 裡面的公式，我有設定如果最後一格有任何註記，則整行都會被上背景色，讓翻譯人員一眼看出哪裡要被注意
      ![Imgur](https://i.imgur.com/DPf4bH4.png)

   <br/>

   3. 翻譯人員翻譯完後，會告知 PM，再由 PM 開單給工程師，手動執行 node.js 程式碼，將 google sheet 目前的翻譯文字更新到 json 檔

   <br/>

   google sheet -> json 檔這段的程式碼
   (亦可到 github gist 查看更好看的排版：{{< a href="https://gist.github.com/zzuhann/83993063dca25115d42b9dd1f178d8ef" target="_blank" >}}google sheet convert to json{{< /a >}})

   ```js
   const fs = require('fs-extra');
   const unflatten = require('flat').unflatten;
   const path = require('path');
   const { google } = require('googleapis');

   const auth = new google.auth.GoogleAuth({
   // 設定您的 Google Sheets API 金鑰憑證
   keyFile: './script/googleSheetConfig.json', // keyFile 在專案的位置
   scopes: ['https://www.googleapis.com/auth/spreadsheets'],
   });
   const sheets = google.sheets({ version: 'v4', auth });
   const spreadsheetId = {googleSheetId}
   const sheetName = {googleSheet 標籤名稱，建議用英文};

   const files = ['zh-TW', 'en', 'vi'];
   const filePaths = [
       path.resolve(__dirname, '../src/locales', `zh-TW.json`),
       path.resolve(__dirname, '../src/locales', `en.json`),
       path.resolve(__dirname, '../src/locales', `vi.json`),
   ];

   /**
   * 讀取 google sheet 資料並轉換成物件格式
   * 如果 key 存在於 google sheet 及 json 檔案且有值，google sheet 的值會覆蓋 json 檔案的該 key 的值
   */
   sheets.spreadsheets.values
   .get({
       spreadsheetId,
       range: `${sheetName}!A2:D`,
   })
   .then(res => {
       const existingFiles = {
           'zh-TW': fs.readJSONSync(filePaths[0]),
           en: fs.readJSONSync(filePaths[1]),
           vi: fs.readJSONSync(filePaths[2]),
       };
       const result = res.data.values;
       for (let i = 1; i < result.length; i++) {
           const key = result[i][0];
           // 語言包
           for (let t = 0; t < files.length; t++) {
               const value = result[i][t + 1] ? result[i][t + 1] : '';
               if (value !== undefined && value !== '') {
                   const keys = key.split('.');
                   let current = existingFiles[files[t]];
                   for (let m = 0; m < keys.length; m++) {
                       const k = keys[m];
                       if (!current[k]) {
                           current[k] = {};
                       }
                       if (m === keys.length - 1) {
                           current[k] = value;
                       } else {
                           current = current[k];
                       }
                   }
               }
           }
       }

           for (const fileName of files) {
           fs.writeJSONSync(
               path.resolve(__dirname, '../src/locales', `${fileName}.json`),
               unflatten(existingFiles[fileName], { object: true }),
               { spaces: 2 },
           );
       }
   });
   ```

### 在那之後

目前大致上是使用此方式維護多國語系，後續像是常用的文字應該要被集中管理，翻譯就不用經常翻譯相同的文字，以及這兩段自動化的程式碼在未來應該也要被優化。但至少已經解決了當初團隊所面臨到的痛點，也分享給大家～！如果有想法可以在透過信箱(zzuhanlin@gmail.com)或 [linkedin](https://www.linkedin.com/in/zzuhanlin/) 向我分享或討論，感謝！
