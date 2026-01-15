結論：下次你要改任何東西，就照 README 的三個入口走：

改文案/FAQ：event.json

改公告：Sheet「公告」B1~B5

改乾淨世界按鈕/來源規則：index.html
# stateorgans.tw 報名網站｜維護 README（完整新版）

> 目的：下次回來不用翻對話，照著這份就能改字、改公告、改按鈕、改來源、上線與排錯。

---

## 0) 目前架構（你要記住的就這幾個）
- 前端：GitHub + Netlify（靜態網站）
- 主要檔案：
  - `index.html`：首頁（置頂按鈕、來源識別、公告欄、FAQ 展開等）
  - `thanks.html`：感謝頁（顯示報名序號、來源等）
  - `event.json`：活動內容文案（最常改）
  - `assets/poster.jpg`：海報圖片
- 後端：Google Apps Script（Web App）
  - `doPost`：寫入 Google Sheet「名單總表」
  - `doGet (mode=notice)`：提供公告欄資料（JSONP）
- Google Sheet：
  - 工作表 `名單總表`：報名資料
  - 工作表 `公告`：公布欄（B1~B5 控制）

---

## 1) 最常修改：event.json（改文案、FAQ、更多介紹）
### 1-1 換行規則（JSON 內文字好看）
- `\n`：換行
- `\n\n`：空一行（分段）
- 建議：一段最多 3 行，手機最好讀。

### 1-2 你目前用到的欄位（重點）
#### 更多介紹（可展開：看更多介紹）
```json
"更多介紹": "調查與訪談為主的紀錄片\n讓人重新思考生命的價值與底線\n映後有與談／交流（若有再放）"
````

#### 常見問題（可展開：常見問題）

```json
"常見問題": [
  {"Q":"需要帶票嗎？","A":"不用。入場資訊會用手機／簡訊通知。"},
  {"Q":"可以幫朋友報名嗎？","A":"可以，但請填能聯絡到的電話，並確定對方能到場。"},
  {"Q":"臨時不能到怎麼辦？","A":"請提前告知，名額會釋出給候補。"}
]
```

#### 注意事項（建議用陣列，每點一句最整齊）

```json
"注意事項": [
  "每人最多可報名 2 張（可分開多次報名）",
  "若臨時無法到場，請提前告知，名額會釋出給候補",
  "請填可接聽的電話／LINE，避免漏收入場通知"
]
```

> 影片策略：你目前選「外部連結」，所以 **不使用「影片嵌入網址」** 欄位（避免頁面冒出內嵌影片）。

---

## 2) 公布欄（動態更新，不用改網站）

### 2-1 在哪改？

Google Sheet → 工作表 `公告`

### 2-2 儲存格規則（B1~B5）

| 儲存格 | 用途       | 範例            |
| --- | -------- | ------------- |
| B1  | 顯示開關     | TRUE / FALSE  |
| B2  | 公告標題     | 公告｜名額更新       |
| B3  | 公告內容     | 2/28 剩最後 20 位 |
| B4  | 按鈕文字（可空） | 前往頻道          |
| B5  | 按鈕連結（可空） | https://...   |

### 2-3 使用方式

* 沒公告：B1 = FALSE（B2/B3 有字沒關係，網頁不顯示）
* 要公告：B1 = TRUE，填 B2/B3（要按鈕再填 B4/B5）

---

## 3) 置頂按鈕：乾淨世界（文字/網址在哪改？）

### 3-1 在哪改？

`index.html` 搜尋：`topExternalLink`

### 3-2 改網址（連到哪裡）

改這行 `href="..."`：

```html
href="https://www.ganjingworld.com/zh-TW/playlist/...."
```

### 3-3 改顯示文字（手機/電腦分開）

```html
<span class="sm:hidden">乾淨世界</span>
<span class="hidden sm:inline">乾淨世界頻道</span>
```

---

## 4) 影片（你選 A：外部連結）

* 不內嵌 iframe
* 用按鈕跳外部（YouTube 或 乾淨世界）
* 若你把影片連結放在 `event.json`，之後只改 `event.json` 即可換影片

---

## 5) 來源識別：?src=A/B/其它 → 寫入備註

### 5-1 你現在要的規則

* `?src=A` → `【來源】FB`
* `?src=B` → `【來源】SM`
* `?src=其他` → `【來源】Other`

### 5-2 前端程式（index.html）要長這樣

在 `index.html` 找到 `getSourceTag()`，改成：

```js
// 來源識別：src=A => FB、src=B => SM，其它有填就 => Other
function getSourceTag() {
  const p = new URLSearchParams(location.search);
  const src = (p.get("src") || "").trim().toUpperCase();
  if (!src) return "";

  if (src === "A") return "【來源】FB";
  if (src === "B") return "【來源】SM";
  return "【來源】Other";
}
```

### 5-3 怎麼用？（你發出去的網址）

* FB：`https://www.stateorgans.tw/?src=A`
* SM：`https://www.stateorgans.tw/?src=B`
* 其它：`https://www.stateorgans.tw/?src=C`（會變 Other）

