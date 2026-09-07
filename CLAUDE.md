# CLAUDE.md — shihpc.github.io 接手速覽

<!-- CANON:BEGIN v1 -->
<!-- 唯一事實來源＝shihpc/claude-harness 的 CANON.md。以下區塊在五個 repo 的 CLAUDE.md 頂端
     有 byte-identical 逐字副本，由各 repo 的 .github/workflows/canon.yml 守門（比對 sha256）。
     改動流程：先改 claude-harness/CANON.md → 跑 tools/sync_canon.py 同步五份 → 更新守門 hash。
     不要只改單一 repo，CI 會擋下來。 -->

## 通用工作鐵律（五個 repo 逐字相同，勿單獨修改）

1. **機密**：token／金鑰一律走 `.env` 或 Actions secret，絕不寫進任何會 commit 的檔案、log 或
   對話輸出。commit 前用 `git diff --staged` 檢查有無夾帶金鑰樣式字串（`sk-ant-`、`ghp_`、`eyJ` 開頭）。
2. **指揮官不下場**：掃 repo、通讀 >300 行的檔、一次讀 >3 個檔、查網頁研究、批次改檔、
   驗收改過的東西——這六類一律派 subagent，主對話只收結論＋`檔案:行號`。
   雲端 session 的 subagent 派工（含第 3 條驗收）已獲常備授權，需要時直接派，不需逐次詢問。
3. **先寫驗收條件再動手**：動手前先寫下目標專案完整路徑＋怎樣算完成＋怎麼驗。改完派
   fresh-context subagent 驗收——**改東西的 agent（含主對話自己）不得擔任驗收者**。
4. **不確定不亂說**：陳述事實（尤其技術細節、數字、外部服務的限制與行為）要嘛附佐證（官方
   文件、實測、`檔案:行號`），要嘛明說「這點我不確定，需要查證」，不可憑印象當確定講。
   區分「已驗證事實」與「推測」，推測要標明。
5. **一次只做一件事**：只做明確要求的那件事，做完給簡短結果；少主動丟一堆延伸提案。
6. **完成的定義**：驗收條件逐條打勾＋fresh-context subagent 驗過＋產物在使用者拿得到的位置。
   **沒實跑過不算完成**。涉及部署者另需 push＋部署 workflow 成功＋**線上驗證本次變更的具體內容**
   （破快取 raw URL／curl／瀏覽器實查），只寫在本機不算完成。
7. **push 前**：先 `git fetch`；`git log --oneline main..origin/main` 非空必須先看內容（訊息／
   時間戳／diff）。一般 push → rebase 整合，嚴禁直接覆蓋；force push 前若 origin 領先的 commit
   是真實新工作 → 停下來問，授權「這次 force push」不等於授權蓋掉 origin 所有領先 commit。
8. **新指標／訊號先問有沒有回測依據**，沒有就先驗證再上線。市場內容可做情境判讀與多空
   因素分析，可研判市場與大眾情緒對該數值或新聞的可能反應，並可提供具體個股／標的的
   買賣建議與進出點位；以上均須附依據、區分事實與推論，並標明屬 AI 研判而非保證。
9. **語言**：對話與文件用繁體中文；程式碼註解可中文，identifier 用英文。

> 判準細則、派工模板、教訓簿見 `shihpc/claude-harness`（private）。雲端 session 需 add_repo 才讀得到。
<!-- CANON:END v1 -->

「股市雷達 · Dashboard Hub」入口站。**純靜態、無建置流程**：站台內容只有一個
`index.html`（573 行，2026-09-07 實測），GitHub Pages 直接從 main root 服務。唯一的 workflow 是
`.github/workflows/canon.yml`，只守 CLAUDE.md 頂端的 CANON 區塊，不產出任何東西。
線上 https://shihpc.github.io/ 。

## 佈局

`index.html` 一檔到底（CSS/JS 內嵌），三段結構：

- `<head>` 門面 meta（:7-11）：`description`／`theme-color`（取 `--bg` 的 `#0b1120`）／
  📡 SVG data URI favicon／`preconnect` 到 Worker（:10）與 `raw.githubusercontent.com`（:11，供「我的異動」）
- 前端密碼門（:164-179；啟動判斷在 :552-570）
- `PROJECTS` 卡片陣列（:187-227）
- `ICONS` SVG 圖庫（:240-246）＋ `PALETTE` 漸層色盤（:230-237）＋ `renderCards()`（:248-266）
- 資料健康狀態列 `loadStatus()`（:296-347，含 `showStatusFail()` :282-294）：解鎖後才非同步抓
  `https://taiwan-flow-v2.shihpc.workers.dev/status`，依卡片 `statusId` 對應
  `sites[].id` 顯示「● 資料日 MM/DD」；fetch 失敗／逾時（8 秒）／非 2xx／JSON 不合形狀
  → 卡片不掛狀態列，改在 `#statusMsg`（:150）顯示一行中性灰
  「資料狀態：查詢失敗（未知）」（語意見「改動注意」第 5 條）
