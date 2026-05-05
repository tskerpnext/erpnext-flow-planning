# ERPNext — 單據無法刪除或取消 解決方法

**建立日期**：2026-04-15
**更新日期**：2026-05-05（新增 SLE actual_qty 異常修正章節；強化 check_stock_integrity.py 警告說明）

---

## 錯誤訊息對照

| DocType | 錯誤訊息範例 |
|---------|-------------|
| 物料移動 Stock Entry | `由于物料移动 MAT-STE-2026-xxxxx 与物料凭证 xxxxxxxx 关联，无法删除或取消` |
| 銷售出庫 Delivery Note | `由于销售出库 MAT-DN-2026-xxxxx 与总账分录 xxxxxxxx 关联，无法删除或取消` |
| 庫存調賬 Stock Reconciliation | `由于库存调账 MAT-RECO-2026-xxxxx 与总账分录 xxxxxxxx 关联，无法删除或取消` |
| 批號 Batch | `由于批号 xxx 与序列号与批号 xxxxxxxxxxxxxxxx 关联，无法删除或取消` |
| 銷售發票 Sales Invoice | `由于销售发票 ACC-SINV-2026-xxxxx 与收付款台账 xxxxxxxx 关联，无法删除或取消` |

---

## 原因

ERPNext 單據提交後會產生關聯記錄（GL Entry、SLE、Serial and Batch Bundle 等）。
若流程中途中斷或狀態異常，主單據無法正常取消/刪除，需手動強制處理。

---

## 方法選擇

| 情況 | 建議方法 |
|------|---------|
| Stock Entry、Delivery Note | **方法 A：frappe.delete_doc**（框架自動清理 SLE/Bin） |
| Stock Reconciliation（尤其有 amended 版本） | **方法 B：直接 SQL**（amended 鏈需手動控制刪除順序） |
| 批號被 Serial and Batch Bundle 卡住 | **方法 C：直接 SQL 清除孤立 Bundle 再刪 Batch** |
| Sales Invoice（已取消，Payment Ledger Entry delinked） | **方法 A：frappe.delete_doc** |

---

## 方法 A：frappe.delete_doc（推薦）

適用：**Stock Entry、Delivery Note**

### Step 1：確認 docstatus

```bash
cd /home/stanley/frappe-bench
bench --site site1.local mariadb
```

```sql
-- Stock Entry
SELECT name, docstatus FROM `tabStock Entry` WHERE name = 'MAT-STE-2026-xxxxx';

-- Delivery Note
SELECT name, docstatus FROM `tabDelivery Note` WHERE name = 'MAT-DN-2026-xxxxx';
-- docstatus: 0=草稿 1=已提交 2=已取消
```

按 `exit` 離開。

---

### Step 2：進入 bench console

```bash
bench --site site1.local console
```

---

### Step 3：取消（若 docstatus = 1）

```python
# Stock Entry
doc = frappe.get_doc("Stock Entry", "MAT-STE-2026-xxxxx")

# Delivery Note
doc = frappe.get_doc("Delivery Note", "MAT-DN-2026-xxxxx")

doc.flags.ignore_linked_doctypes = True
doc.cancel()
frappe.db.commit()
```

---

### Step 4：強制刪除

```python
# Stock Entry
frappe.delete_doc("Stock Entry", "MAT-STE-2026-xxxxx", force=True, ignore_permissions=True)

# Delivery Note
frappe.delete_doc("Delivery Note", "MAT-DN-2026-xxxxx", force=True, ignore_permissions=True)

# Sales Invoice
frappe.delete_doc("Sales Invoice", "ACC-SINV-2026-xxxxx", force=True, ignore_permissions=True)

frappe.db.commit()
```

> ✅ `frappe.delete_doc` 走框架，會自動刪除關聯的 SLE、更新 Bin.actual_qty、清除子表記錄。
> ⚠️ Delivery Note 刪除後，對應的 Sales Order 狀態會回到未出貨，確認符合預期再執行。
> ℹ️ Sales Invoice 的 Payment Ledger Entry（收付款台账）若已 delinked=1，直接 delete_doc 即可；若 delinked=0 代表有未結清款項，需先確認付款狀態再處理。

按 **Ctrl+D** 離開 console。

---

### Step 5：驗證 Bin 一致性

```bash
cd /home/stanley/projects/erpnext-customizations
source /home/stanley/frappe-bench/env/bin/activate
python3 check_stock_integrity.py
```

腳本會回報兩種類型，處理方式不同：

| 警告類型 | 符號 | 意義 | 處理方式 |
|---------|------|------|---------|
| 一般不一致 | ⚠️ | Bin 與 SLE 加總不符，SLE 為準 | 執行 `--fix` 自動修正 |
| SLE 疑似異常 | 🔍 | `SUM(actual_qty)=0` 但 `qty_after_transaction>0` | **不可用 `--fix`**，參考下方「SLE actual_qty 異常修正」 |

---