### 5-4 在 Google Sheet 哪裡看？

看 `名單總表` 的「備註」欄（或你設定的追蹤欄位）。

* 正常會看到 `【來源】FB/SM/Other`
* 若同時有 UTM 也會看到 `【追蹤】utm_source=...` 一起寫入

---

## 6) GAS Web App URL（表單送出要用的那條網址）在哪改？

> 你表單送出是用 `fetch(no-cors)` 丟到 GAS Web App。

### 6-1 在哪改？

`index.html` 搜尋：

* `GAS_URL`
* 或 `script.google.com/macros/s/`

你會看到類似：

```js
const GAS_URL = "https://script.google.com/macros/s/XXXX/exec";
```

要換 Web App 就改這條網址。

---

## 7) 公布欄 API（GAS doGet mode=notice）怎麼用？

> 這是為了讓靜態網站也能讀公告（避免 CORS），用 JSONP 的方式回傳。

### 7-1 網址格式

在 `index.html` 會組出一條像這樣的讀取網址：

```
GAS_URL?mode=notice&callback=renderNotice
```

* `mode=notice`：告訴 GAS 你要讀公告
* `callback=renderNotice`：JSONP 回呼函式名（前端必須有這個 function）

### 7-2 回傳長什麼樣？

GAS 會回傳類似：

```
renderNotice({"enabled":true,"title":"...","message":"...","linkText":"...","linkUrl":"..."});
```

前端拿到後就把公告顯示在頁面上。

### 7-3 若公告不顯示，先檢查

1. Google Sheet `公告` 的 B1 是否為 TRUE
2. GAS 是否已部署成 Web App（且網址正確）
3. `index.html` 是否用的是同一條 GAS_URL（不要兩條混用）
4. 如果你有改 Code.gs，記得「重新部署」Web App（Update deployment）

---

## 8) 上線流程（你每次改完必做）

### 8-1 GitHub 網頁版（你目前用這個）

1. 點檔案（例如 `event.json` 或 `index.html`）
2. 點 ✏️ Edit 修改
3. 按右上綠色 **Commit changes...**
4. 寫訊息（例如：`更新 event：新增 FAQ`）
5. Commit 完，Netlify 會自動部署

### 8-2 手機看不到更新（快取）

* 用無痕視窗開
* 或網址加 `?v=1`（例如 `https://www.stateorgans.tw/?v=1`）

---

## 9) 快速排錯（改了沒變）

先檢查這四件事：

1. ✅ 是否改到正確檔名？

   * 首頁一定是 `index.html`
   * 文案一定是 `event.json`
2. ✅ 是否真的按了 **Commit changes**？
3. ✅ Netlify Deploys 是否有新的一筆 Published？
4. ✅ 手機快取：無痕或加 `?v=1`

---

## 10) 「Commit changes」= 備份（你已理解的重點）

* 你按一次 **Commit changes** 就多一個「版本記錄」
* 檔案不會變兩份，仍然只有 `index.html`
* 你可以在檔案頁按 **History** 看每次改了什麼，必要時回復舊版

---

## 11) 建議保留的習慣（避免以後忘）

* 大改前先 commit 一次，訊息寫 `備份：大改前`
* 出問題就能從 History 找回上一版

```

如果你想，我也可以再把「公告 doGet 的 Code.gs 程式碼」也貼到 README 最後，讓你連程式都不用回頭找。
::contentReference[oaicite:0]{index=0}
```
