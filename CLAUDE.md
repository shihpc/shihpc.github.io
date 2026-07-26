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
8. **新指標／訊號先問有沒有回測依據**，沒有就先驗證再上線；不做預測宣稱，只描述歷史統計
   傾向與局限。
9. **語言**：對話與文件用繁體中文；程式碼註解可中文，identifier 用英文。

> 判準細則、派工模板、教訓簿見 `shihpc/claude-harness`（private）。雲端 session 需 add_repo 才讀得到。
<!-- CANON:END v1 -->

「股市雷達 · Dashboard Hub」入口站。**純靜態、無建置流程**：站台內容只有一個
`index.html`（231 行），GitHub Pages 直接從 main root 服務。唯一的 workflow 是
`.github/workflows/canon.yml`，只守 CLAUDE.md 頂端的 CANON 區塊，不產出任何東西。
線上 https://shihpc.github.io/ 。

## 佈局

`index.html` 一檔到底（CSS/JS 內嵌），三段結構：

- 前端密碼門（:120-133）
- `PROJECTS` 卡片陣列（:141-170）
- `ICONS` SVG 圖庫（:182-189）＋ `PALETTE` 漸層色盤 ＋ `renderCards()`

## 四張卡（`index.html:141-170`）

| 卡片 | 連往 |
|------|------|
| 即時類股動態 | https://shihpc.github.io/taiwan-flow-live-v2/ |
| 盤後法人動態 | https://shihpc.github.io/taiwan-flows/ |
| 新聞晨報 | https://shihpc.github.io/taiwan-stock-news/ |
| 盤後分析 | https://shihpc.github.io/postmkt/ |

## 改動注意

1. **新增站台**＝在 `PROJECTS` 加一筆：`name`／`desc`／`url` 必填；`icon` 要對應
   `ICONS` 既有 key（新圖示先到 `ICONS`（:182-189）加一個 key → SVG path）；
   `color` 可省略，省略時由 `PALETTE` 依順序循環分配。
2. **既有卡的 `color` 是釘死的**：`index.html:154` 註解「綠(釘原色,不受新卡位移影響)」——
   四張卡都手動指定顏色，就是為了讓新卡插入時既有卡配色不位移。不要為了「統一」而拿掉。
3. **子站有回程連結硬編 `https://shihpc.github.io/`**（例：taiwan-stock-news）——
   若改動 Hub 網址，必須同步各子站的回程連結。
4. **密碼門**（:120-133）是前端 SHA-256 比對，註解自承「輕量遮罩,非真正安全」；
   改密碼＝重算 SHA-256 換掉 `PW_HASH` 那一行（:121）。
   **不要把雜湊值或密碼寫進任何文件、commit message 或對話輸出。**

## 驗證方式

```bash
python -m http.server 8000   # 開 localhost:8000，過密碼門後確認四張卡渲染且連結可點
```

無測試、無 CI；改完 push 到 main 即由 GitHub Pages 上線。
