# CLAUDE.md - 眼底影像檢視系統 (Fundus PACS Viewer)

## 專案概述

**Fundus PACS** 是一個專門為眼底攝影（視網膜成像）設計的醫療影像檢視器。這是一個獨立的單檔案網頁應用程式，用於檢視和分析眼底影像，支援雙眼對比（同時顯示兩眼）。

**專案類型**：醫療影像檢視器（眼科）
**主要語言**：JavaScript (透過 CDN 的 React)
**架構**：獨立 HTML 檔案的單頁應用程式 (SPA)
**醫療背景**：PACS = 影像存檔與通訊系統 (Picture Archiving and Communication System)

## 儲存庫結構

```
FUNDUS_PACS/
├── FUNDUS_PACS.html    # 主應用程式（獨立的 React 應用程式）
├── README.md           # 基本專案說明（中文）
└── CLAUDE.md           # 本檔案 - AI 助理指南
```

### 關鍵檔案

- **FUNDUS_PACS.html**：包含所有 HTML、CSS 和 JavaScript 的完整獨立應用程式
  - 不需要建置流程
  - 可直接在瀏覽器中開啟
  - 所有相依套件透過 CDN 載入

## 技術堆疊

### 相依套件（基於 CDN）
- **React 18** (`react.development.js`, `react-dom.development.js`)
- **Tailwind CSS** (透過 CDN)
- **Babel Standalone** (瀏覽器中的 JSX 轉換)
- **Lucide React** (圖示函式庫, v0.468.0)

### 無建置系統
本專案刻意使用獨立 HTML 方式：
- 無 npm/package.json
- 無建置工具（webpack、vite 等）
- 無轉譯步驟
- 直接在瀏覽器執行

## 醫療影像慣例

### 檔案命名規則（重要）

應用程式期望眼底影像檔案遵循以下命名模式：

```
{病歷號}_{年月日}_{時分秒}_{類型}_{側別}_{序號}.{副檔名}
```

**範例**：`01424_20251202_080330_Color_R_001.jpg`

**組成部分**：
- `病歷號`：病歷號碼 (Medical Record Number, MRN)
- `年月日`：拍攝日期（8 位數字，YYYYMMDD）
- `時分秒`：拍攝時間（6 位數字，HHMMSS）
- `類型`：影像類型（例如 "Color"）
- `側別`：眼睛側別
  - `R` 或 `OD` = 右眼 (Oculus Dexter)
  - `L` 或 `OS` = 左眼 (Oculus Sinister)
- `序號`：影像序號（例如 001、002）
- `副檔名`：`.jpg`、`.jpeg` 或 `.png`

**解析邏輯** (FUNDUS_PACS.html:106-133)：
```javascript
const parseFilename = (filename) => {
    const parts = nameOnly.split('_');
    if (parts.length < 5) return null;

    const mrn = parts[0];
    const dateStr = parts[1]; // YYYYMMDD
    const timeStr = parts[2];
    const sideCode = parts[4];

    const side = (sideCode === 'R' || sideCode === 'OD') ? 'OD' : 'OS';
    return { mrn, date, side, ... };
};
```

### 醫療術語

- **OD** (Oculus Dexter)：右眼
- **OS** (Oculus Sinister)：左眼
- **OU** (Oculus Uterque)：雙眼
- **Fundus**：眼底，眼球內部與晶狀體相對的表面
- **Red-Free Imaging**：無紅光影像，使用綠光濾鏡突顯視網膜血管的成像技術

## 應用程式架構

### 元件結構

應用程式遵循標準的 React 元件階層：

```
App（主容器）
├── Sidebar（側邊欄）
│   ├── 檔案匯入按鈕
│   ├── 搜尋框
│   └── 依日期分組的檔案列表
├── Toolbar（工具列）
│   ├── 檢視模式切換（單張/分割）
│   ├── 工具選擇（移動/調整）
│   ├── 影像調整滑桿
│   └── 濾鏡控制（無紅光、反相）
└── Main Viewport（主檢視區）
    └── ImageCanvas（1-2 個實例，取決於檢視模式）
```

### 關鍵元件

#### 1. Sidebar 側邊欄 (FUNDUS_PACS.html:137-285)
**功能**：檔案瀏覽和病患搜尋
- 接受本機資料夾匯入
- 依病歷號 (MRN) 搜尋
- 依日期分組影像
- 支援雙擊開啟影像
- 雙擊日期標題開啟雙眼檢視 (OU)

