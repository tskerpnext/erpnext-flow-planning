# ERPNext 版本升級說明書

**建立日期**：2026-04-18
**更新日期**：2026-04-29

## 背景說明

ERPNext 與 Frappe Framework 使用 Git branch `version-16` 進行滾動更新。
每次小版本升級（如 v16.13.3 → v16.14.0）需透過 `bench update` 指令完成，
但系統有自訂修改時須先妥善處理，升級後服務也需正確重啟才能反映新版本。

---

## 環境資訊

| 項目 | 內容 |
|------|------|
| bench 路徑 | `/home/stanley/frappe-bench` |
| 站台名稱 | `site1.local` |
| 服務管理方式 | systemd |
| 第一次升級 | Frappe v16.14.0 / ERPNext v16.13.3 → Frappe v16.15.0 / ERPNext v16.14.0 |
| 第二次升級 | Frappe v16.15.0 / ERPNext v16.14.0 → Frappe v16.16.0 / ERPNext v16.15.0 |
| 第三次升級 | Frappe v16.16.0 / ERPNext v16.15.0 → Frappe v16.17.0 / ERPNext v16.16.0 |

---

## 官方最新版本查詢

- Frappe Framework：https://github.com/frappe/frappe/releases
- ERPNext：https://github.com/frappe/erpnext/releases

> 兩者版本號同步，目前皆在 version-16 分支滾動更新。

---

## 升級方式選擇

| 情況 | 建議方式 |
|------|---------|
| apps 目錄無自訂程式碼修改（只有 DB 自訂） | `bench update --reset`（簡單快速） |
| apps 目錄有自訂程式碼需保留（如 work_order 修改） | 傳統方式（git stash + 手動移走 translations/） |

---

## 方式一：bench update --reset（推薦）

適用於 frappe/erpnext app 目錄內**無自訂程式碼修改**的情況。
自訂內容全存在資料庫（Custom Field、Client Script、Server Script 等）不受影響。

### 步驟 1：備份自訂內容

```bash
cd /home/stanley/projects/erpnext-customizations
source /home/stanley/frappe-bench/env/bin/activate
python3 export.py
git add .
git commit -m "backup: 升版前備份"
git push
```

### 步驟 2：執行升級

```bash
cd ~/frappe-bench
bench update --reset
```

`--reset` 會對 frappe 與 erpnext app 執行 `git reset --hard upstream/version-16`，
自動解決：
- shallow clone 造成的 branch divergence（本地比 upstream 多大量 commit）
- 未追蹤檔案（如 `frappe/translations/`）被誤判為本地修改而中止升級

### 步驟 3：重啟 systemd web 服務（關鍵步驟）

```bash
sudo systemctl restart frappe-bench-frappe-web.service
systemctl is-active frappe-bench-frappe-web.service   # 應回傳 active
```

### 步驟 4：確認版本

```bash
bench version
```

---

## 方式二：傳統方式（有自訂 app 程式碼時）

### 步驟 1：確認最新版本

```bash
bench version
```

### 步驟 2：停用排程器（可選）

```bash
bench --site all disable-scheduler
```

### 步驟 3：處理 ERPNext 自訂修改

ERPNext app 內有自訂的 work_order 等修改，升級前需先暫存：

```bash
cd ~/frappe-bench/apps/erpnext
git stash
```

### 步驟 4：處理未追蹤的翻譯目錄

`frappe` 及 `erpnext` 下的 `translations/` 資料夾為未追蹤檔案，
bench 的 git 檢查會因此誤判為「有本地變更」而中止升級，需暫時移走：

```bash
mv ~/frappe-bench/apps/frappe/frappe/translations /tmp/frappe_translations_backup
mv ~/frappe-bench/apps/erpnext/erpnext/translations /tmp/erpnext_translations_backup
```

### 步驟 5：執行升級

> 必須加 `LANG=C`，否則 git status 輸出為中文，bench 無法正確比對導致升級中止。

```bash
cd ~/frappe-bench
LANG=C bench update --pull --patch --build --restart-supervisor
```

### 步驟 6：還原暫存內容

```bash
# 還原翻譯目錄
mv /tmp/frappe_translations_backup ~/frappe-bench/apps/frappe/frappe/translations
mv /tmp/erpnext_translations_backup ~/frappe-bench/apps/erpnext/erpnext/translations

# 還原 erpnext 自訂修改
cd ~/frappe-bench/apps/erpnext
git stash pop
```

> `git stash pop` 後若出現 conflict，需手動解決衝突。

### 步驟 7：重啟 systemd web 服務（關鍵步驟）

```bash
sudo systemctl restart frappe-bench-frappe-web.service
systemctl is-active frappe-bench-frappe-web.service
```

### 步驟 8：重新啟用排程器

```bash
bench --site all enable-scheduler
```

### 步驟 9：確認版本

```bash
bench version
```

---

## 注意事項

| 項目 | 說明 |
|------|------|
| `--reset` | `git reset --hard upstream/version-16`，自動解決 branch divergence 與 untracked files，無需手動處理 translations/ |
| shallow clone branch divergence | bench 設定 `shallow_clone` 後 unshallow 時本地會多出大量 commit，用 `--reset` 解決 |
| `LANG=C` | 傳統方式必加，否則中文 git 輸出導致 bench 誤判本地變更（`--reset` 方式不需要） |
| `translations/` 目錄 | 傳統方式升級前移走，升級後移回；`--reset` 方式不需手動處理 |
| `git stash` | 保護 erpnext 自訂修改；若 app 內無自訂程式碼則不需要 |
| systemd 重啟 | **兩種方式都必須執行**，`bench restart` 無效，需 `sudo systemctl restart frappe-bench-frappe-web.service` |
| 版本顯示 | 確認需使用**全新瀏覽器或無痕視窗**，並在 systemd 重啟後才會更新 |

---

## 快速指令（方式一：--reset）

```bash
# 1. 備份自訂內容
cd /home/stanley/projects/erpnext-customizations
source /home/stanley/frappe-bench/env/bin/activate
python3 export.py && git add . && git commit -m "backup: 升版前備份" && git push

# 2. 升級
cd ~/frappe-bench && bench update --reset

# 3. 重啟 web 服務
sudo systemctl restart frappe-bench-frappe-web.service

# 4. 確認版本
bench version
```

## 快速指令（方式二：傳統）

```bash
# 1. 暫存修改與翻譯目錄
cd ~/frappe-bench/apps/erpnext && git stash
mv ~/frappe-bench/apps/frappe/frappe/translations /tmp/frappe_translations_backup
mv ~/frappe-bench/apps/erpnext/erpnext/translations /tmp/erpnext_translations_backup

# 2. 升級
cd ~/frappe-bench
LANG=C bench update --pull --patch --build --restart-supervisor

# 3. 還原
mv /tmp/frappe_translations_backup ~/frappe-bench/apps/frappe/frappe/translations
mv /tmp/erpnext_translations_backup ~/frappe-bench/apps/erpnext/erpnext/translations
cd ~/frappe-bench/apps/erpnext && git stash pop

# 4. 重啟 web 服務
sudo systemctl restart frappe-bench-frappe-web.service

# 5. 確認版本
bench version
```
