# ERPNext v16.14.0 更新日誌 (繁體中文翻譯)

> - **建立日期：** 2026-06-09
> - **來源：** [Frappe v16.15.0](https://github.com/frappe/frappe/releases/tag/v16.15.0) / [ERPNext v16.14.0](https://github.com/frappe/erpnext/releases/tag/v16.14.0)
> - **對應升級**：第一次升級 — Frappe v16.14.0 → v16.15.0 / ERPNext v16.13.3 → v16.14.0

---

## 環境資訊

| 項目 | 內容 |
|------|------|
| bench 路徑 | `/home/stanley/frappe-bench` |
| 站台名稱 | `site1.local` |
| 服務管理方式 | systemd |
| 升級日期 | 2026-04-14 ~ 2026-04-22 |
| 第一次升級 | Frappe v16.14.0 / ERPNext v16.13.3 → Frappe v16.15.0 / ERPNext v16.14.0 |

---

## Frappe Framework v16.15.0（2026-04-14）

### ✨ 新功能 (Features)

- **PDF 除錯模式**
  新增 `pdf_debug` 開發者選項，讓 PDF 視窗保持開啟以便排查列印格式問題，不影響一般下載。

- **after_build Hook**
  新增 `after_build` hook，讓每個 app 在建置完成後自動執行自訂任務。

- **複製附件到其他文件**
  不用重新上傳，可直接將現有附件複製到另一份文件，同時保留權限檢查。

- **自訂確認對話框按鈕文字**
  確認彈窗支援自訂按鈕文字，例如顯示「刪除」/「取消」而非固定的「Yes」/「No」。

- **資料庫變更上限設定**
  站台設定新增 `max_db_changes_per_action`，可提高上限以避免儲存大量文件時出現 "Too many changes" 錯誤。

### 🐞 錯誤修復 (Bug Fixes)

- 隱藏手機版 onboarding 彈窗
- 沒有 Email Account 時自動設定預設值，避免通知寄送失敗
- `bench run-tests --junit-xml-output` 不再產生空白 XML
- Portal 列表頁停用 HTML 快取，訂單/報價/發票列表正確顯示
- **File** 的 Attached To Name 連結指向正確文件
- 支援匯入 **Custom DocPerm** 權限設定
- 已取消文件顯示停用的 Amend 按鈕並附說明
- **Translation** Source Text 保留 HTML 格式
- 郵件內嵌圖片（inline_images）正確顯示
- 外部檔案引入增加路徑安全檢查
- 側邊欄在頁面重整後保留 workspace 和自訂連結
- Chrome PDF 匯出修復
- 連結欄位自動完成遵循表單權限設定
- 相對時間標籤改為日曆天計算
- 關閉 Onboarding 時隱藏側邊欄 Getting Started 面板
- 建立私人 Workspace 時自動設定 For User
- 修復非標準桌面圖示無法重新命名
- 連結欄位進階搜尋移除 % 萬用字元提示
- 報表檢視新載入列正確顯示連結標題
- Query Builder 報表 Excel/CSV 匯出修復
- **User** 姓名欄位阻擋 HTML 標籤
- 密碼欄位 show/hide 按鈕圖示一致化

---

## ERPNext v16.14.0（2026-04-14）

### ✨ 新功能 (Features)

- **BOM 工序品質檢驗**
  BOM 每個工序可勾選「需要品質檢驗」，勾選後對應的 **Job Card** 在檢驗完成前無法提交。

- **報價單預設列印格式更新**
  **Quotation** 預設使用新版列印範本。

### 🐞 錯誤修復 (Bug Fixes)

- **Asset Movement** 儲存後保留 Target Location / Source Location / From Employee / To Employee 的必填/選填狀態
- **Purchase Invoice**、**Purchase Receipt**、**Delivery Note** 移除強制六位小數捨入，避免庫存與會計出現 0.01 誤差
- 取消 **Delivery Trip** 不再強制取消關聯的 **Delivery Note**
- **Quality Inspection** 的 Item Code 權限檢查修正
- 從 **Job Card** 建立製造 **Stock Entry** 時剩餘數量為零不再報錯
- **Stock Entry**、**Sales Invoice**、**Delivery Note** 等文件的庫存維度欄位僅在需要時必填
- 庫存維度升級修正
- **Repost Item Valuation** 納入最後一筆 Stock Ledger Entry
- 退貨 Delivery Note 出現在退貨 Sales Invoice 的「從 Delivery Note 取貨」列表中
- **Repost Item Valuation** 自動設定 Posting Time 並驗證倉庫
- **Sales Invoice** / **Purchase Invoice** 的 Remarks 空白時不再顯示 "No Remarks"
- 回溯日期 **Stock Reconciliation** 不再改變已出貨序號狀態
- **Workstation** 列表 Status 篩選修正
- 變更 **Warehouse** 的 Account 時觸發警告
- **BOM Item** 取消 Sourced by Supplier 時恢復原始 Rate
- **Job Card** 工時計時器顯示修復
- 自動建立的 **Batch** / **Serial No** 使用文件 Posting Date 產生編號
- 資料匯入時允許建立新的 **Selling Settings**
- Letterhead 自動填入時增加權限檢查