## 方法 B：直接 SQL（Stock Reconciliation）

適用：**Stock Reconciliation**，尤其有 amended 版本鏈時（-1、-2 等）

### Step 1：確認所有相關調賬單狀態

```sql
SELECT name, docstatus, amended_from, posting_date
FROM `tabStock Reconciliation`
WHERE name LIKE 'MAT-RECO-2026-%'
ORDER BY name;
-- 確認 docstatus 均為 2（已取消）才繼續
```

---

### Step 2：確認關聯記錄狀態

```sql
-- GL Entry（確認 is_cancelled = 1）
SELECT name, voucher_no, account, debit, credit, is_cancelled
FROM `tabGL Entry`
WHERE voucher_no IN ('MAT-RECO-2026-xxxxx')
ORDER BY voucher_no;

-- Stock Ledger Entry
SELECT name, voucher_no, item_code, warehouse, actual_qty, is_cancelled
FROM `tabStock Ledger Entry`
WHERE voucher_no IN ('MAT-RECO-2026-xxxxx')
ORDER BY voucher_no;
```

---

### Step 3：依序刪除

```sql
-- 1. 刪除 GL Entry
DELETE FROM `tabGL Entry`
WHERE voucher_no IN ('MAT-RECO-2026-xxxxx');

-- 2. 刪除 Stock Ledger Entry
DELETE FROM `tabStock Ledger Entry`
WHERE voucher_no IN ('MAT-RECO-2026-xxxxx');

-- 3. 刪除子表
DELETE FROM `tabStock Reconciliation Item`
WHERE parent IN ('MAT-RECO-2026-xxxxx');

-- 4. 主表：amended 版本從最新往回刪
DELETE FROM `tabStock Reconciliation`
WHERE name IN ('MAT-RECO-2026-xxxxx-2', 'MAT-RECO-2026-xxxxx-1', 'MAT-RECO-2026-xxxxx');
```

> ⚠️ 直接 SQL DELETE 繞過框架，**Bin.actual_qty 不會自動更新**，Step 4 驗證為必做。

---

### Step 4：驗證 Bin 一致性

```bash
cd /home/stanley/projects/erpnext-customizations
source /home/stanley/frappe-bench/env/bin/activate
python3 check_stock_integrity.py
```

| 狀況 | 處理方式 |
|------|---------|
| ✅ 全部一致 | 完成 |
| Bin 有殘留（SLE 已無對應） | `python3 check_stock_integrity.py --fix` |
| Bin 代表真實庫存，SLE 被刪 | 不可歸零，需重新建一張 Stock Reconciliation 補回 |

---

### Step 5：確認清除

```sql
SELECT name FROM `tabStock Reconciliation`
WHERE name IN ('MAT-RECO-2026-xxxxx');
-- 應回傳空結果
```

---

## 方法 C：批號被孤立 Bundle 卡住

適用：刪除 **Batch（批號）** 時出現 `由于批号 xxx 与序列号与批号 xxxxxxxx 关联，无法删除或取消`

常見原因：原本關聯的主單據（Stock Reconciliation 等）已被刪除，但 Serial and Batch Bundle 成為孤立記錄，仍持有對 Batch 的連結。

---

### Step 1：確認 Batch 與 Bundle 狀態

```bash
cd /home/stanley/frappe-bench
bench --site site1.local mariadb
```

```sql
-- 確認 Batch
SELECT name, item, batch_qty, disabled FROM `tabBatch` WHERE name = 'xxx';

-- 確認 Bundle（用錯誤訊息中的 Bundle name）
SELECT name, voucher_no, voucher_type, item_code, docstatus
FROM `tabSerial and Batch Bundle`
WHERE name = 'xxxxxxxxxxxxxxxx';
```

若 `voucher_no` 指向的主單據已不存在（或已刪），即為孤立記錄，可安全刪除。

---

### Step 2：確認 Bundle 子表內容

```sql
SELECT name, parent, batch_no, qty
FROM `tabSerial and Batch Entry`
WHERE parent = 'xxxxxxxxxxxxxxxx';
```

---

### Step 3：刪除孤立 Bundle

```sql
-- 先刪子表
DELETE FROM `tabSerial and Batch Entry` WHERE parent = 'xxxxxxxxxxxxxxxx';

-- 再刪主表
DELETE FROM `tabSerial and Batch Bundle` WHERE name = 'xxxxxxxxxxxxxxxx';
```

按 `exit` 離開 mariadb。

---

### Step 4：刪除 Batch

```bash
bench --site site1.local console
```

```python
frappe.delete_doc("Batch", "xxx", force=True, ignore_permissions=True)
frappe.db.commit()
```

按 **Ctrl+D** 離開 console。

---

### Step 5：確認清除

```bash
bench --site site1.local mariadb
```

```sql
SELECT name FROM `tabBatch` WHERE name = 'xxx';
SELECT name FROM `tabSerial and Batch Bundle` WHERE name = 'xxxxxxxxxxxxxxxx';
-- 兩筆均應回傳空結果
```

