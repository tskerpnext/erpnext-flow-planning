# ERPNext 線束生產流程優化指南

**建立日期**：2026-04-16
**更新日期**：2026-04-21

本文件旨在解決線束生產中，部分原料（電線）需從獨立的「線邊倉」領料，而其他原料從「原物料倉」領料的流程管理問題。文件提供了一套完整的倉庫結構設定，以及透過客戶端腳本（Client Script）實現發料倉自動化的解決方案。

---

## 一、 倉庫結構設定

為了清晰地追蹤物料在不同生產階段的狀態，並區分「線邊倉」與「在途庫存」的不同意義，建議採用以下倉庫結構。

### 1. 新建倉庫：線邊倉 (Line-side Warehouse)

直接使用 `Goods In Transit`（在途庫存倉）來當作線邊倉是**不恰當的**，因為這會混淆庫存報表的意義。

**正確做法：**
在 `所有仓库 - SKT` 群組下，新建一個實體倉庫。

- **倉庫名稱 (Warehouse Name):** `线边仓 - SKT`
- **倉庫類型 (Warehouse Type):** Warehouse
- **隸屬於 (Parent Warehouse):** `All Warehouses - SKT`

### 2. 優化後的倉庫結構圖

```
All Warehouses - SKT (集團總倉)
  ├─ 物料仓 - SKT (原物料倉)
  ├─ 线边仓 - SKT (線邊倉)  <-- 新增
  ├─ 在制程仓 - SKT (在製品倉)
  ├─ 成品仓 - SKT (成品倉)
  └─ 在途仓 - SKT (在途庫存倉)
```

### 3. 優化後的物料流程

1. **電線備料 (Stock Transfer):**
   - **從：** `物料仓 - SKT`（原物料倉）
   - **到：** `线边仓 - SKT`（線邊倉）
   - *此步驟清晰地記錄了為了生產而進行的「內部備料」行為。*

2. **線束生產領料 (Work Order & Stock Entry):**
   - 在線束的 BOM 中，將「電線」的**來源倉庫 (Source Warehouse)** 指定為 `线边仓 - SKT`。
   - 其他物料的來源倉庫指定為 `物料仓 - SKT`。
   - 生產時，系統會自動從正確的倉庫發料至 `在制程仓 - SKT`。

---

## 二、 自動化設定發料倉 (Client Script)

為了避免在建立 BOM 時需要人工為每種物料手動選擇發料倉，我們可以使用「客戶端腳本」，讓系統根據**物料組 (Item Group)** 來自動填入預設的發料倉。

### 1. 腳本目標

當使用者在 BOM 的物料清單中選擇一個物料時：
- 如果該物料的**物料組**是 `Cable Products`，則自動將該行的「發料倉」設定為 `线边仓 - SKT`。
- 如果該物料的**物料組**是 `Raw Material`，則自動設定為 `物料仓 - SKT`。

### 2. 操作步驟（需系統管理員權限）

1. 在 ERPNext 的全域搜尋列輸入 `Client Script List` 並進入列表。
2. 點擊右上角的 **「Add Client Script」**。
3. 在表單中填入以下資訊：
   - **Script Name:** `BOM自动规划发料仓`
   - **DocType**: `BOM`（表示此腳本作用於 BOM 文件類型）
   - **Enabled**: 勾選
4. 將下方的腳本程式碼完整貼入 **Script** 欄位中。
5. 點擊 **Save** 儲存。

### 3. 客戶端腳本 (Client Script) 程式碼

```javascript
frappe.ui.form.on('BOM Item', {
    item_code: function(frm, cdt, cdn) {
        let row = locals[cdt][cdn];
        if (!row.item_code) return;

        // 透過 API 從伺服器取得該料號的 item_group
        frappe.db.get_value('Item', row.item_code, 'item_group')
            .then(r => {
                let item_group = r && r.message && r.message.item_group;
                let warehouse = null;

                if (item_group === 'Cable Products') {
                    warehouse = '线边仓 - SKT';
                } else if (item_group === 'Raw Material') {
                    warehouse = '物料仓 - SKT';
                }
                // 可繼續在此新增其他物料組的規則

                if (warehouse) {
                    // 延遲 600ms 寫入，避免被 ERPNext 的 item fetch 覆蓋
                    setTimeout(() => {
                        frappe.model.set_value(cdt, cdn, 'source_warehouse', warehouse);
                        frm.refresh_field('items');
                    }, 600);
                }
            });
    }
});
```

### 4. 重要說明

#### 為什麼要用 `setTimeout`？

當使用者在 BOM Item 子表選擇品項後，ERPNext 會自動向伺服器發出 fetch 請求，補填品項名稱、UOM、單價等欄位。這個 fetch 動作發生在我們的 `set_value` 之後，會把剛寫入的 `source_warehouse` 清空。

使用 `setTimeout(..., 600)` 延遲 600 毫秒，確保在 ERPNext 的 fetch 完成後才寫入發料倉，避免被覆蓋。

#### 欄位名稱

BOM Item 子表中「發料倉」對應的欄位名稱為 `source_warehouse`（非 `set_source_warehouse`）。

#### API 回應格式

`frappe.db.get_value()` 的回傳格式為：

```json
{ "message": { "item_group": "Cable Products" } }
```

因此必須使用 `r.message.item_group` 取值，不可直接用 `r.item_group`。

---

### 5. 驗證

設定完成後，建立一筆新的 BOM，在物料清單中選擇品項：

| 品項物料組 | 自動填入的發料倉 |
|---|---|
| `Cable Products` | `线边仓 - SKT` |
| `Raw Material` | `物料仓 - SKT` |
| 其他 | 不填入（保持空白） |

---

## 三、 除錯紀錄

### 問題：BOM 建立後沒有自動填入發料倉

**排查過程（按 F12 → Console 觀察）：**

1. **確認觸發是否執行** → Console 有輸出 `BOM Item trigger fired` ✓
2. **確認 API 回應正確** → `r.message.item_group` 有正確回傳物料組 ✓
3. **確認 set_value 有被呼叫** → Console 有輸出 `set_value called` ✓
4. **但欄位仍然空白** → ERPNext 的 item fetch 在 `set_value` 後執行，把值清空

**根本原因：** ERPNext 在 `item_code` 被設定後，會觸發 server-side fetch 補填品項資訊，此過程覆蓋了 `source_warehouse`。

**解決方案：** 加入 `setTimeout(..., 600)` 延後寫入，等待 ERPNext fetch 完成後再設值。

---

*文件建立日期：2026-04-15*  
*最後更新：2026-04-16*  
*適用版本：ERPNext v16 / Frappe v16*
