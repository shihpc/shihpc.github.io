# CLAUDE.md — shihpc.github.io 接手速覽

<!-- CANON:BEGIN v1 -->
<!-- 唯一事實來源＝shihpc/claude-harness 的 CANON.md。以下區塊在六個 repo 的 CLAUDE.md 頂端
     有 byte-identical 逐字副本，由各 repo 的 .github/workflows/canon.yml 守門（比對 sha256）。
     改動流程：先改 claude-harness/CANON.md → 跑 tools/sync_canon.py 同步六份 → 更新守門 hash。
     不要只改單一 repo，CI 會擋下來。 -->

## 通用工作鐵律（六個 repo 逐字相同，勿單獨修改）

1. **機密**：token／金鑰只存在不受版控的本機設定或受控 secrets（`.env`／Actions secret／
   `wrangler secret`），絕不寫進會 commit 的檔案、log 或對話輸出。commit 前掃 staged 內容，
   **只回報檔名／行號／類型、不把可疑字串原文印出來**；`sk-ant-`／`ghp_`／`eyJ` 是線索不是全集。
2. **指揮官不下場**：掃 repo、通讀 >300 行的檔、一次讀 >3 個檔、查網頁研究、批次改檔、
   驗收改過的東西——這六類一律派 subagent，主對話只收結論＋`檔案:行號`；但 subagent 回報
   **不等於事實**，主對話為核對結論可直接查原始證據。雲端 session 的 subagent 派工（含第 3 條
   驗收）已獲常備授權，需要時直接派，不需逐次詢問。
3. **先寫驗收條件再動手**：動手前先寫下目標專案完整路徑＋怎樣算完成＋怎麼驗。修改者先自測，
   再派 fresh-context subagent 驗收——**改東西的 agent（含主對話自己）不得擔任驗收者**。
   驗收要綁**確切 commit／產物版本**；驗收後又改動，受影響部分要重驗。
4. **不確定不亂說**：陳述事實（尤其技術細節、數字、外部服務的限制與行為）要嘛附佐證（官方
   文件、實測、`檔案:行號`），要嘛明說「這點我不確定，需要查證」，不可憑印象當確定講。區分
   「已驗證事實」與「推測」，推測要標明；**`檔案:行號` 只證明程式這樣寫，不證明線上這樣跑**。
5. **一次只做一件事**：聚焦一個明確目標，完成該目標必要的修改、測試與整合；不擅自加入無關
   重構或延伸功能。範圍外問題簡短記錄、不自行擴張任務。
6. **完成的定義**：驗收條件逐條打勾＋fresh-context subagent 驗過＋產物在使用者拿得到的位置，
   並明示已完成與未完成；**可執行的東西沒實跑過不算完成**（純文件交付以內容與結構檢查為準）。
   涉及部署者另需 push＋部署 workflow 成功＋在**實際服務的位置**驗證本次變更（線上頁面／API／
   資料時戳）——**raw URL 只證明原始碼進了 repo，不證明線上跑的是該版本**，200 也不等於功能正確。
7. **push 前**：先確認目前分支與推送目標，`git fetch` 後檢查遠端是否領先，非空必須先看內容
   （訊息／時間戳／diff）。一般 push → rebase 整合（本專案既定政策），嚴禁直接覆蓋；force push
   前若遠端領先的 commit 是真實新工作 → 停下來問，且一律用 `--force-with-lease=<ref>:<預期 SHA>`；
   授權「這次 force push」不等於授權蓋掉遠端所有領先 commit。
8. **新指標／訊號若會影響投資方向、候選排序、進出場或風險判定，先問有沒有回測依據**，沒有就
   先驗證再上線；純描述性顯示（欄位、日期、圖示）只需驗算式正確。市場內容可做情境判讀與多空
   因素分析，可研判市場與大眾情緒對該數值或新聞的可能反應，並可提供具體個股／標的的買賣建議
   與進出點位；以上均須附依據、區分事實與推論，並標明屬 AI 研判而非保證。
9. **語言**：對話與文件用繁體中文；程式碼註解可中文，identifier 用英文；外部原文、API 名稱、
   指令與錯誤訊息保留原樣。

> 判準細則、派工模板、教訓簿見 `shihpc/claude-harness`（private）。雲端 session 需 add_repo 才讀得到。
<!-- CANON:END v1 -->