- **「我的異動」可收合區**（HTML `<section id="mychg">` :152-157、CSS :82-102、
  JS :349-550）：解鎖後掛在五張卡與 `#statusMsg` 之下，`unlock()` 呼叫 `loadMyChanges()`（:178）。
  細節見下方「我的異動」節。

## 五張卡（`index.html:187-227`）

| 卡片 | 連往 |
|------|------|
| 即時類股動態 | https://shihpc.github.io/taiwan-flow-live-v2/ |
| 盤後法人動態 | https://shihpc.github.io/taiwan-flows/ |
| 新聞晨報 | https://shihpc.github.io/taiwan-stock-news/ |
| 盤後分析 | https://shihpc.github.io/postmkt/ |
| 策略回測 | https://shihpc.github.io/taiwan-backtest/ （`index.html:220-226`，無 `statusId`，不顯示狀態列） |

## 我的異動（`index.html:349-550`，2026-09-07 批次三 #18）

解鎖後、五張卡之下的可收合區塊（`<details id="mychgBox">`），列出「我持有的股票今天有沒有
相對明顯的變化」，一眼看完再決定要不要進子站。

- **誠實原則（不可淡化）**：**這不是買賣訊號，只是相對變化提醒**；`MYCHG_LOT_TH`（±100 張，:378）與
  `MYCHG_CHG_PCT_TH`（±3%，:379）是**顯示用可調常數、無回測依據**（CANON 第 8 條）。畫面 `.note`
  倒數第二句（`loadMyChanges()` 內 `notes.push`，:545-546；最後一句是 `:547` 的資料源說明）**逐字**含
  `為顯示用可調常數、無回測依據,不是買賣訊號,只是相對變化提醒`（逗號同 index.html 原樣為半形）
  ——改門檻要連畫面文字一起改，不要改成「訊號／建議」語氣。
- **隱私（CANON 第 1 條、postmkt 隱私鐵則）**：持股只從同 origin `localStorage["pm_holdings"]`
  （postmkt 寫入，本站**唯讀**，格式 `[{c,sh,cost}]`）讀取，**絕不放進任何網路請求的 URL、
  header 或 body**。本節的外部請求全是**無參數的固定 GET**（下方三支 URL，內容與持股無關），
  比對完全在瀏覽器端做（Playwright 逐請求稽核 URL＋body＋headers，2026-09-07 實測零命中）。
- **資料源（2026-09-07 換掉原本只讀 `latest.json` 的做法）**：
  | 檔案 | 角色 | 原始／線上 gzip（2026-09-07 curl 實測 `content-length`） |
  |------|------|------|
  | `data/status.json` | 先取資料日（`sources.daily`，退回 `date`）組出 daily 檔名 | 500 B／**290 B** |
  | `data/daily/<YYYYMMDD>.json` | **主資料**，全市場逐檔 2650 列 | 256,209 B／**81,686 B** |
  | `data/latest.json` | **選配**，只補股名（daily 無股名欄），與主流程並行、失敗不影響 | 108,341 B／**19,333 B** |
  三支合計線上約 **99 KB**（`cache-control: max-age=300`）。daily 的 `cols` 含
  `code`／`chg_pct`／`f_net`／`t_net`＝畫面三欄剛好齊備（欄位語意見 `taiwan-flows/CLAUDE.md`
  daily schema）。**刻意不抓 `meta.json`（270KB）**：只為股名不值得，查不到就顯示代號。
- **覆蓋率：全市場 100%（2650/2650）**。舊版只讀 `latest.json`＝各榜前 30 名（ETF 榜前 20）、
  去重後僅 **294 檔（11.1%）**；以本模組門檻實測 2026-09-07 達門檻者 **911 檔，其中 687 檔（75.4%）
  在 `latest.json` 查不到**，卻被寫成「無顯著異動」——**缺資料卻呈現正常**，與 taiwan-flows
  `no_data` 靜默、入口站 `/status` 失敗靜默同型，已消滅。系統性盲區是槓桿／主動式 ETF 與中型股
  （例：`00637L` 當日外資 −954 張，舊版顯示為「無顯著異動」）。
- **三種狀態必須分得開（必修核心）**：三分在 `loadMyChanges()` :509-520、出句在 :538-548，
  `.note` 一行一句以 `<br>` 分行（:548），**三句不可合併成同一個字串**：
  1. **有資料且達門檻** → 列進表格（最多 `MYCHG_MAX_ROWS`＝5 檔，:380；超出寫「另 M 檔已達門檻但未列出」）
  2. **有資料但未達門檻** → 「其餘 N 檔未達門檻（當日資料查得到，只是變化不到門檻）」
  3. **代號不在當日 daily 檔** → 「M 檔在當日資料中查無此代號（已下市/停牌/代號有誤），不代表沒有異動」
     （`myChgDailyIndex()` :420-438 只為查得到的代號建 key，呼叫端據有無 key 三分）
  現在有全市場資料，第 3 類會很少，但仍**必須與第 2 類分開講**，否則使用者無從分辨。
