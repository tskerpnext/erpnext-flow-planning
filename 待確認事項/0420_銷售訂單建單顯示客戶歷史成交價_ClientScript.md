# 銷售訂單建單時顯示客戶歷史成交價 — Client Script 規劃

建立日期：2026-04-20
適用版本：ERPNext V16
發生環境：本機（192.168.1.60）

---

## 需求說明

建立銷售訂單時，當使用者填入物料後，自動查詢該客戶對此物料的最近一筆成交價，顯示在畫面上供參考，不強制覆蓋，使用者仍可手動修改。

---

## 與 Server Script 版本比較

| 項目 | Server Script（已規劃） | Client Script（本文） |
|------|------------------------|----------------------|
| 觸發時機 | 銷售訂單**結案後** | 銷售訂單**建單中**（填物料時） |
| 作用 | 自動更新 tabItem Price | 畫面上顯示歷史成交價供參考 |
| 寫入資料庫 | ✅ 會寫入 | ❌ 不寫入 |
| 影響所有使用者 | ✅ | 只影響當前操作者畫面 |
| 升級風險 | 中（依賴 frappe 事件 API） | 低（依賴 JS 欄位事件，較穩定） |
| 適合情境 | 自動維護價格，下次預帶 | 讓業務在建單時看到參考價 |

---

## 升級相容性分析

### Server Script 升級風險
- `on_update` 事件與 `get_doc_before_save()` 為 Frappe 核心 API，歷版本穩定
- `frappe.get_all`、`frappe.db.set_value` 為基礎方法，不易被移除
- **風險點**：若 ERPNext 修改 Sales Order 結案流程（如新增中間狀態），觸發邏輯可能需調整
- **整體風險：低～中**

### Client Script 升級風險
- `frm.fields_dict`、`frm.set_value` 為 Frappe Form API，長期穩定
- 查詢使用 `frappe.call`，為標準前後端溝通方式
- **風險點**：物料行欄位名稱（`items.item_code`、`items.rate`）若改名則失效，但 ERPNext 核心欄位極少改名
- **整體風險：低**

---

## Client Script 規格

| 欄位 | 值 |
|------|-----|
| 類型 | Form |
| 文件類型 | Sales Order |
| 觸發事件 | items 子表的 item_code 欄位 onChange |
| 腳本名稱 | show-last-price-on-item-select |

---

## 完整腳本

```javascript
// 銷售訂單建單時，填入物料後顯示該客戶最近成交價
frappe.ui.form.on("Sales Order Item", {
    item_code: function(frm, cdt, cdn) {
        var row = locals[cdt][cdn];
        if (!row.item_code || !frm.doc.customer) return;

        frappe.call({
            method: "frappe.client.get_list",
            args: {
                doctype: "Sales Order Item",
                filters: [
                    ["parent", "!=", frm.doc.name],
                    ["item_code", "=", row.item_code],
                    ["Sales Order", "customer", "=", frm.doc.customer],
                    ["Sales Order", "docstatus", "=", 1],
                ],
                fields: ["rate", "parent", "Sales Order.transaction_date"],
                order_by: "Sales Order.transaction_date desc",
                limit: 1,
            },
            callback: function(r) {
                if (r.message && r.message.length > 0) {
                    var last = r.message[0];
                    frappe.show_alert({
                        message: __("客戶 {0} 上次購買 {1} 的成交價：{2} 元（訂單：{3}）",
                            [frm.doc.customer, row.item_code, last.rate, last.parent]),
                        indicator: "blue"
                    }, 8);
                } else {
                    frappe.show_alert({
                        message: __("{0} 尚無此客戶的歷史成交紀錄", [row.item_code]),
                        indicator: "grey"
                    }, 5);
                }
            }
        });
    }
});
```

---

## 程式碼說明

| 段落 | 說明 |
|------|------|
| `frappe.ui.form.on("Sales Order Item", ...)` | 監聽子表物料行的欄位變更事件 |
| `item_code: function(frm, cdt, cdn)` | 當 item_code 欄位變更時觸發 |
| `frappe.call / frappe.client.get_list` | 向後端查詢歷史訂單資料 |
| filters 條件 | 排除目前訂單、同客戶、已提交、同物料 |
| `order_by: "transaction_date desc"` | 取最新一筆 |
| `frappe.show_alert` | 在畫面右上角顯示提示，8 秒後自動消失，不影響欄位值 |

---

## 部署步驟（待執行）

1. 進入 ERPNext → 構建 → Client Script → 新增
2. 填入上表規格
3. 貼入腳本內容
4. 勾選「已啟用」
5. 儲存

部署後測試：新建銷售訂單 → 選客戶 → 在物料行填入曾售出的物料 → 右上角應出現藍色提示顯示上次成交價。

---

## 兩版本部署建議

| 情境 | 建議部署 |
|------|----------|
| 只需「下次建單自動帶正確價格」 | Server Script 即可 |
| 需要「建單時看到參考價再決定」 | Client Script |
| 兩者都想要 | 同時部署，功能互補不衝突 |

兩個腳本功能不重疊，可以同時部署。
