# mission-timer 設計規範

單頁任務計時器。只有一個 `index.html`，HTML／CSS／JS 全部內嵌，無建置流程、無外部套件依賴，開啟即用。

## 檔案結構

```
mission-timer/
├── index.html   ← 唯一的程式檔
├── CLAUDE.md    ← 這個專案的硬性規則（每次進來都要遵守）
├── DESIGN.md    ← 本檔案
├── .gitignore   ← 只忽略 .vercel
└── .vercel/     ← Vercel CLI 本機專案連結（不進版控）
```

## 部署鏈路

Git repo（[github.com/vaiskalivuan/mission-timer](https://github.com/vaiskalivuan/mission-timer)）已跟 Vercel 專案串接，push 到 `main` 會自動部署到 [mission-timer.vercel.app](https://mission-timer.vercel.app)。

Commit 慣例：一個功能一個 commit，訊息用英文寫「做了什麼＋為什麼」。

## 樣式上的既定做法

### 色彩系統
CSS 變數集中在 `:root`。目前四色：
- `--accent`：主色橘，平時用在「發射」按鈕、進度條、火箭機身
- `--accent-dim`：主色的暗版，用於禁用態
- `--gray` / `--gray-dim`：次要文字、灰階按鈕
- `--danger`：紅色警示，倒數剩 5 分鐘時的「快來不及」狀態

新增顏色照這個模式加變數，不要在規則裡直接寫死色碼。

### 字體
兩種字族分工明確：
- 等寬字體（`SF Mono` 系列）：倒數數字、日出日落資訊等「數字/時間類」顯示
- 非等寬（`-apple-system` 系列）：按鈕文案等一般文字

### 背景：純 Canvas 手繪，不用圖片
`<canvas id="bg">` 用 2D context 畫星空 + 軌道星球：
- 星空：星星座標用固定種子的 seeded PRNG（`mulberry32`）在載入時算好存進陣列
- 星球：經緯線網格點陣（`LAT_STEP` / `LON_STEP`），左上光源做出明暗面、輪廓邊緣光、大氣輝光

**關鍵原則**：亂數只在載入時算一次存進陣列，resize 時只重新投影、不重新算亂數，確保畫面不會每次重繪就跳動。

### 狀態靠 CSS class 切換，不是行內動態樣式
例如「快來不及」狀態，是同一個 `warn` class 同時掛在三個元素（`timerDisplay`、`progressFill`、`rocket`）上，各元素自己在 CSS 裡定義 `.xxx.warn` 規則。JS 只用 `classList.toggle` 開關，顏色/動畫邏輯全部留在 CSS。

### 動畫尊重無障礙偏好
有 `@keyframes` 的地方都配一條 `@media (prefers-reduced-motion: reduce)` 關掉動畫。

### JS 結構
單一 IIFE，內部分三段，用註解隔開：
1. 背景：軌道視角星球
2. 計時器
3. 台北日出日落

寫法是 ES5 風格（`var`、`function` 宣告），無框架、無建置工具，純瀏覽器原生 JS。

### 外部資料一律做離線防呆
外部 API 呼叫要有逾時保護（`AbortController`）+ try/catch，失敗時給明確的文字狀態（目前是「離線」），不能讓畫面壞掉。

---

以上是既定做法的說明；**具體的硬性規則見同資料夾的 [CLAUDE.md](CLAUDE.md)**，兩者有衝突時以 CLAUDE.md 為準。