#### 2. Toolbar 工具列 (FUNDUS_PACS.html:287-408)
**功能**：影像操作控制
- 檢視模式：單張 vs. 分割（雙眼）
- 啟用工具：移動 vs. 調整 (W/L)
- 亮度/對比度滑桿
- 無紅光濾鏡切換
- 反相濾鏡切換
- 縮放控制
- 重設按鈕

#### 3. ImageCanvas 影像畫布 (FUNDUS_PACS.html:410-526)
**功能**：個別影像顯示和互動
- 滑鼠滾輪縮放
- 移動工具：點擊拖曳移動影像
- 調整工具：垂直拖曳 = 亮度，水平拖曳 = 對比度
- 顯示病患中繼資料（病歷號、日期、側別）
- 處理遺失影像的佔位符

#### 4. App 主程式 (FUNDUS_PACS.html:528-670)
**功能**：應用程式狀態管理
- 管理檔案列表
- 控制檢視模式（單張/分割）
- 管理影像設定（雙眼共享）
- 處理檔案選擇邏輯

### 狀態管理

應用程式使用 React hooks 進行狀態管理：

```javascript
// 主應用程式狀態
const [allFiles, setAllFiles] = useState(DEFAULT_FILES);
const [selectedFiles, setSelectedFiles] = useState([]);
const [viewMode, setViewMode] = useState('single');
const [activeTool, setActiveTool] = useState('pan');

// 影像設定（雙眼共享）
const [imageSettings, setImageSettings] = useState({
    brightness: 100,
    contrast: 100,
    zoom: 1,
    invert: false,
    redFree: false
});
```

**重要**：在分割檢視時，影像設定會在兩眼之間同步。這是為了醫療比對目的而刻意設計的。

## 主要功能

### 1. 檔案匯入
- 透過 `<input webkitdirectory>` 瀏覽器資料夾選擇
- 篩選影像檔案（.jpg、.jpeg、.png）
- 自動解析檔名以提取中繼資料

### 2. 檢視模式
- **單張**：一次一張影像
- **分割 (OU)**：雙眼並排比對

### 3. 影像操作
- **縮放**：滑鼠滾輪或按鈕（10%-500%）
- **移動**：點擊拖曳（移動工具啟用時）
- **亮度/對比度**：點擊拖曳（調整工具）或滑桿
- **無紅光濾鏡**：數位綠色通道濾鏡（SVG 濾鏡）
- **反相**：顏色反轉

### 4. 智慧配對
在分割模式下選擇影像時：
- 自動尋找同日期的對側眼
- 顯示 OD（右眼）在左側，OS（左眼）在右側（標準醫療慣例）

### 5. 模擬資料
用於示範的預設模擬資料 (FUNDUS_PACS.html:87-101)：
- 3 位範例病患（病歷號：01424、09932、10234）
- 每位 3 個日期
- 每個日期 4 張影像（每眼 2 張）

## 開發工作流程

### 測試應用程式

1. **在瀏覽器中開啟**：
   ```bash
   # 直接開啟 HTML 檔案
   open FUNDUS_PACS.html
   # 或在 Linux 上
   xdg-open FUNDUS_PACS.html
   ```

2. **使用模擬資料測試**：
   - 在側邊欄搜尋 "01424"
   - 雙擊日期標題以開啟雙眼檢視
   - 測試影像控制

3. **使用實際資料測試**：
   - 點擊「載入本機資料夾」
   - 選擇包含正確命名眼底影像的資料夾
   - 驗證檔名解析是否正確運作

### 進行變更

#### 編輯應用程式

由於這是一個獨立的 HTML 檔案，所有變更都在 `FUNDUS_PACS.html` 中進行：

1. **JavaScript/React 程式碼**：位於 `<script type="text/babel">` 區塊（第 66-674 行）
2. **樣式**：
   - 內嵌 `<style>` 區塊（第 21-48 行）
   - JSX 中的 Tailwind 工具類別
3. **SVG 濾鏡**：在隱藏的 SVG 區塊中定義（第 53-64 行）

#### 檔案內的程式碼組織

```
FUNDUS_PACS.html
├── <head>
│   ├── Meta 標籤
│   ├── CDN 腳本匯入
│   └── 自訂 CSS
├── <body>
│   ├── <div id="root">（React 掛載點）
│   ├── SVG 濾鏡定義
│   └── <script type="text/babel">
│       ├── 匯入和設定
│       ├── 模擬資料生成
│       ├── 輔助函式（parseFilename）
│       ├── 元件：Sidebar
│       ├── 元件：Toolbar
│       ├── 元件：ImageCanvas
│       ├── 元件：App
│       └── ReactDOM.render()
└── </body>
```

### 新增功能

新增功能時，請遵循以下模式：