- **無持股時整節維持 `hidden`**（`loadMyChanges()` 直接 return），入口與原本一模一樣。
- **開合狀態**存 `localStorage` key `hub_mychg_open`（`"1"`／`"0"`，**預設展開**），
  存取走既有 `lsGet`／`lsSet`（try/catch）。
- **失敗行為**：`status.json` 或 daily 任一支 fetch 8 秒逾時／非 2xx／JSON 不合形狀
  → `#mychgBody` 顯示「無法取得異動資料」，**不得靜默當成無異動**；`latest.json` 失敗只讓股名
  退回顯示代號，不影響主流程。三者皆**不影響五張卡與 `/status` 狀態列**（各自 try/catch）。
- **XSS**：本節新增標準 `esc()`（:384-386，逃 `&<>"'`），持股代號（使用者可控）與 `latest.json`
  股名全部過它才拼進 `innerHTML`（`.note` 也是逐句 `esc()` 後才用 `<br>` 串接）；連結 href 為
  `https://shihpc.github.io/postmkt/#tab=diag&code=`＋`encodeURIComponent(代號)`（postmkt 的
  hash 路由，該格式須維持可用，見 `postmkt/index.html` hash 路由註解）。
- 驗收（Playwright，2026-09-07 實測 **17/17 PASS、pageerror 零**）：無持股→整節不出現／三種狀態
  各自正確顯示且兩句文案分行／連結格式正確／`status.json` 500、daily 500、daily 逾時三種都顯示
  「無法取得異動資料」且五張卡與狀態列照常／`latest.json` 500 時股名退回代號／代號與股名含
  `<img onerror>` 皆不執行（以字面文字顯示）／收合狀態 reload 保留／所有 request 的 URL＋body＋
  headers 不含持股代號或 `pm_holdings`。另以 curl 取回的**真實線上三檔**重放實測（2026-09-07）：
  `2330` 與 `00637L` 達門檻列出（`00637L` 不在 `latest.json` → 名稱欄顯示代號）、`6949` 歸「未達門檻」、
  `9999` 歸「查無此代號」，四檔各自落在正確狀態。
  **沙箱限制**：本容器的 headless Chromium 無法直連外網，線上驗證走 curl 取檔＋`page.route` 重放真檔。

## 改動注意

1. **新增站台**＝在 `PROJECTS` 加一筆：`name`／`desc`／`url` 必填；`icon` 要對應
   `ICONS` 既有 key（新圖示先到 `ICONS`（:240-246）加一個 key → SVG path）；
   `color` 可省略，省略時由 `PALETTE` 依順序循環分配；`statusId` 可省略，
   填了才會對應 `/status` 顯示健康狀態列。
2. **既有卡的 `color` 是釘死的**：`index.html:201` 註解「綠(釘原色,不受新卡位移影響)」——
   五張卡都手動指定顏色，就是為了讓新卡插入時既有卡配色不位移。不要為了「統一」而拿掉。
3. **子站有回程連結硬編 `https://shihpc.github.io/`**（例：taiwan-stock-news）——
   若改動 Hub 網址，必須同步各子站的回程連結。
4. **密碼門**（:164-179）是前端 SHA-256 比對，註解自承「輕量遮罩,非真正安全」；
   改密碼＝重算 SHA-256 換掉 `PW_HASH` 那一行（:166）。
   **不要把雜湊值或密碼寫進任何文件、commit message 或對話輸出。**
5. **狀態列失敗顯示「查詢失敗（未知）」是刻意的，不要改回靜默**：`/status` 查不到
   （逾時／非 2xx／形狀不合／網路例外）時 `showStatusFail()` 在 `#statusMsg` 顯示中性灰
   「資料狀態：查詢失敗（未知）」——「未知」＝不知道資料好壞，**不是**資料異常，所以不用
   紅黃綠。`localStorage` key `hub_status_ok_at` 只在 `/status` 回應**形狀合格**時寫入
   ISO 時間（不代表各站資料綠燈），失敗時若讀得到就附「上次成功 MM/DD HH:MM」（瀏覽器
   本地時區）。存取 `localStorage` 全部 try/catch，被封鎖時只少那句、不影響顯示。

## 驗證方式

```bash
python -m http.server 8000   # 開 localhost:8000，過密碼門後確認五張卡渲染且連結可點
```

無測試、無 CI；改完 push 到 main 即由 GitHub Pages 上線。
