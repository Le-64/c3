# 迪卡儂每日報表系統

## 專案簡介

這是一套專為迪卡儂門市夥伴設計的每日報表系統，取代紙本或純文字的交班記錄。夥伴只需開啟手機瀏覽器，不需安裝任何 App，即可在任何地點填寫當日業績、例行事務、晨會佈達及交接事項，所有資料即時同步給全體夥伴。

---

## 系統架構

```
夥伴的手機 / 電腦瀏覽器
        │
        ▼
GitHub Pages（免費靜態網頁托管）
  └── index.html   → 主頁（夥伴每日使用）
  └── admin.html   → 後台設定（主管專用）
        │
        ▼（讀寫資料）
Firebase Realtime Database（免費雲端資料庫）
  └── /config      → 後台目標設定
  └── /days        → 每日填寫資料
  └── /handovers   → 交接項目
```

**GitHub Pages** 負責存放網頁程式本身，**Firebase** 負責存放所有夥伴填寫的資料。兩者都是免費服務，資料永久保留，無需架設實體伺服器。

---

## 檔案結構

```
decathlon-daily/
│
├── docs/                    ← 正在使用的版本（GitHub Pages 部署來源）
│   ├── index.html           ← 主頁（夥伴每日操作介面）
│   └── admin.html           ← 後台設定頁（主管專用）
│
├── public/                  ← 舊版備用（搭配本機 server 使用，已停用）
│   ├── index.html
│   └── admin.html
│
├── server.js                ← 舊版本機伺服器（已停用）
├── start.sh                 ← 舊版一鍵啟動腳本（已停用）
├── package.json             ← Node.js 套件設定（舊版用，已停用）
├── package-lock.json        ← 自動產生，無需手動修改
├── data/
│   └── data.json            ← 舊版本機資料庫（已停用）
├── node_modules/            ← 舊版套件程式碼（自動產生，無需手動修改）
└── README.md                ← 本文件
```

> 目前實際運作的只有 `docs/` 資料夾內的兩個 HTML 檔案。

---

## 功能說明

### 主頁 `index.html`（夥伴每日使用）

| 功能 | 說明 |
|------|------|
| 日期導航 | 切換查看任意日期的紀錄，可回顧歷史或預填明日 |
| 部門目標 | 可直接輸入修改，即時同步給所有夥伴 |
| 今日業績 | 填入後自動計算達成率，顏色區分（綠／橘／紅） |
| Q/T（MTD）| 填入當日指標數值，旁邊顯示後台設定的目標 |
| NPS（MTD）| 同上 |
| 部門TO成長率（MTD）| 同上 |
| 例行事務 | 六項固定任務，✓ 已完成 / ✗ 未完成 / 空白 未操作 |
| 晨會佈達及 Action | 自由文字欄位，全體夥伴共同編輯 |
| 待辦交接 | 顯示前幾天尚未完成的交接事項，打勾即消失 |
| 今日新增交接 | 輸入要交給下一班的事項，明天起出現在待辦交接 |
| 自動儲存 | 所有欄位修改後約 1.2 秒自動儲存，無需手動按存檔 |

### 後台 `admin.html`（主管專用）

| 功能 | 說明 |
|------|------|
| 密碼保護 | 預設密碼 `dkt2024`，可直接修改 HTML 更換 |
| Q/T（MTD）目標 | 設定後顯示於主頁指標旁的藍色標籤 |
| NPS（MTD）目標 | 同上 |
| 部門TO成長率（MTD）目標 | 同上 |

---

## Firebase 資料結構

所有資料存於 Firebase Realtime Database，路徑結構如下：

```
（資料庫根目錄）
│
├── config/
│   ├── target: 440000              ← 部門業績目標（可從主頁修改）
│   ├── qt_target: "85%"            ← Q/T 目標（後台設定）
│   ├── nps_target: "90"            ← NPS 目標（後台設定）
│   └── growth_target: "+5%"        ← 部門TO成長率目標（後台設定）
│
├── days/
│   └── 2026-05-18/
│       ├── performance: "380000"   ← 今日業績
│       ├── qt: "88%"               ← Q/T 填寫值
│       ├── nps: "92"               ← NPS 填寫值
│       ├── growth: "+3.2%"         ← 部門TO成長率填寫值
│       ├── tasks: [true, true, false, null, true, true]  ← 例行事務狀態
│       └── notes: "今日重點..."     ← 晨會佈達內容
│
└── handovers/
    └── -自動產生ID/
        ├── text: "確認倉庫庫存"     ← 交接事項內容
        ├── date: "2026-05-18"      ← 新增日期
        ├── done: false             ← 是否已完成
        └── ts: 1747584000000       ← 建立時間戳記（用於排序）
```

---

## 部署說明

### GitHub Pages 設定

1. 將 `docs/index.html` 與 `docs/admin.html` 上傳至 GitHub repository
2. 進入 repository → Settings → Pages
3. Branch 選 `main`，資料夾選 `/ (root)` 或 `/docs`
4. 儲存後等待約 1～2 分鐘，網址格式為：
   `https://【帳號】.github.io/【repository名稱】/`

### Firebase 設定

1. 前往 [console.firebase.google.com](https://console.firebase.google.com)，建立新專案
2. 左側選「建構」→「Realtime Database」→「建立資料庫」→「以測試模式啟動」
3. 複製資料庫網址（格式：`https://xxx-default-rtdb.firebaseio.com`）
4. 將網址填入 `index.html` 與 `admin.html` 的 `FIREBASE_URL` 常數

---

## 維護注意事項

### Firebase 測試模式到期（建立後 30 天）

測試模式的安全規則 30 天後自動到期，到期後夥伴將無法存取資料。到期前請至 Firebase Console 手動更新規則：

1. Firebase Console → 你的專案 → Realtime Database → 規則
2. 將規則改為以下內容並儲存：

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

### 更新網頁內容

修改 `docs/` 內的 HTML 檔案後，重新上傳至 GitHub repository 即可自動更新，約 1～2 分鐘生效。

### 更換管理員密碼

開啟 `docs/admin.html`，找到以下這行，將 `dkt2024` 改為新密碼：

```javascript
const ADMIN_PW = 'dkt2024';
```

修改後重新上傳至 GitHub。

---

## 免費額度說明

| 平台 | 限制 | 預估使用量 | 是否足夠 |
|------|------|-----------|---------|
| GitHub Pages | 流量 100 GB／月 | < 0.1 GB／月 | ✅ 充裕 |
| Firebase 儲存 | 1 GB | < 10 MB／年 | ✅ 充裕 |
| Firebase 流量 | 10 GB／月 | < 0.01 GB／月 | ✅ 充裕 |
| Firebase 連線數 | 100 人同時 | 約 5～20 人 | ✅ 充裕 |

以門市小型團隊的使用規模，兩個平台的免費額度均綽綽有餘，無需付費升級。
