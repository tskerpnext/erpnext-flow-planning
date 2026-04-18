# ERPNext 流程規劃文件集

本 Repository 收錄 ERPNext 實務導入與應用的流程規劃文件，適用於製造業情境。
文件以 Markdown 格式撰寫，持續更新。

---

## 文件列表

| 文件 | 說明 | 日期 |
|------|------|------|
| [框架訂單與生產工單流程規劃](0418_框架訂單與生產工單流程規劃.md) | 客戶提供月度銷售預測，透過 Blanket Order 管理，串聯 Sales Order、Production Plan、Work Order 的完整流程與自訂設定 | 2026-04-18 |
| [ERPNext 魔法指令說明](0418_ERPNext魔法指令說明.md) | Claude Code 魔法指令說明：匯出自訂內容 push、流程規劃文件 push，含完整路徑與操作步驟 | 2026-04-18 |

### 遷移及升級

| 文件 | 說明 | 日期 |
|------|------|------|
| [ERPNext 自訂內容版本控制說明書](遷移及升級/0417_ERPNext自訂內容版本控制說明書.md) | ERPNext 自訂內容（Client Script、Custom Field、Property Setter）的版本控制方式與備份還原流程 | 2026-04-17 |
| [需手動還原的程式碼修改清單](遷移及升級/0418_需手動還原的程式碼修改清單.md) | 直接修改原始碼與系統設定檔的清單，升級或遷移時需手動重新套用，含 Nos 整數修正、Socket.IO 修正、Nginx 設定等 | 2026-04-18 |

---

## 適用環境

- **ERPNext 版本：** v14 / v15
- **情境：** 製造業，客戶提供預測性採購計劃

---

## 文件命名規則

所有文件以日期前綴命名：

```
mmdd_文件名稱.md
```

例如：`0418_框架訂單與生產工單流程規劃.md`

---

## 相關 Repository

| Repo | 說明 |
|------|------|
| [erpnext-customizations](https://github.com/tskerpnext/erpnext-customizations) | ERPNext 自訂腳本、Custom Field、Property Setter 版本控制 |

---

## 維護者

GitHub：[tskerpnext](https://github.com/tskerpnext)
