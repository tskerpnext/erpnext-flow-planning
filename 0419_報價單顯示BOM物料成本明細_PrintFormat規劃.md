# 報價單顯示 BOM 物料成本明細 — Print Format 規劃

日期：2026-04-19

---

## 需求說明

報價單品項中，若該品項有對應的 BOM，希望在列印時能展開顯示 BOM 的**原物料明細與成本數字**，讓客戶或業務人員能清楚了解報價組成。

### 確認範圍

| 項目 | 結論 |
|------|------|
| 顯示工序工時 | 否 |
| 顯示成本數字 | 是 |
| 多層 BOM 展開 | 否（單層） |

---

## 背景說明

### UOM 說明

ERPNext 品項主檔有三個單位欄位：

| 欄位 | 說明 |
|------|------|
| Stock UOM | 庫存單位（基準，必填） |
| Sales UOM | 預設銷售單位（選填） |
| Purchase UOM | 預設採購單位（選填） |

若 Sales UOM 有填，報價單會優先使用；若空白，則回退使用 Stock UOM。

---

## 實作方式

**採用：Custom Print Format（Jinja 模板）**

不修改資料結構，僅調整列印呈現邏輯，風險低。

### 資料來源對應

```
Quotation Item → BOM No
                    └── BOM Item（原物料明細）
                          item_code / item_name / qty / uom / rate / amount
```

### 前置條件

- 報價單品項的 **BOM No** 欄位須填入對應 BOM
- BOM 須為**已提交（Submitted）**狀態，`raw_material_cost` 才有正確數值
- 若 BOM 單位與報價單位不同，需另行確認換算比例

---

## Print Format 建立步驟

1. 進入 `Settings > Print Format > New`
2. **DocType** 選：`Quotation`
3. **Print Format Type** 選：`Jinja`
4. 貼入以下 HTML 程式碼
5. 儲存後，在報價單列印選單選此 Print Format

---

## 完整程式碼

```html
<style>
  .qtn-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 13px;
    margin-bottom: 20px;
  }
  .qtn-table th {
    background: #f0f0f0;
    padding: 6px 8px;
    border: 1px solid #ccc;
    text-align: left;
  }
  .qtn-table td {
    padding: 5px 8px;
    border: 1px solid #ddd;
  }
  .bom-header {
    background: #fafafa;
    font-weight: bold;
    color: #555;
    font-size: 12px;
  }
  .bom-row td {
    background: #fffef5;
    font-size: 12px;
    padding-left: 20px;
    color: #333;
  }
  .subtotal-row td {
    background: #f5f5f5;
    font-weight: bold;
    text-align: right;
  }
  .text-right { text-align: right; }
</style>

<!-- 報價單表頭 -->
<h4>報價單：{{ doc.name }}</h4>
<p>客戶：{{ doc.customer_name }}　日期：{{ doc.transaction_date }}　幣別：{{ doc.currency }}</p>

<hr>

<table class="qtn-table">
  <thead>
    <tr>
      <th style="width:5%">#</th>
      <th style="width:25%">品項</th>
      <th style="width:20%">說明</th>
      <th style="width:10%" class="text-right">數量</th>
      <th style="width:8%">單位</th>
      <th style="width:12%" class="text-right">單價</th>
      <th style="width:12%" class="text-right">金額</th>
    </tr>
  </thead>
  <tbody>

  {% for item in doc.items %}

    {# 主品項列 #}
    <tr>
      <td>{{ loop.index }}</td>
      <td>{{ item.item_code }}</td>
      <td>{{ item.item_name }}</td>
      <td class="text-right">{{ item.qty }}</td>
      <td>{{ item.uom }}</td>
      <td class="text-right">{{ frappe.utils.fmt_money(item.rate, currency=doc.currency) }}</td>
      <td class="text-right">{{ frappe.utils.fmt_money(item.amount, currency=doc.currency) }}</td>
    </tr>

    {# BOM 原物料明細 #}
    {% if item.bom_no %}
      {% set bom = frappe.get_doc("BOM", item.bom_no) %}

      <tr>
        <td colspan="7" class="bom-header">
          &nbsp;&nbsp;▼ 原物料明細（BOM：{{ item.bom_no }}）
        </td>
      </tr>

      {% for bom_item in bom.items %}
      <tr class="bom-row">
        <td></td>
        <td>{{ bom_item.item_code }}</td>
        <td>{{ bom_item.item_name }}</td>
        <td class="text-right">{{ bom_item.qty }}</td>
        <td>{{ bom_item.uom }}</td>
        <td class="text-right">{{ frappe.utils.fmt_money(bom_item.rate, currency=doc.currency) }}</td>
        <td class="text-right">{{ frappe.utils.fmt_money(bom_item.amount, currency=doc.currency) }}</td>
      </tr>
      {% endfor %}

      <tr class="subtotal-row">
        <td colspan="6">原物料小計</td>
        <td>{{ frappe.utils.fmt_money(bom.raw_material_cost, currency=doc.currency) }}</td>
      </tr>

    {% endif %}

  {% endfor %}

  </tbody>
  <tfoot>
    <tr>
      <td colspan="6" style="text-align:right; font-weight:bold;">合計</td>
      <td style="text-align:right; font-weight:bold;">
        {{ frappe.utils.fmt_money(doc.total, currency=doc.currency) }}
      </td>
    </tr>
    {% if doc.discount_amount %}
    <tr>
      <td colspan="6" style="text-align:right;">折扣</td>
      <td style="text-align:right;">{{ frappe.utils.fmt_money(doc.discount_amount, currency=doc.currency) }}</td>
    </tr>
    {% endif %}
    <tr>
      <td colspan="6" style="text-align:right; font-weight:bold;">應付總額</td>
      <td style="text-align:right; font-weight:bold;">
        {{ frappe.utils.fmt_money(doc.grand_total, currency=doc.currency) }}
      </td>
    </tr>
  </tfoot>
</table>
```

---

## 列印輸出格式示意

```
品項：成品A　BOM：BOM-00001

┌────────────────────────────────────────────────────────┐
│ ▼ 原物料明細（BOM：BOM-00001）                           │
│   鋼板　　2 kg　　@ 150　　= 300                         │
│   螺絲　10 pcs　　@ 2　　　= 20                          │
│                              原物料小計：320             │
└────────────────────────────────────────────────────────┘

合計：XXXX
應付總額：XXXX
```

---

## 注意事項

- BOM 須已提交，`raw_material_cost` 才有正確數值
- 若同一報價單有多個品項，每個有 BOM 的品項都會獨立展開
- 此 Print Format 僅影響列印呈現，不修改報價單資料結構