1. **將狀態新增至 App 元件**（如果它影響多個元件）
2. **將 props 向下傳遞**給子元件
3. **使用回呼函式**向上與父元件溝通
4. **在分割檢視模式下保持影像設定同步**

範例 - 新增濾鏡：
```javascript
// 1. 新增至 imageSettings 狀態
const [imageSettings, setImageSettings] = useState({
    // ... 現有設定
    newFilter: false  // 在此新增
});

// 2. 在 Toolbar 中新增 UI 控制
<button
    onClick={() => updateSetting('newFilter', !settings.newFilter)}
>
    New Filter
</button>

// 3. 在 ImageCanvas filterStyle 中套用
const filterStyle = {
    filter: `
        ...現有濾鏡...
        ${settings.newFilter ? 'sepia(1)' : ''}
    `,
    // ...
};
```

## 常見修改模式

### 變更影像處理

所有影像濾鏡都透過 `ImageCanvas` 元件中的 CSS 濾鏡套用：

```javascript
const filterStyle = {
    filter: `
        brightness(${settings.brightness}%)
        contrast(${settings.contrast}%)
        invert(${settings.invert ? 1 : 0})
        ${settings.redFree ? 'url(#red-free-filter)' : ''}
    `,
    // ...
};
```

**可用的 CSS 濾鏡**：
- `brightness()`、`contrast()`、`saturate()`
- `hue-rotate()`、`invert()`、`sepia()`
- `blur()`、`grayscale()`
- 透過 `url(#filter-id)` 的自訂 SVG 濾鏡

### 修改檔名解析

如果您的命名規則不同，請編輯 `parseFilename()` 函式（第 106-133 行）：

```javascript
const parseFilename = (filename) => {
    // 在此修改解析邏輯
    // 必須回傳包含以下屬性的物件：mrn, date, side, filename, id
};
```

### 變更預設設定

修改 App 元件中的初始狀態（第 535-541 行）：

```javascript
const [imageSettings, setImageSettings] = useState({
    brightness: 100,  // 在此變更預設值
    contrast: 100,
    zoom: 1,
    invert: false,
    redFree: false
});
```

## 安全性考量

### 醫療資料隱私

**重要**：本應用程式在瀏覽器中本地處理醫療影像。

- **無伺服器上傳**：所有處理都在客戶端進行
- **無資料持久化**：檔案透過 `FileReader` API 載入並儲存在記憶體中
- **清理物件 URL**：卸載時呼叫 `URL.revokeObjectURL()`
- **無外部 API 呼叫**：初始載入後應用程式完全離線執行

### HIPAA 合規性注意事項

雖然本應用程式不傳輸資料，但請考慮：
- 瀏覽器快取可能儲存影像
- 瀏覽器歷史記錄可能記錄檔名（包含病歷號）
- 螢幕截圖可能擷取 PHI（受保護的健康資訊）
- 建議使用私密/無痕模式處理敏感資料

## 瀏覽器相容性

### 測試過的瀏覽器
- Chrome/Edge 90+（建議）
- Firefox 88+
- Safari 14+

### 必要功能
- ES6+ JavaScript 支援
- CSS Grid
- Flexbox
- FileReader API
- Input `webkitdirectory` 屬性（用於資料夾選擇）

### 已知限制
- 較舊的瀏覽器不支援資料夾選擇
- 大型影像集（>100 張影像）可能導致效能問題
- 行動瀏覽器對大型影像的記憶體可能有限

## 鍵盤快捷鍵

目前尚未實作鍵盤快捷鍵。潛在的新增項目：

- `R`：重設所有設定
- `空白鍵`：在移動/調整工具之間切換
- `1/2`：在單張/分割檢視之間切換
- `+/-`：放大/縮小
- 方向鍵：移動影像

## 疑難排解

### 影像無法載入

1. **檢查檔名格式**：必須符合 `病歷號_日期_時間_類型_側別_序號.副檔名`
2. **驗證副檔名**：必須是 `.jpg`、`.jpeg` 或 `.png`
3. **檢查主控台**：開啟瀏覽器開發者工具查看解析錯誤

### 搜尋無法運作

- 模擬資料僅包含病歷號：01424、09932、10234
- 匯入實際檔案後，搜尋會依病歷號子字串比對篩選

### 雙眼檢視缺少對側眼

- 確保兩眼都存在且病歷號和日期相符
- 驗證側別代碼解析正確（R/OD vs L/OS）
- 檢查檔案是否來自同一拍攝時段（相同日期）

## 未來增強想法