「股市雷達 · Dashboard Hub」入口站。**純靜態、無建置流程**：站台內容只有一個
`index.html`（348 行，2026-09-09 `wc -l` 實測），GitHub Pages 直接從 main root 服務。唯一的 workflow 是
`.github/workflows/canon.yml`，只守 CLAUDE.md 頂端的 CANON 區塊，不產出任何東西。
線上 https://shihpc.github.io/ 。

## 佈局

`index.html` 一檔到底（CSS/JS 內嵌），**六段結構**（2026-09-09 更正：原寫「三段」但其下一直是 6 個項目，是加項目時漏改的計數）：

- `<head>` 門面 meta（grep `name="description"`／`name="theme-color"`／`rel="icon"`／
  `rel="preconnect"`，四行相鄰）：`description`／`theme-color`（取 `--bg` 的 `#0b1120`）／
  📡 SVG data URI favicon／`preconnect` 到 Worker。**`raw.githubusercontent.com` 的
  preconnect 已於 2026-09-09 隨「我的異動」一併移除**——它只服務那一段，`/status` 走 Worker
  是另一條；日後若又有需求要自己加回來。
- 前端密碼門：HTML 是 grep `<div id="gate">` 那個區塊，JS 是 grep `/* ===== 密碼門` 起至
  `function unlock` 結尾止；啟動判斷在 grep `/* ===== 啟動` 起的區塊（檔案最末）
- `PROJECTS` 卡片陣列（grep `const PROJECTS`）
- `ICONS` SVG 圖庫（grep `const ICONS`）＋ `PALETTE` 漸層色盤（grep `const PALETTE`）
  ＋ `renderCards()`（grep `function renderCards`）
- 資料健康狀態列 `loadStatus()`（grep `function loadStatus`，含 `showStatusFail()`／
  grep `function showStatusFail`）：解鎖後才非同步抓
  `https://taiwan-flow-v2.shihpc.workers.dev/status`，依卡片 `statusId` 對應
  `sites[].id` 顯示「● 資料日 MM/DD」；fetch 失敗／逾時（8 秒）／非 2xx／JSON 不合形狀
  → 卡片不掛狀態列，改在 `#statusMsg`（grep `id="statusMsg"`）顯示一行中性灰
  「資料狀態：查詢失敗（未知）」（語意見「改動注意」第 5 條）
- **「持股異動」搬遷提示列** `<p id="moved">`（grep `<p id="moved">`；CSS 是 grep
  `#moved{margin-top` 起的三條規則，含 `#moved a:hover`，其上兩行為說明註解）：一行靜態連結，指到
  `https://shihpc.github.io/postmkt/#tab=mychg`。**純 HTML、無 JS、無 fetch**，由
  `body.locked #moved{display:none}`（grep `body.locked`，全檔唯一一行）與五張卡一起藏在密碼門後。
  見下方「我的異動（已搬走）」節。

## 五張卡（`index.html` grep `const PROJECTS`）

| 卡片 | 連往 |
|------|------|
| 即時類股動態 | https://shihpc.github.io/taiwan-flow-live-v2/ |
| 盤後法人動態 | https://shihpc.github.io/taiwan-flows/ |
| 新聞晨報 | https://shihpc.github.io/taiwan-stock-news/ |
| 盤後分析 | https://shihpc.github.io/postmkt/ |
| 策略回測 | https://shihpc.github.io/taiwan-backtest/ （`PROJECTS` 內最後一筆；**2026-09-07 起補上 `statusId:"backtest"`**——`/status` 的 backtest 站由 taiwan-flow-live-v2 提供，`loadStatus()` 對缺站本來就 `return` 靜默不掛狀態列，故先加不會壞） |

## 我的異動（已於 2026-09-09 移除，搬去 postmkt）

2026-09-07 批次三 #18 在入口站加的可收合區塊「我的異動」（commit `ca4ca38`＋`d0aa7ad`），
**已於 2026-09-09 整段從本站移除**，功能搬到 **postmkt 的第 14 個 tab「持股異動」**
（`postmkt/index.html`，tab id `mychg`）。

