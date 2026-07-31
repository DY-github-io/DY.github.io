# 拼貼筆記 — 網站維護指南

## 這是什麼
GitHub Pages 上的個人資料收藏網站。使用者（naveenaa）用手機或電腦直接操作 UI 新增日常內容；需要永久保存或結構性修改時，由 Claude 編輯 index.html 並推送。

## 更新內容的做法
使用者說「幫我加一筆：[內容]」時：
1. 讀 index.html，找到 `SEED_ENTRIES` 陣列
2. 新增一筆，id 格式為 `seed-XXX`（流水號，看目前最大的 +1）
3. commit + push 直接推 main（不開分支，改了馬上上線）

種子資料會自動合併進使用者的雲端資料，不會覆蓋他們自己加的內容。使用者刪除過的種子 ID 會被記住，不會重新出現。

## 每筆資料的格式
```javascript
{
  id: 'seed-002',           // 必填，seed- 前綴 + 流水號
  title: '標題',             // 必填
  tags: ['分類1', '分類2'],   // 選填，陣列
  note: '簡短說明',           // 選填
  image: 'https://...',     // 選填，圖片網址
  link: 'https://...',      // 選填，來源連結
  createdAt: 'ISO字串'       // 必填，用當下時間
}
```

## 預設標籤
參考資料、想買的、想存的、食譜。使用者可以自訂新標籤。

## 設計規則
- 深色底、暖色卡片（ink + paper 配色）
- 主色：brass (#C9A227)、jade (#4B7A6C)
- 字體：Noto Serif TC（標題）、Noto Sans TC（內文）、IBM Plex Mono（標籤/小字）
- 不加不必要的功能，保持目前的簡潔感

## 技術備註
- 純靜態 HTML，無後端，託管在 GitHub Pages
- 資料存在 Firebase Realtime Database（跨裝置同步）
- 種子資料寫在 index.html 的 `SEED_ENTRIES` 陣列裡
- 資料變動時會同步寫入 Google Sheets 作為可瀏覽的備份
- Google Fonts 外連（GitHub Pages 可用）
- 匯出功能會產生 .md 備份檔

## 雲端服務
### Firebase Realtime Database
- 專案名稱：naveenaa-note
- 用途：跨裝置即時同步所有筆記資料
- 資料路徑：`/data/entries/` 存使用者新增的資料，`/data/deletedSeeds/` 記錄已刪除的種子 ID
- 設定寫在 index.html 的 `firebase.initializeApp(...)` 裡
- 安全規則：開放讀寫（資料非機密，與密碼鎖同等級的防護）
- 免費方案，不需付費

### Google Sheets 同步
- 試算表：https://docs.google.com/spreadsheets/d/14DhpOCaLdfzvoMzNRgiss2zsc0Ugygsnhv-ffSMEzP0
- 用途：讓使用者可以在試算表裡瀏覽、整理所有資料
- 透過 Google Apps Script 接收資料，每次網頁上有變動會自動同步整份資料到試算表
- Apps Script 網址寫在 index.html 的 `SHEET_URL` 變數裡
- 如果要重新部署 Apps Script，需要更新 `SHEET_URL`

## 密碼鎖
首頁進去前有一層密碼提示框（`#lockOverlay`），前端用 SHA-256 比對輸入值。
- 這是輕度的網頁鎖，不是真正的安全機制：repo 是公開的，密碼的雜湊值也在原始碼裡，懂技術的人可以繞過或直接用 raw 檔案網址讀到內容
- 通過一次後會記在該裝置的 localStorage（key: `naveenaa-unlocked`），下次同一台裝置/瀏覽器打開不用再輸入
- 要換密碼：把 `PASS_HASH` 換成新密碼的 SHA-256 雜湊值（不要直接寫明文密碼進原始碼）

## 語言與溝通
- 所有面對使用者的文字用繁體中文
- 使用者不是工程師，避免技術術語；要用就附白話解釋
- 語氣像朋友，輕鬆自然；要產出東西時像可靠的助理把事辦好
- 給建議不只列選項，說推薦哪個、為什麼
- 講真話，有更好的做法直接說

## 做事規則
- 超過兩步的任務先說要做什麼再動手
- 涉及檔案操作（新增/修改/推送），執行完要驗證，確認成功才回報完成
- 不做重複的事，動手前先查目前狀態
- 不加不必要的功能，保持簡潔
