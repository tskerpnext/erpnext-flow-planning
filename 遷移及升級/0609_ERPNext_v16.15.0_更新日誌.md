# ERPNext v16.15.0 更新日誌 (繁體中文翻譯)

> - **建立日期：** 2026-06-09
> - **來源：** [Frappe v16.16.0](https://github.com/frappe/frappe/releases/tag/v16.16.0) / [ERPNext v16.15.0](https://github.com/frappe/erpnext/releases/tag/v16.15.0)
> - **對應升級**：第二次升級 — Frappe v16.15.0 → v16.16.0 / ERPNext v16.14.0 → v16.15.0

---

## 環境資訊

| 項目 | 內容 |
|------|------|
| bench 路徑 | `/home/stanley/frappe-bench` |
| 站台名稱 | `site1.local` |
| 服務管理方式 | systemd |
| 升級日期 | 2026-04-21 ~ 2026-04-22 |
| 第二次升級 | Frappe v16.15.0 / ERPNext v16.14.0 → Frappe v16.16.0 / ERPNext v16.15.0 |

---

## Frappe Framework v16.16.0（2026-04-21）

### ✨ 新功能 (Features)

- **POST API 改進**
  POST API 呼叫將資料放入 request body 而非附加在網址後，也可選擇直接傳送 JSON。

- **security.txt 支援**
  **Security Settings** 新增選項發布 `security.txt`，方便訪客回報安全問題，並在檔案即將到期時通知管理員。

- **連結欄位清除按鈕**
  恢復所有連結欄位的 × 清除按鈕。**System Settings** 新增「允許清除連結欄位」開關。

- **側邊欄子選單 Hover 展開**
  滑鼠懸停時自動展開子選單，同時關閉先前展開的選單，保持一次只顯示一個子選單。

- **快捷鍵 Ctrl+G / Cmd+K**
  從任何頁面開啟或關閉 Awesomebar 搜尋面板。

- **快捷鍵 Ctrl+/**
  立即展開或折疊側邊欄。

- **Data Import 自訂分隔符**
  背景作業正確套用自訂分隔符（如分號），CSV 匯入不再出現列錯誤。

### 🐞 錯誤修復 (Bug Fixes)

- **Customize Form** 驗證修正，子表格 DocType 欄位可啟用 In List View
- security.txt 日期改用 UTC 格式並加上結尾換行
- **File** 的 Is Home Folder 欄位加上索引，減少大量檔案時的載入延遲
- **Prepared Report** 卡在 Started 的時序問題修正
- 密碼重設請求不論信箱是否存在都顯示相同訊息，防止列舉攻擊
- 支援 "15-Mar-24" 格式日期匯入
- 登入頁只在語言選擇器啟用時顯示導航列
- Hero banner 網頁正確顯示
- **Note** 儲存時清理不安全 HTML
- 列表標籤篩選 No Tags 正確顯示無標籤文件
- **Custom HTML Block** Private 設定僅限管理員修改
- 連結欄位清除和開啟圖示對齊
- **System Settings** 啟用時連結欄位 Clear 按鈕正常運作
- 列印文件 Text / Long Text 特殊字元跳脫
- **User** 列表正確顯示 Role Profile
- **Web Form** 隱藏欄位跳過必填檢查
- 側邊欄未讀通知點位置修正
- 無模組的公開 **Workspace** 出現在所有使用者側邊欄
- 表格批次貼上列不完整修正

---

## ERPNext v16.15.0（2026-04-22）

### ✨ 新功能 (Features)

- **Sales Order 直接建立 Production Plan**
  **Sales Order** 的 Create 選單新增 Production Plan 選項。

- **BOM Creator 支援 Phantom BOM**
  新增 Is Phantom BOM 勾選框，建立時即可標記。對庫存品項建立 phantom BOM 或對非庫存品項建立一般 BOM 時顯示警告。

- **Item Tax Template 不適用選項**
  稅別明細新增 Not Applicable 勾選框，排除不適用品項，混合稅率發票顯示正確淨額。

- **BOM 回沖方式**
  新增 Backflush Based On 選項，可針對每個 BOM 設定原料消耗方式，不再只能使用全域製造設定。

- **Remark 欄位整合**
  合併 User Remark 和 Remark 為單一 Remark 欄位，新增 Custom Remark 勾選框可自訂文字。

### 🐞 錯誤修復 (Bug Fixes)

- Portal 使用者無法為未關聯供應商建立 **Supplier Quotation**
- **Customer** / **Supplier** 快速輸入時保持 Primary Contact/Address 展開
- 關閉 Rounded Total 時重設隱藏的 base rounding 為零
- 多幣別交易稅額拆分顯示文件幣別
- **Accounting Dimension** 更新時同步刷新會計文件欄位
- **Accounts Settings** 安裝時自動填入 Repost Allowed Types
- **BOM Creator** Phantom 選取時隱藏 Operations 區塊
- **Work Order** Target Warehouse 在特定條件下設為必填
- 選擇對話框移除 Delete row 選項
- Negative Batch Report 每組 batch-warehouse 只顯示一次
- MRP 報表無品項篩選時不再報錯
- 新 **Item** 自動繼承 Item Group 的 Item Tax Template
- 未啟用 Update Stock 的 Sales Invoice 不再顯示零估值警告
- Gross Profit 報表優先使用 dropship 採購資料
- **Buying Settings** 新增 Allow Negative Rates for Items 開關
- VAT Audit Report 限制南非公司
- **Workstation** 變更 Workstation Type 時自動更新作業成本
- 開啟表單時自動填入 Fiscal Year 和 Company
