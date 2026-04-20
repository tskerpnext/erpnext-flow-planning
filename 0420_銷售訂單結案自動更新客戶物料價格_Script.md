# 銷售訂單結案自動更新客戶物料價格 — Server Script

建立日期：2026-04-20
適用版本：ERPNext V16
發生環境：本機（192.168.1.60）

---

## 建立位置

ERPNext 後台 → 構建 → Server Script → 新增

| 欄位 | 值 |
|------|-----|
| 腳本類型 | DocType Event |
| 參考文件類型 | Sales Order |
| 事件 | on_update |
| 腳本名稱 | auto-update-item-price-on-close |

---

## 完整腳本

```python
# 銷售訂單結案後，自動更新 tabItem Price（客戶專屬成交價）
# 觸發：on_update，僅在狀態剛變為 Closed 時執行一次

if doc.status != "Closed":
    return

# 防止重複執行：取結案前的狀態，若前一狀態已是 Closed 則跳過
doc_before = doc.get_doc_before_save()
if doc_before and doc_before.status == "Closed":
    return

price_list = "Standard Selling"
customer = doc.customer
order_date = doc.transaction_date
currency = doc.currency

for item in doc.items:
    # 跳過 rate 為 0 的物料（贈品、免費項目）
    if not item.rate or item.rate == 0:
        continue

    existing = frappe.get_all(
        "Item Price",
        filters={
            "item_code": item.item_code,
            "price_list": price_list,
            "customer": customer,
        },
        fields=["name"],
        limit=1,
    )

    if existing:
        frappe.db.set_value("Item Price", existing[0].name, {
            "price_list_rate": item.rate,
            "valid_from": order_date,
            "currency": currency,
        })
    else:
        new_price = frappe.get_doc({
            "doctype": "Item Price",
            "item_code": item.item_code,
            "price_list": price_list,
            "customer": customer,
            "price_list_rate": item.rate,
            "valid_from": order_date,
            "selling": 1,
            "currency": currency,
            "uom": item.uom,
        })
        new_price.insert(ignore_permissions=True)

frappe.db.commit()
```

---

## 說明

| 段落 | 說明 |
|------|------|
| `doc.status != "Closed"` | 非結案狀態直接跳出，不執行任何更新 |
| `get_doc_before_save()` | 取存檔前的狀態，防止結案後再次編輯時重複觸發 |
| `item.rate == 0` | 跳過贈品或免費物料，不更新價格 |
| `frappe.get_all(...)` | 查詢是否已有此客戶 + 物料的 Item Price 紀錄 |
| `frappe.db.set_value` | 有紀錄則更新單價與生效日期 |
| `frappe.get_doc(...).insert` | 無紀錄則新增一筆客戶專屬 Item Price |
| `frappe.db.commit()` | 確保所有變更寫入資料庫 |

---

## 部署步驟（待執行）

1. 進入 ERPNext → 構建 → Server Script → 新增
2. 填入上表規格
3. 貼入腳本內容
4. 勾選「已啟用」
5. 儲存

部署後測試：開一張銷售訂單 → 提交 → 結案 → 確認 tabItem Price 是否新增/更新對應紀錄。