### 潛在功能
- 鍵盤快捷鍵
- 套用目前設定的影像匯出
- 標註工具（標記、測量）
- 比對模式（同一眼，不同日期）
- DICOM 支援
- 影像儲存的伺服器整合
- 多使用者支援
- 影像品質指標
- 自動病變偵測（AI/ML）

### 架構改進
- 拆分為具有建置系統的模組化檔案
- 新增 TypeScript 以提供型別安全
- 實作適當的狀態管理（Redux/Zustand）
- 新增單元測試
- 漸進式網頁應用程式 (PWA) 支援
- 離線快取

## Git 工作流程

### 分支策略
- 在以 `claude/` 開頭的功能分支上工作
- 分支命名：`claude/claude-md-{session-id}`
- 永不直接提交至 main

### 提交訊息格式
```
<類型>: <簡短描述>

<可選的詳細描述>
```

**類型**：`feat`、`fix`、`docs`、`style`、`refactor`、`perf`、`test`

**範例**：
```
feat: add keyboard shortcuts for common operations
fix: correct bilateral pairing for mixed laterality codes
docs: update CLAUDE.md with deployment instructions
```

### 提交前

1. 至少在 2 個瀏覽器中測試
2. 驗證模擬資料仍正常運作
3. 檢查主控台錯誤
4. 確保 HTML 檔案有效

## 部署

### 託管選項

**選項 1：靜態託管**
- GitHub Pages
- Netlify
- Vercel
- 任何靜態檔案伺服器

**選項 2：本地網路**
```bash
# 簡易 Python 伺服器
python3 -m http.server 8000

# 在 http://localhost:8000/FUNDUS_PACS.html 存取
```

**選項 3：內部網路**
- 部署至醫院/診所內部網路
- 確保 HIPAA 合規措施
- 考慮存取控制

### 正式環境考量

正式環境部署前：

1. **切換至正式版 React 建置**：
   ```html
   <!-- 將開發版替換為正式版 -->
   <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
   <script src="https://unpkg.com/react-dom@18/umd/react.production.min.js"></script>
   ```

2. **移除模擬資料**或明確標示為示範

3. **新增免責聲明**關於醫療用途

4. **使用實際資料測試**來自目標環境

5. **記錄**預期的確切檔名規則

## AI 助理指南

### 修改此程式碼庫時

1. **始終先讀取完整檔案**再進行變更 - 這是單一檔案
2. **測試變更**，建議使用者在瀏覽器中開啟
3. **保留醫療慣例**（OD/OS、OU 術語）
4. **維護檔名解析相容性** - 此處的變更會影響所有使用者
5. **在分割檢視模式下保持影像設定同步**
6. **不要增加建置複雜性** - 除非明確要求，否則維持獨立性質
7. **考慮醫療背景** - 這是診斷工具

### 程式碼風格偏好

- **React**：使用 hooks 的函式元件
- **命名**：JS 使用 camelCase，CSS 類別使用 kebab-case
- **註解**：解釋醫療/領域邏輯，而非顯而易見的程式碼
- **Tailwind**：使用工具類別，最小化自訂 CSS
- **檔案組織**：在 HTML 檔案中將邏輯區段保持在一起

### 醫療領域知識

處理此程式碼庫時，請理解：
- **右眼 (OD) 顯示在左側**（放射學慣例）
- **左眼 (OS) 顯示在右側**
- **無紅光影像**有助於視覺化視網膜神經纖維層
- **亮度/對比度**（醫療影像中的視窗/層級）對診斷至關重要
- **縮放和移動**對於細節檢查至關重要

### 詢問使用者的問題

進行重大變更前：
- 「您確切的檔名規則是什麼？」
- 「是否需要 DICOM 支援？」
- 「這應該保持為獨立檔案還是改用建置系統？」
- 「是否有需要遵循的特定醫療標準？」
- 「您的部署目標是什麼？（網頁伺服器、內部網路、離線）」

## 參考資料

### 醫療影像標準
- DICOM：醫學數位影像與通訊 (Digital Imaging and Communications in Medicine)
- PACS：影像存檔與通訊系統 (Picture Archiving and Communication System)
- HL7：第七層健康資訊交換協定 (Health Level Seven)，用於 EHR 整合

### 相關技術
- React 文件：https://react.dev
- Tailwind CSS：https://tailwindcss.com
- Lucide Icons：https://lucide.dev

## 聯絡與支援

這是一個開源專案。如有問題或疑問：
- 查看 README.md 以獲取基本資訊
- 閱讀此 CLAUDE.md 以獲取詳細文件
- 在回報問題前先使用模擬資料測試

---

**最後更新**：2025-12-05
**版本**：1.0
**維護者**：AI 助理與貢獻者
