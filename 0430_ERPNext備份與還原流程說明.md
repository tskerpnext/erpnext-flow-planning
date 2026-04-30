# ERPNext 備份與還原流程說明

建立日期：2026-04-30

---

## 備份倉庫

- GitHub Repo：`https://github.com/tskerpnext/erpnext-backups`（私有）
- 資料夾命名規則：`yyyyddmm`（例：2026-04-30 → `20263004`）

---

## 兩種備份的差異

| 備份類型 | 存放位置 | 備份內容 | 還原範圍 |
|---------|---------|---------|---------|
| 自訂設定備份 | `yyyyddmm/` | Client Scripts、Custom Fields、Property Setters、Print Formats、Server Scripts、翻譯、稅務、付款條件等 JSON 檔 | 只還原自訂設定，不含任何業務資料 |
| 完整站台備份 | `private-backups/yyyyddmm/` | 完整資料庫 + 上傳檔案 | 還原整個站台，含所有單據、庫存、客戶、使用者等 |

---

## 建立自訂設定備份

```bash
# 1. 匯出最新自訂設定
source /home/stanley/frappe-bench/env/bin/activate
cd /home/stanley/projects/erpnext-customizations
python3 export.py

# 2. 複製到備份 repo（yyyyddmm 資料夾）
cp -r /home/stanley/projects/erpnext-customizations/* \
      /home/stanley/erpnext-backups/yyyyddmm/customizations/

# 3. 推送
cd /home/stanley/erpnext-backups
git add .
git commit -m "backup: yyyyddmm 自訂設定備份"
git push
```

---

## 建立完整站台備份

```bash
# 1. 執行 bench backup（含上傳檔案）
cd /home/stanley/frappe-bench
bench --site site1.local backup --with-files

# 備份檔案產生於：
# sites/site1.local/private/backups/

# 2. 複製到備份 repo
mkdir -p /home/stanley/erpnext-backups/private-backups/yyyyddmm
cp sites/site1.local/private/backups/日期前綴* \
   /home/stanley/erpnext-backups/private-backups/yyyyddmm/

# 3. 推送
cd /home/stanley/erpnext-backups
git add private-backups/
git commit -m "backup: yyyyddmm 完整站台備份"
git push
```

---

## 還原自訂設定

適用情境：測試新腳本或欄位後效果不理想，只需復原自訂設定。

```bash
source /home/stanley/frappe-bench/env/bin/activate
cd /home/stanley/erpnext-backups/yyyyddmm

python3 import.py                    # 還原全部（僅支援以下 4 類）
python3 import.py client_scripts
python3 import.py custom_fields
python3 import.py property_setters
python3 import.py print_formats

bench --site site1.local clear-cache
bench restart
```

> **注意：** `server_scripts`、`translations`、`tax_categories`、`sales_tax_templates`、`item_tax_templates`、`payment_terms`、`payment_terms_templates` 需手動從 ERPNext UI 匯入。

---

## 還原完整站台

適用情境：測試新功能後資料異常，需要完整回到備份當下狀態（含所有業務資料）。

> ⚠️ 還原會覆蓋現有資料庫，所有備份後的變更都會消失。

```bash
cd /home/stanley/frappe-bench

bench --site site1.local restore \
  /home/stanley/erpnext-backups/private-backups/yyyyddmm/*-database.sql.gz \
  --with-public-files  /home/stanley/erpnext-backups/private-backups/yyyyddmm/*-files.tar \
  --with-private-files /home/stanley/erpnext-backups/private-backups/yyyyddmm/*-private-files.tar

bench --site site1.local clear-cache
bench restart
```

---

## 測試新功能的標準流程

1. **測試前**：先建立完整站台備份（`bench backup --with-files`）並推送
2. **進行測試**：在 ERPNext 中操作
3. **效果不理想**：
   - 只動了腳本/欄位 → 用 `import.py` 還原自訂設定
   - 有建立測試單據或庫存異動 → 用 `bench restore` 完整還原
4. **效果滿意**：更新自訂設定備份並推送

---

## 備份紀錄

| 資料夾 | 日期 | 說明 |
|--------|------|------|
| `20263004/` | 2026-04-30 | 自訂設定初版備份（199 檔） |
| `private-backups/20263004/` | 2026-04-30 | 完整站台備份（DB 1.9MiB） |
