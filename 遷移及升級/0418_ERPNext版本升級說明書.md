# ERPNext 版本升級說明書

**建立日期**：2026-04-18

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
| 升級前版本 | Frappe v16.14.0 / ERPNext v16.13.3 |
| 升級後版本 | Frappe v16.15.0 / ERPNext v16.14.0 |

---

## 官方最新版本查詢

- Frappe Framework：https://github.com/frappe/frappe/releases
- ERPNext：https://github.com/frappe/erpnext/releases

> 兩者版本號同步，目前皆在 version-16 分支滾動更新。

---

## 升級流程

### 步驟 1：確認最新版本

```bash
bench version
```

### 步驟 2：停用排程器（可選）

```bash
bench --site all disable-scheduler
```

### 步驟 3：處理 ERPNext 自訂修改（重要）

ERPNext app 內有自訂的 work_order 等修改，升級前需先暫存：

```bash
cd ~/frappe-bench/apps/erpnext
git stash
```

### 步驟 4：處理未追蹤的翻譯目錄（重要）

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

`bench restart` 及 `bench update --restart-supervisor` 無法重啟 systemd 管理的 gunicorn，
網頁版本號不會更新，**必須手動執行**：

```bash
sudo systemctl restart frappe-bench-frappe-web.service
```

確認服務狀態：

```bash
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
| `LANG=C` | 必加，否則中文 git 輸出導致 bench 誤判本地變更 |
| `translations/` 目錄 | 升級前移走，升級後移回，不影響系統功能 |
| `git stash` | 保護 erpnext 自訂修改（work_order 等），升級後 `git stash pop` 還原 |
| systemd 重啟 | `bench restart` 無效，需 `sudo systemctl restart frappe-bench-frappe-web.service` |
| 版本顯示 | 確認需使用**全新瀏覽器或無痕視窗**，並在 systemd 重啟後才會更新 |
| frappe app | 若 frappe 有未 commit 的自訂，先 `git reset --hard upstream/version-16` 再升級 |

---

## 快速指令總覽

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