- **搬家原因**：入口站那張表有 **6 欄**（代號／名稱／資料日／外資／投信／漲跌%），
  手機上只看得到前 2 欄，最重要的三個數字全在畫面外。postmkt 改用 **4 欄**版面，
  375px 實測四欄全部可見。
- **本站移除了什麼**：CSS `#mychg` 全部規則、HTML `<section id="mychg">`、`unlock()` 裡的
  `loadMyChanges()` 呼叫、JS 整段（`HOLD_KEY`／`MYCHG_*` 常數／`esc`／`myHoldings`／
  `myChgFetchJson`／`myChgDailyIndex`／`myChgNameIndex`／`myChgSignificant`／`myChgNum`／
  `loadMyChanges`），以及 `raw.githubusercontent.com` 的 `preconnect`（加它的唯一理由就是這一段）。
  共 −234／+8 行，`index.html` 574 → 348 行
  （`git diff --numstat a30d5fa^ a30d5fa -- index.html` 與 `wc -l` 實測）。
- **刻意保留**：`lsGet`／`lsSet`（雖然定義在 `loadStatus` 區塊裡，但 `STATUS_OK_KEY`
  也在用，**不是** mychg 專屬）、`loadStatus()`／`showStatusFail()`／`PROJECTS`／
  `renderCards()`／密碼門，全部行為不變。
- **⚠ 本站目前沒有任何 HTML 逃逸函式**：原 `esc()` 定義在 mychg 區塊內、全檔只被它用到，
  已一併刪除。現行程式碼**沒有任何把外部字串拼進 `innerHTML` 的路徑**——`renderCards()`
  的 `innerHTML` 只放程式內字面量（漸層色碼、`ICONS` 的 SVG path），`loadStatus()` 一律
  走 `document.createTextNode`／`className`／`.title`。**日後若要新增 `innerHTML` 路徑，
  必須自己補一支 `esc()`**（逃 `&<>"'`），不要以為站上已經有一支。
- **`localStorage` key `hub_mychg_open`（開合狀態）已成孤兒**：程式不再讀寫它，
  舊使用者瀏覽器裡殘留的值無害，**刻意不寫清除碼**（為了刪一個 key 而加開機邏輯不划算）。
- **入口指引**：五張卡下方新增一行靜態提示 `<p id="moved">`（見「佈局」節），
  「盤後分析」卡的 `desc` 也補上「· 持股異動」。**深連結格式
  `https://shihpc.github.io/postmkt/#tab=mychg`**——postmkt 的 `HASH_TABS` 由 `TABS`
  自動生成，改動 postmkt 的 tab id 會讓這條連結失效。
- 隱私面因此**變乾淨**：本站現在完全不讀 `localStorage["pm_holdings"]`，
  持股資料只留在 postmkt 同 origin 內。

## 改動注意

1. **新增站台**＝在 `PROJECTS` 加一筆：`name`／`desc`／`url` 必填；`icon` 要對應
   `ICONS` 既有 key（新圖示先到 `ICONS`（grep `const ICONS`）加一個 key → SVG path）；
   `color` 可省略，省略時由 `PALETTE` 依順序循環分配；`statusId` 可省略，
   填了才會對應 `/status` 顯示健康狀態列。
2. **既有卡的 `color` 是釘死的**：`index.html` grep `釘原色` 那行註解「綠(釘原色,不受新卡位移影響)」——
   五張卡都手動指定顏色，就是為了讓新卡插入時既有卡配色不位移。不要為了「統一」而拿掉。
3. **子站有回程連結硬編 `https://shihpc.github.io/`**（例：taiwan-stock-news）——
   若改動 Hub 網址，必須同步各子站的回程連結。
4. **密碼門**（grep `/* ===== 密碼門`）是前端 SHA-256 比對，註解自承「輕量遮罩,非真正安全」；
   改密碼＝重算 SHA-256 換掉 `PW_HASH` 那一行（grep `const PW_HASH`）。
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
**沙箱限制**：本容器的 headless Chromium 連不到外網，`/status` 一律用 Playwright `page.route`
餵 fixture 驗證（2026-09-09 移除「我的異動」的驗收即如此做：五張卡／狀態列／`#mychg` 不存在／
console 與 pageerror 零／375・390・1280 三寬度不水平溢出／`#moved` 連結 href 正確，全數通過）。
