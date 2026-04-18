# ERPNext 自訂內容版本控制說明書

**建立日期**：2026-04-17

## 背景說明

ERPNext 的自訂腳本（Client Script、Custom Field、Property Setter）存放在 **MariaDB 資料庫**中，
不是一般檔案，無法直接用 Git 追蹤。

本方案透過 `export.py` 將資料庫內容匯出成 JSON 檔案，再用 Git 進行版本控制，
並推送到 GitHub 私有 repo，達到備份與版本管理的目的。

---

## 環境資訊

| 項目 | 內容 |
|------|------|
| ERPNext bench 路徑 | `/home/stanley/frappe-bench` |
| 站台名稱 | `site1.local` |
| 自訂內容 repo 路徑 | `/home/stanley/projects/erpnext-customizations` |
| GitHub 帳號 | `tskerpnext` |
| GitHub Repo | `https://github.com/tskerpnext/erpnext-customizations`（private） |

---

## 一、首次建立流程（已完成，供參考）

### 1. 建立 GitHub 帳號並產生 Personal Access Token

1. 登入 GitHub（支援 Google 登入）
2. 右上角頭像 → **Settings** → 左側最底部 **Developer settings**
3. **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**
4. 填寫：
   - Note：`erpnext-push`
   - Expiration：依需求設定
   - 勾選權限：**repo**（全選）
5. 點 **Generate token** → 複製 token（離開頁面就消失）

> 安全提醒：token 不要貼在對話或文件中，產生後直接在終端機使用。

---

### 2. 在 GitHub 建立私有 Repo

1. GitHub 右上角 **+** → **New repository**
2. Repository name：`erpnext-customizations`
3. 設為 **Private**
4. **不勾選** Initialize README
5. 點 **Create repository**

---

### 3. 設定 Git Credential（本機儲存 token）

在終端機執行，之後不需要重複輸入 token：

```bash
git config --global credential.helper store
cd /home/stanley/projects/erpnext-customizations
git remote set-url origin https://tskerpnext@github.com/tskerpnext/erpnext-customizations.git
git push
# 要求輸入時：
# Username: tskerpnext
# Password: 貼上 token（畫面不會顯示）
```

token 會儲存在 `~/.git-credentials`，之後 push 無需再輸入。

---

### 4. 首次匯出並建立 Git repo

```bash
mkdir -p /home/stanley/projects/erpnext-customizations
cd /home/stanley/projects/erpnext-customizations
git init

# 執行匯出腳本
python3 export.py

git add .
git commit -m "init: 初始匯出 ERPNext 自訂內容"
git remote add origin https://tskerpnext@github.com/tskerpnext/erpnext-customizations.git
git push -u origin master
```

---

## 二、日常版本控制（魔法指令）

每次在 ERPNext 修改腳本後，執行以下三步驟：

```bash
# 步驟 1：匯出最新內容
cd /home/stanley/projects/erpnext-customizations
python3 export.py

# 步驟 2：確認變更內容
git diff --stat

# 步驟 3：commit 並推送
git add .
git commit -m "update: 說明這次改了什麼"
git push
```

### commit 訊息格式參考

| 情境 | 範例 |
|------|------|
| 修改腳本邏輯 | `update: BOM發料倉改為優先選原料倉` |
| 新增腳本 | `feat: 新增Work Order自動填倉腳本` |
| 修改欄位 | `update: Work Order新增Section1 Date欄位` |
| 刪除腳本 | `remove: 移除舊版Sales Order選倉邏輯` |

### Claude Code CLI 魔法指令

直接對 Claude 說：

> **「幫我匯出ERPNext自訂內容並push，這次改了 [說明內容]」**

Claude 會自動執行全部步驟。

---

## 三、遷移到正式機流程

### 前置條件

正式機需已安裝 ERPNext（frappe-bench），並確認：
- bench 路徑（預設 `/home/frappe/frappe-bench`）
- 站台名稱

### 步驟 1：正式機下載 repo

```bash
git clone https://github.com/tskerpnext/erpnext-customizations.git
cd erpnext-customizations
```

### 步驟 2：修改 import.py 設定

編輯 `import.py` 開頭兩行，填入正式機的實際路徑：

```python
BENCH_PATH = '/home/frappe/frappe-bench'  # ← 改成正式機路徑
SITE_NAME  = 'site1.local'               # ← 改成正式機站台名稱
```

### 步驟 3：執行匯入

```bash
# 匯入全部
python3 import.py

# 或只匯入特定類型
python3 import.py client_scripts
python3 import.py custom_fields
python3 import.py property_setters
```

### 步驟 4：重啟 ERPNext 讓設定生效

```bash
cd /home/frappe/frappe-bench
bench --site [站台名稱] clear-cache
bench restart
```

---

## 四、repo 目錄結構說明

```
erpnext-customizations/
├── client_scripts/         # Client Script（JS 邏輯腳本）
│   ├── BOM自动规划发料仓.json
│   └── Sales_Order-自動選倉.json
├── custom_fields/          # 自訂欄位
│   └── *.json
├── property_setters/       # 表單屬性設定
│   └── *.json
├── export.py               # 從開發機匯出用
└── import.py               # 部署到正式機用
```

---

## 五、token 過期處理

token 到期後需重新產生並更新本機設定：

```bash
# 刪除舊的 credential
git credential reject <<EOF
protocol=https
host=github.com
EOF

# 重新執行一次 push，輸入新 token
cd /home/stanley/projects/erpnext-customizations
git push
```
