---
name: travel-export-html
description: 將規劃好的旅遊行程輸出為可部署至 GitHub Pages 的單頁 HTML 檔案。
---

# travel-export-html

將已規劃好的旅遊行程輸出為單頁 HTML 檔案，可直接部署至 GitHub Pages。

## 觸發時機

當使用者說「輸出 HTML」、「產生網頁」、「export HTML」或類似指令時執行此 skill。

## 輸入

執行前確認以下資訊已備妥（若無則詢問）：

- 旅遊目的地與日期區間
- 完整每日行程（景點、餐廳、交通）
- 費用總覽數據
- 旅行主題語（若無則自動生成一句）

## 輸出規格

生成**完整單一 HTML 檔案**（self-contained），命名為 `trip-{目的地}-{年月}.html`。

---

### 設計風格

- 色調：米白 `#FAF7F2`、珊瑚橘 `#E8855A`、深棕 `#3D2B1F`、淺棕 `#C4A882`
- 繁體中文字體：Google Fonts `Noto Sans TC`
- 英文/數字字體：Google Fonts `Playfair Display`（標題）、`Inter`（內文）
- 整體基調：現代簡約、溫馨浪漫，情侶旅遊感
- 響應式設計（RWD），手機優先

---

### 動畫規格（iOS 質感風格）

**核心原則**：所有動畫必須感覺自然、有物理重量感，參考 iOS 的 spring animation 與 UIKit 慣例。

#### Easing 曲線
- 標準進場：`cubic-bezier(0.34, 1.56, 0.64, 1)` — 帶有輕微 overshoot 的彈性感（iOS spring）
- 離場 / 淡出：`cubic-bezier(0.4, 0, 1, 1)` — 快速加速離開
- 位移滑動：`cubic-bezier(0.25, 0.46, 0.45, 0.94)` — easeOutQuart，流暢自然
- 顏色 / 陰影過渡：`ease` 即可，duration 200ms

#### 頁面載入動畫（Staggered Entrance）
```
1. Hero 背景：從下方 30px + 透明度 0 → 原位 + 透明度 1，duration 700ms，delay 0ms
2. Hero 標題：從下方 20px 進場，duration 600ms，delay 150ms
3. Hero 副標：duration 500ms，delay 280ms
4. 概覽卡片：每張依序 delay +80ms（第1張 400ms，第2張 480ms...）
5. 各區塊（行程、費用、小貼士）：使用 IntersectionObserver，進入視窗才觸發進場
```

#### Scroll 進場（IntersectionObserver）
- 未進入視窗：`opacity: 0; transform: translateY(24px)`
- 進入視窗後：加上 class `.visible`，transition 到 `opacity: 1; translateY(0)`
- duration：500ms，easing：`cubic-bezier(0.25, 0.46, 0.45, 0.94)`
- `threshold: 0.15`，`rootMargin: '0px 0px -40px 0px'`（提早一點觸發）

#### Tab 切換動畫
- 舊內容：`opacity: 1 → 0`，`translateX(0 → -16px)`，duration 180ms
- 新內容：`opacity: 0 → 1`，`translateX(16px → 0)`，duration 220ms，delay 180ms（等舊內容離場）
- Tab 指示器（active underline）：`transform: translateX()`，duration 280ms，spring easing

#### 互動回饋（Micro-interactions）
- 按鈕 hover：`transform: translateY(-2px)`，`box-shadow` 加深，duration 200ms
- 按鈕 active（按下）：`transform: scale(0.96)`，duration 100ms — 模擬 iOS 觸覺回饋
- 卡片 hover：`transform: translateY(-4px)`，`box-shadow: 0 12px 32px rgba(0,0,0,0.12)`，duration 250ms
- Google Maps 按鈕：hover 時背景色過渡 + 輕微放大 `scale(1.03)`

#### Hero 背景效果
- 漸層背景搭配極緩慢的 `@keyframes` 色相漂移（hue-rotate），duration 12s，`infinite alternate`
- 或：使用 CSS mesh gradient 靜態漸層，不動畫，保持優雅

#### 費用長條圖動畫
- 頁面載入或區塊進入視窗時，長條從 `width: 0` 延伸至目標寬度
- duration：800ms，easing：`cubic-bezier(0.25, 0.46, 0.45, 0.94)`
- 各條依序 stagger +60ms

#### 效能規範
- 所有動畫只使用 `transform` 和 `opacity`（不動 layout 屬性，避免 reflow）
- 加上 `will-change: transform, opacity`（僅針對有動畫的元素）
- 尊重使用者設定：加入 `@media (prefers-reduced-motion: reduce)` 區塊，關閉所有動畫，改為即時顯示

---

### 頁面結構

#### 1. Hero 區塊
```
背景：漸層或純色（珊瑚橘調）
- 旅遊標題（目的地 + 年份）
- 日期區間（格式：YYYY/MM/DD - YYYY/MM/DD）
- 旅行主題語（一句話，斜體）
- 天數 badge
```

#### 2. 旅行概覽卡片列（橫排 4 格）
```
📅 天數  👫 人數  💰 預算總計  ✈️ 主要交通
```

#### 3. 每日行程（Tab 切換）
- Tab 標籤：Day 1、Day 2 ...（點擊切換，純 CSS 或少量 JS）
- 每個 Tab 內容：
  - 日期標題 + 星期
  - 時間軸（Timeline）：上午 / 下午 / 晚上
    - 每個時段列出景點：emoji 圖示 + 景點名稱 + [地圖] 按鈕
    - 交通資訊（小字灰色）
    - 備註（開放時間、票價）
  - 餐飲卡片區（獨立區塊）：
    - 早餐 / 午餐 / 晚餐 / 甜點，各自一張小卡
    - 含店名、風格描述、人均預算
  - 當日費用小計條

#### 4. 費用總覽
- 視覺化表格：項目 / 金額 / 佔比
- 使用純 CSS 長條圖呈現各項目比例（不依賴外部圖表庫）
- 顯示幣別，外幣同時標示台幣參考值

#### 5. 旅遊小貼士
- 當地注意事項（3-5 點）
- 推薦 App（Google Maps、當地交通 App 等）
- 緊急電話（當地警察、救護、台灣外交部急難救助）

#### 6. Footer
```
製作日期：YYYY/MM/DD　|　由 Claude AI 協助規劃　|　奕承 & 沛書 的旅行
```

---

### 技術規格

- 所有 CSS 寫在 `<style>` 區塊內（inline）
- JavaScript 寫在 `<style>` 後的 `<script>` 區塊，用於：Tab 切換、IntersectionObserver scroll 動畫
- 允許使用 CDN：
  - Google Fonts（字體）
  - 不引入其他外部 JS 庫，所有動畫以原生 CSS + 少量 Vanilla JS 實作（保持輕量、無依賴）
- Google Maps 連結格式：`https://maps.google.com/?q=景點名稱+城市`
- 所有外部連結加 `target="_blank" rel="noopener noreferrer"`
- 加入 `@media print` 樣式：隱藏 Tab 切換按鈕，所有 Day 展開列印

---

### 輸出後動作

1. 將 HTML 內容完整輸出到對話（使用 code block，lang 標記為 `html`）
2. 提示使用者：
   - 存檔為 `trip-{目的地}-{年月}.html`
   - 上傳至 GitHub repo，在 Settings > Pages 啟用 GitHub Pages
   - 分享連結給同行夥伴
