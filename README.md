# 貸款轉貸提案計算器 — 交接說明

## 一、專案簡介
單一檔案的網頁工具（`index.html`），給貸款業務員製作「轉貸提案」用。
輸入兩個貸款方案（現況 vs 轉貸後），自動算出月付金、總利息、收費、回本時間，
並可一鍵匯出 PDF / 圖片給客戶。全部用原生 HTML + CSS + JavaScript，無框架。

## 二、技術棧 / 依賴
- 純 HTML / CSS / vanilla JS，無 build 流程，雙擊即可在瀏覽器開啟。
- 外部 CDN（匯出用）：
  - `html2canvas` 1.4.1 — 把 DOM 截成圖
  - `jspdf` 2.5.1 — 產生 PDF
- 因為用到 CDN，匯出功能需在「有網路的瀏覽器」環境執行。

## 三、檔案結構
- `index.html` — 主程式（= `loan_proposal.html`，兩份內容相同，`index.html` 供 GitHub Pages 根目錄用）
- 單檔內分三區：`<style>` 樣式 / `<body>` 版面 / `<script>` 邏輯。

## 四、資料模型（全域 `state`）
```js
state = {
  loan1: { principal, periods, grace, stages: [{ fromMonth, rate }] },
  loan2: { principal, periods, grace, stages: [{ fromMonth, rate }] },
  svc:   { mode: 'percent'|'amount', percent, amount },  // 轉貸服務費
  adv:   { mode: 'percent'|'amount', percent, amount }   // 代墊費用
}
```
- `principal` 元、`periods` 期數（月）、`grace` 寬限期（月，0 = 無）。
- `stages` 為分段利率，第一段 `fromMonth` 固定為 1；第 2~4 段為選填。
- 費用 `svc`/`adv` 可選「按 %」或「直接填金額」，未填（算出為 0）則整個費用與回本區塊不顯示。

## 五、核心函式
| 函式 | 用途 |
|------|------|
| `payment(bal, annualRate, n)` | 本息平均攤還公式，回傳單段月付金 |
| `calcStaged(principal, periods, stages, graceMonths)` | 主計算引擎，逐月模擬，回傳 segments、總利息、總繳款等 |
| `feeAmount(fee, base)` | 依 mode 算出費用金額（% × base 或直接金額） |
| `ratesLabel(loan)` | 產生利率顯示字串（單段或多段） |
| `renderStages(loanKey, containerId)` | 重建某方案的分段利率輸入欄（新增/移除時呼叫） |
| `render()` | 重算並重繪 `#proposal` 輸出區（每次輸入變動都呼叫） |
| `capture()` / 匯出按鈕 | html2canvas 截圖 → 下載 PNG 或拼成 A4 PDF |

## 六、計算邏輯（`calcStaged` 重點）
逐月（month 1 → periods）模擬，每月依 `rateAt(m)` 取得當月利率：
1. **寬限期內**（m ≤ graceMonths）：只繳息，`pay = balance × 月利率`，本金不動。
2. **寬限期後**：正常攤還。當「利率變動」或「剛結束寬限期」時，
   用「當下本金餘額 + 剩餘期數」重算月付金（符合台灣銀行分段重算規則）。
3. 連續且金額相同的月份會合併成一個 `segment`，供「月付金階段明細」顯示。
- 回傳：`firstPayment`（初期月付）、`afterGracePayment`（寬限期滿後月付）、
  `totalInterest`、`totalPayments`、`segments`、`hasGrace`、`multiSegment`。
- 三種情境（本息攤還 / 多段利率 / 寬限期）可任意疊加。

## 七、渲染與事件流
- 固定輸入欄（金額、期數、寬限期、第一段利率、費用）用 `bind(id, fn)` 綁 `input` 事件 → 更新 state → `render()`。
- `render()` 只重繪輸出區 `#proposal`，**不動輸入欄**，所以打字不會跳掉焦點。
- 分段利率為動態數量，由 `renderStages()` 重建（在按「新增/移除」時才呼叫）。

## 八、已知限制 / 注意事項
- 月付金以「本息平均攤還法」計算；與部分機構（如新鑫）實際帳單可能有小數差，
  建議客戶提案以實際帳單核對。畫面底部已有免責聲明。
- 匯出只截 `#proposal` 區塊（不含輸入欄與按鈕）。
- 無資料儲存（重新整理即清空），目前不需後端。

## 九、後續可擴充方向（建議）
- 攤銷表完整明細（目前主畫面以 segment 摘要呈現，可加可展開的逐期表）。
- 多方案比較（>2 個方案）。
- 提案可填入「客戶姓名 / 業務員 / 日期」抬頭。
- 匯入/匯出 JSON 方案設定，方便存檔重用。
- 利率輸入驗證與錯誤提示。

## 十、部署（GitHub Pages）
- repo 命名 `stanfordcan.github.io`、放 `index.html`、設為 Public。
- Settings → Pages → Source 選 `main` / root → Save，數分鐘後即上線。

---
給 Claude Code 的一句話交接：
> 這是一支單檔 HTML 的貸款轉貸提案計算器（vanilla JS，html2canvas + jsPDF 匯出）。
> 主邏輯在 `<script>` 內，計算引擎是 `calcStaged()`，畫面由 `render()` 重繪。
> 請在不破壞現有「多段利率 / 寬限期 / 費用選填 / 匯出」功能的前提下繼續開發。