---

---

## 附錄：SLE actual_qty 異常修正

### 發生情境

執行 `check_stock_integrity.py` 後出現 🔍 警告（而非 ⚠️），顯示：

```
🔍 發現 N 筆 SLE actual_qty 疑似異常（加總=0 但 qty_after_transaction>0）
```

**根本原因**：Stock Reconciliation 提交時，若目標數量等於當前 Bin 數量（差異 = 0），ERPNext 仍建立 SLE，但 `actual_qty` 寫入 0，`qty_after_transaction` 則正確記錄。腳本用 `SUM(actual_qty)` 比對時誤判為不一致。

---

### Step 1：確認受影響品項與倉庫

由腳本輸出取得 `item_code` 與 `warehouse`，或直接查詢：

```sql
SELECT item_code, warehouse, actual_qty, qty_after_transaction, voucher_no, posting_date
FROM `tabStock Ledger Entry`
WHERE is_cancelled = 0
  AND actual_qty = 0
  AND qty_after_transaction > 0
ORDER BY warehouse, item_code;
```

確認 `qty_after_transaction` 與 Bin 顯示的庫存一致（代表 Bin 正確、只是 SLE actual_qty 記錯）。

---

### Step 2：SQL 修正 SLE actual_qty

```bash
cd /home/stanley/frappe-bench
bench --site site1.local mariadb
```

```sql
UPDATE `tabStock Ledger Entry`
SET actual_qty = qty_after_transaction
WHERE item_code IN ('品項1', '品項2', ...)
  AND warehouse = '倉庫名稱 - SKT'
  AND is_cancelled = 0
  AND actual_qty = 0
  AND qty_after_transaction > 0;
```

> ⚠️ `IN` 清單請依 Step 1 查詢結果填入，勿使用萬用條件避免誤改其他記錄。

按 `exit` 離開。

---

### Step 3：驗證

```bash
cd /home/stanley/projects/erpnext-customizations
python3 check_stock_integrity.py
```

應回報 `✅ 所有 Bin 數量與 SLE 一致，無需修正。`

---

### 預防說明

此問題源自 ERPNext 框架寫入邏輯，目前採**觀察策略**：
- `check_stock_integrity.py` 已加入偵測，下次發生時可即時發現
- 若調賬頻率提高導致問題常態出現，可評估加裝 Server Script 在 Stock Reconciliation `on_submit` 後自動修正

---

## 若有關聯的 Pick List 也需刪除

Pick List 必須在 Stock Entry 刪除後才能處理：

```python
frappe.delete_doc("Pick List", "STO-PICK-2026-xxxxx", force=True, ignore_permissions=True)
frappe.db.commit()
```

---

## 注意事項

| 項目 | 說明 |
|------|------|
| `force=True` | 跳過所有關聯檢查，確認單據確實不需保留再執行 |
| Stock Entry 孤立 Bundle | 建立時若流程中斷，Serial and Batch Bundle 可能成為孤立記錄，可定期清理 |
| GL Entry 未沖銷 | SQL 刪除前必確認 `is_cancelled = 1`，否則造成帳務異常 |
| amended 刪除順序 | 最新版本（-2）→ 中間（-1）→ 原始版本，不可顛倒 |
| Delivery Note 刪除影響 | 對應 Sales Order 會回到未出貨狀態 |
| 初始入庫記錄 | 若調賬單是品項唯一入庫，刪除後需重新建調賬單補回庫存 |
| Bundle 孤立判斷 | voucher_no 指向的主單據不存在，且 Bundle docstatus = 2，才算孤立 |
| check_stock_integrity.py 警告分類 | ⚠️ 一般不一致可用 `--fix`；🔍 SLE 疑似異常需人工確認後 SQL 修正，不可直接 `--fix` |

---

## 孤立 Bundle 清查 SQL（定期維護）

```sql
-- Stock Entry 類孤立 Bundle
SELECT b.name, b.voucher_no, b.item_code
FROM `tabSerial and Batch Bundle` b
LEFT JOIN `tabStock Entry` se ON b.voucher_no = se.name
WHERE b.voucher_type = 'Stock Entry'
  AND se.name IS NULL;

-- Stock Reconciliation 類孤立 Bundle
SELECT b.name, b.voucher_no, b.item_code
FROM `tabSerial and Batch Bundle` b
LEFT JOIN `tabStock Reconciliation` sr ON b.voucher_no = sr.name
WHERE b.voucher_type = 'Stock Reconciliation'
  AND sr.name IS NULL;
```

---

## 相關工具

| 工具 | 路徑 | 說明 |
|------|------|------|
| check_stock_integrity.py | `/home/stanley/projects/erpnext-customizations/` | 檢查並修正 Bin 與 SLE 不一致 |

---

*建立日期：2026-04-15 ／ 更新日期：2026-05-05*
*適用版本：ERPNext v16 / Frappe v16*
