# ERPNext 版本更新日誌

**建立日期**：2026-06-09
**升級區間**：ERPNext v16.16.0 → v16.21.1 / Frappe Framework v16.17.0 → v16.20.0
**資料來源**：[Frappe GitHub Releases](https://github.com/frappe/frappe/releases) / [ERPNext GitHub Releases](https://github.com/frappe/erpnext/releases)
**升級方式**：`bench update --reset` + `bench --site site1.local migrate`

---

## 升級前後版本對照

| 項目 | 升級前 | 升級後 |
|------|--------|--------|
| Frappe Framework | v16.17.0 | **v16.20.0** |
| ERPNext | v16.16.0 | **v16.21.1** |

---

## ERPNext 更新內容

### v16.21.1（2026-06-03）

**Bug 修復**
- 修復 **Item** 列表自訂欄位標題影響報表檢視的問題

### v16.21.0（2026-06-02）

**新功能**
- **Buying Settings** 新增獨立的「超額訂購容許百分比」，與超收容許百分比分開
- 取消有銀行交易記錄的 **Payment Entry** 時增加確認提示，說明取消會同時移除銀行對帳

**Bug 修復**
- **Stock Entry**  transit 追蹤改用 Transfer Qty 而非 Qty
- **Work Order** 操作表的 Batch Size 可輸入小數
- **Stock Reconciliation** 序號數量重複計算修正
- **Sales Funnel** 跳過缺少 UTM Source 的 Opportunity
- **Financial Report Template** 結帳餘額加總修正
- **Tax Withholding Category** 空白群組比對修正
- **BOM** 新版不覆蓋已複製的操作明細
- 取消 **Stock Entry** 時 Bin 數量改用歷史資料修正
- **BOM** 超額訂購容許值從設定讀取修正

### v16.20.1（2026-06-01）

**Bug 修復**
- 採購表單更換 Company 時自動更新 Billing Address 與 Shipping Address

### v16.20.0（2026-05-27）

**新功能**
- **Job Card** 新增 Pending Qty 欄位，支援部分完工；Process loss 在 Pending Qty 為零時才計算
- **Party Specific Item** 新增 Customer Group / Supplier Group 層級支援

**Bug 修復**
- **Stock Reconciliation** 序號品項重複計算修正
- **Production Plan** 保留庫存時自動填入 From BOM 欄位
- **Sales Order** 缺少倉庫提示文字修正為 "Source warehouse"
- "Update Items" 按鈕對新增列自動填入庫存明細
- 退貨 **Sales Invoice**（無原始發票）手動輸入成本費率不再被覆蓋
- Customer/Supplier **Item Price** 不同 UOM 時也顯示價格
- **Stock Entry** 製造成本按操作分開追蹤
- **Asset** 複合資產 Net Purchase Amount 保留現值
- **Sales Invoice** Debit Note 說明文字釐清
- **Payment Entry** 外幣交易自動備註幣別顯示修正

### v16.19.1（2026-05-20）

**效能改善**
- **Process Period Closing Voucher** 開帳日期計算加速，排程時不鎖定其他記錄

### v16.19.0（2026-05-20）

**新功能**
- 財務報表列印格式大改版，新增專用列印範本：
  - Balance Sheet（資產負債表）
  - Profit and Loss Statement（損益表）
  - Cash Flow Statement（現金流量表）
  - Trial Balance（試算表）
  - Accounts Receivable Summary / Accounts Payable Summary
  - General Ledger（總帳）
  - Accounts Receivable / Accounts Payable

**Bug 修復**
- General Ledger 報表 Categorize By 空白時顯示逐筆分錄
- **Item** 增加工具提示與自動價格建立提示
- **Stock Reconciliation** 整數 UOM 不允許小數數量
- **Stock Entry**（Repack）驗證成品 target warehouse 與原料 source warehouse
- 庫存交易 Posting Date/Time 合併修正
- **Buying Settings** 表單版面清理與 Dark Mode 支援
- Drop-ship PO/SO 狀態同步修正
- General Ledger 新增「停用開帳餘額計算」選項

### v16.18.3（2026-05-14）

**Bug 修復**
- Drop-ship PO/SO 狀態隨收貨數量自動同步

### v16.18.2（2026-05-14）

**Bug 修復**
- 庫存交易使用輸入的 Posting Date/Time 而非當前時間，避免批號數量錯誤

### v16.18.1（2026-05-13）

**其他變更**
- 退回 rejected qty 計入 billed total 的變更，Purchase Receipt 恢復排除 rejected items

### v16.18.0（2026-05-12）

**Bug 修復**
- **Work Order** 從 Production Plan 建立時序號傳遞修正
- **Subcontracting Order** 服務項目計算修正
- **Landed Cost Voucher** 多幣別匯率處理修正
- 進項稅額分錄帳戶自動帶入修正

### v16.17.0（2026-05-05）

**新功能**
- **Selling Settings** 新增命名序號預覽對話框
- **Terms and Conditions** 支援自動複製附件到交易文件

**Bug 修復**
- PO/SO/DN/PINV/SINV/PR 新增列自動複製 Project 值
- **Blanket Order** 項目 ordered qty 隨 SO 狀態重新計算
- Serial No Ledger 報表狀態顯示修正
- **Sales Forecast** Connections tab 資料庫錯誤修正
- 已提交 **Purchase Receipt** 不再顯示序號/批號按鈕
- **Payment Entry** 日期範圍篩選 outstanding references 錯誤修正
- 已完成/已取消 Project 從下拉選單移除

---

## Frappe Framework 更新內容

### v16.20.0（2026-06-02）

**新功能**
- **表格批次編輯**：表內選取多行可批次修改（Customize Form 可開關 Allow Bulk Edit）
- **側邊欄折疊按鈕**：hover 時顯示折疊/展開按鈕
- **全域搜尋大改版**：網格顯示結果、搜尋字高亮、支援 Cmd+G 快速鍵
- 欄位選擇器（表格欄位、報表欄位）支援輸入即搜尋
- Global Search Settings 新增「Configure」按鈕選擇搜尋欄位

**Bug 修復**
- **System Settings** 新增 Allowed Doctypes for Guest Uploads 限制訪客上傳
- 只有 System Manager 可還原已刪除文件，並檢查原始文件權限
- 自訂 Workspace Sidebar 重新命名後不再覆寫檔案
- 選單圖示樣式隔離，不影響間距
- Ctrl/Cmd+K 在表單中開啟 App 搜尋列而非瀏覽器搜尋
- **Number Card** 必填欄位條件修正
- 從 Address 列表新增地址時不帶入前一個 Customer
- Banker's rounding 負數四捨五入修正

### v16.19.0（2026-05-27）

**新功能**
- **側邊欄記憶功能**：每個頁面記住上次顯示的側邊欄

**Bug 修復**
- 表單記錄導航（上/下一筆）在排序欄位為空時仍可運作
- 下拉欄位移除多餘垂直 padding
- 多選對話框首次搜尋使用預設值
- 含空白表格區塊的記錄可匯出
- **Report** 儲存時檢查權限
- 自訂報表欄位過濾修正
- 通知面板關閉按鈕行為修正
- 備份下載路徑安全檢查
- **Communication** timeline 連結記錄標題顯示
- PDF 產生改用新瀏覽器 session
- Dashboard 圖表讀取新舊格式篩選條件
- Web Form 多頁表單關閉已開啟的表格列
- 模組側邊欄在無 workspace sidebar 時顯示

### v16.18.3（2026-05-20）

**Bug 修復**
- **Document Follow** 狀態在頁面重新載入後正確反映
- 標準 Report 在 migrate/patch/install/import 時可儲存
- 報表訊息與狀態訊息分開顯示
- Read Only 欄位值檢查修正
- 升級時移除殘留 Workspace 項目後再建立新項目
- 無圖片欄位設定時不觸發預覽錯誤
- Prepared Report CSV 匯出總計列修正
- Workspace Manager 可儲存未被指派的公開 Workspace
- 共用 User 記錄給本人時跳過追蹤
- 暫時登入連結使用站台自身地址
- Web Form 只顯示已發布且欄位相符的連結選項
- 限制 Script 中的 SQL 匯出
- Timeline 絕對時間顯示修正

### v16.18.2（2026-05-15）

**Bug 修復**
- Prepared Report 含總計列時 CSV 匯出錯誤修正

### v16.18.1（2026-05-13）

**Bug 修復**
- 標準 Report 在系統更新時跳過 developer mode 檢查

### v16.18.0（2026-05-12）

**新功能**
- **Assignment Rule** 新增加權分配模式（Weighted Distribution）

**Bug 修復**
- 刪除 DocType 欄位時同步移除 unique rule 與 index
- 使用者自訂 Workspace 只能由擁有者編輯
- 停用的 Report 在側邊欄與 workspace 中隱藏
- **User** Role Profiles 清除後儲存正確
- 新增 User 時密碼在自動儲存後保留
- 多個下拉選單不再同時開啟
- 導航列高度 45px → 48px
- Azure PostgreSQL 安裝時 public schema 權限修正
- Setup Wizard 在 dev server 不自動登入
- **Communication** Relink 權限檢查
- Website preload Link header 大小限制
- 表單欄位說明圖示簡化
- Webhook 新增 "Update After Submit" 事件
- 記錄導航快捷鍵改為 Ctrl+Shift+> / Ctrl+Shift+<

### v16.17.5（2026-05-07）

**Bug 修復**
- **User** Role Profiles 清除後儲存正確

### v16.17.4（2026-05-05）

**Bug 修復**
- 私人 File 權限驗證
- User Type 文件類型數量限制移除
- **Document Follow** 權限檢查
- Permission Type 列表恢復 Add 按鈕
- 報表連結標題 HTML 轉義
- 子表格跨頁批次貼上修正
- API key 登入錯誤處理
- 邀請連結使用後立即失效
- 檔案上傳預覽改進
- Video 檔案圖示顯示
- MariaDB/SQLite 分頁查詢預設 LIMIT
- Datetime/Time 欄位匯出修正
- Default App 含空白字元導致登入失敗修正
- File 子目錄上傳修正

### v16.17.3（2026-05-04）

**Bug 修復**
- Print Format Small Text 欄位保留原始字元不轉換 HTML entities

---

## 注意事項

- 本次升級使用 `bench update --reset` 方式，app 目錄內無自訂程式碼修改
- 升級後需執行 `bench --site site1.local migrate` 同步資料庫 schema
- 升級後需執行 `sudo systemctl restart frappe-bench-frappe-web.service` 重啟 web 服務
- 版本確認需使用無痕視窗，避免瀏覽器快取顯示舊版號
