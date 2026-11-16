# IxChariot WASM Telemetry & Topology Analyzer

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Engine](https://img.shields.io/badge/Engine-Pyodide%20WASM%20%2B%20Chart.js-informational)](#系統架構)
[![Status](https://img.shields.io/badge/Release-v3.10.0-success)](#版本歷程)

專為長天期（可達 30 天 / 2,592,000 秒）IxChariot 網路效能測試設計的客戶端分析工具。基於 WebAssembly (Pyodide) 與純瀏覽器端渲染技術，無需將測試檔案上傳至後端伺服器，直接在本地端完成二進位 `.tst` 逆向標籤解構、JSONC 規範讀寫、信號雙向濾波、LTTB 幾何降採樣，以及具備防重疊排版與手動拖曳功能的端點串流拓撲繪製。

---

## 核心工程特性

### 1. 本地端 WASM 訊號處理 (In-Browser DSP)
- **資料零外流**：完全透過 WebAssembly 在瀏覽器記憶體內執行二進位解析與數值矩陣運算，杜絕企業測試數據洩漏疑慮。
- **雙向中值滑動濾波 (Bidirectional Rolling Median Filter)**：
  針對長時老化測試中因作業系統進程排程凍結造成的瞬降（Negative Dips / Dropouts），以及網卡緩衝區批次 Flush 造成的虛假衝頂（Positive Spikes），提供可切換的抑制演算法，避免極端異常值干擾整體統計。
- **精確統計特徵量提取**：
  同步計算每條傳輸對（Pair）的 Peak（峰值）、RMS（均方根有效能量）、Avg（算術平均速率）與 Min（底線谷值）。

### 2. 30 天超長測試渲染最佳化 (LTTB Downsampling)
- **動態幾何採樣**：導入 **Largest-Triangle-Three-Buckets (LTTB)** 演算法，將數十萬至數百萬筆高密度量測點降採樣至 1,200 ~ 1,800 點區間，完整保留波形極值與視覺輪廓特徵。
- **流暢縮放互動**：結合 Chart.js Zoom/Pan 插件，支援日、時、分、秒級自適應座標軸刻度，縮放視圖皆在 16ms 內完成畫面重繪。

### 3. 可互動防碰撞拓撲架構圖 (Interactive Direct-Manipulation Topology)
- **二分扇出排版 (Bipartite Fan-out)**：主測端點（Hub / AP）固定於左側中央，附屬端點垂直對稱於右側，雙向流採用立方貝茲外弧分流。
- **梯次防重疊位移 (Staggered T-Offsetting)**：同路徑標籤藥丸（Pill Labels）自動錯開於曲線各參數位置，並輔以防穿透遮罩背景。
- **完全自由拖曳 (Free Drag & Drop)**：支援滑鼠自由按住端點圓圈或流量標籤進行任意拖曳定位，動態連線與箭頭 60 FPS 即時重算追蹤，並支援一鍵重置佈局。

### 4. 模組化工程控制項
- **工程視圖時間窗篩選**：以 `天/時/分/秒` 雙向聯動控制 Viewport，精確聚焦任一異常區間。
- **雙 Y 軸支援**：自動區分 5G/6G 高速頻寬軸與 2.4G 專屬軸，避免低速流被壓縮為直線。
- **視圖無損切片匯出**：支援將當前縮放視圖內的數據單獨切片匯出為 JSONC，或一鍵截取高解析度 PNG。
- **多主題與國際化**：內建暗黑（Dark Space）、工程純白（Clean White）與板岩灰（Slate Gray）三種色彩主題，介面具備繁體中文（台灣）與英文雙語切換。

---

## 支援資料格式與規範

| 格式 | 延伸檔名 | 說明 |
| :--- | :--- | :--- |
| **IxChariot Binary** | `.tst` | 支援讀取 Ganymede 標記檔頭，自動識別 `2G`、`5G`、`6G`、`56G_TX`、`56G_RX` 等串流標籤與測試時長。 |
| **開放 JSONC** | `.jsonc` / `.json` | 標準 JSON 格式附加 C-Style 註解支援，包含完整 `metadata` 描述與時間序列表列。 |
| **視圖切片 JSONC** | `*_slice_*.jsonc` | 僅包含目前視窗選定區間之採樣點與該視窗內獨立統計特徵值。 |

---

## 快速開始

### 單機使用 (免伺服器環境)
本工具為單一靜態 HTML 架構，直接下載專案主檔案即可於現代瀏覽器運行：
1. 下載或複製 `index.html`。
2. 使用 Chrome、Edge、Firefox 或 Safari 開啟該檔案。
3. 待右上角狀態指示徽章變更為 **`WASM 引擎就緒`** 後即可開始載入數據。

### 本地開發伺服器啟動 (可選)
若因瀏覽器跨域限制導致 CDN 載入阻擋，可透過輕量 HTTP 伺服器啟動：
```bash
# 使用 Python 啟動 HTTP 服務
python -m http.server 8080

# 或使用 Node.js http-server
npx http-server -p 8080
```
瀏覽器開啟 `http://localhost:8080`。

---

## 操作指南

1. **資料載入與模擬**：
   - 點擊 **`載入 .tst`** 或 **`載入 .jsonc`** 選擇測試檔案。
   - 亦可直接點擊 **`模擬 144 小時`** 或 **`模擬 30 天測試`** 驗證極限負載與渲染效能。
2. **訊號濾波切換**：
   - 若時序圖出現瞬間毛刺或單點掉零，點擊 **`排除極端突波/瞬降`** 啟動雙向滑動中值濾波，系統將同步更新特徵值表格與折線趨勢。
3. **視圖微調與拓撲佈局**：
   - 使用滑鼠滾輪可直接在時序圖上進行局部縮放與拖曳平移。
   - 在端點串流拓撲架構圖中，可直接以滑鼠拖動端點節點或 RMS 標籤藥丸；點擊 **`重置拓撲佈局`** 可恢復預設對稱排版。
4. **局部匯出與清空**：
   - 調整觀看時間範圍後，點擊 **`匯出目前範圍 JSONC`** 或 **`匯出視圖圖片 (PNG)`**。
   - 分析完畢後點擊紅色 **`清空數據`** 按鈕，即可釋放所有記憶體與畫布狀態。

---

## 軟體架構與相依組件

```text
┌────────────────────────────────────────────────────────┐
│                 Web Browser UI Layer                   │
│   - CSS Grid/Flexbox Layout                            │
│   - SVG Dynamic Topology Engine (Drag-and-Drop Core)   │
│   - Viewport Time Controllers (D/H/M/S)                │
└───────────▲────────────────────────────────▲───────────┘
            │                                │
┌───────────▼───────────┐        ┌───────────▼───────────┐
│      Chart.js 4.x     │        │     Pyodide (WASM)    │
│   - Time-series plot  │        │   - NumPy DSP Engine  │
│   - Zoom / Pan Plugin │        │   - LTTB Sampling     │
│   - Multi-Axis Engine │        │   - Median Filter     │
└───────────────────────┘        └───────────────────────┘
```

- **WebAssembly Engine**: [Pyodide v0.25.1](https://pyodide.org/)
- **Scientific Computation**: [NumPy](https://numpy.org/)
- **Data Visualization**: [Chart.js v4.4.1](https://www.chartjs.org/)
- **Interaction Controller**: [chartjs-plugin-zoom v2.0.1](https://www.chartjs.org/chartjs-plugin-zoom/) / [Hammer.js v2.0.8](https://hammerjs.github.io/)

---

## 著作權與開源授權聲明 (License)

本專案基於 **GNU General Public License v3.0 (GPLv3)** 條款開放原始碼。

```text
Author: startgo (startgo@yia.app)
License: GNU General Public License v3.0 (GPLv3)
```

### GPLv3 條款摘要：
- **商業使用**：允許在符合授權義務下進行整合使用。
- **修改與衍生**：允許自由修改與擴充代碼，但任何包含或修改本專案之衍生作品散布時，必須同樣基於 GPLv3 授權且公開原始碼。
- **無免責保證**：本軟體按「現狀」提供，不附帶任何形式之明示或默示擔保。詳細條款請參閱官方 [GNU GPLv3 授權全文](https://www.gnu.org/licenses/gpl-3.0.txt)。