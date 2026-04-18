# ERPNext 魔法指令說明

建立日期： 2026-04-18
適用版本： ERPNext v16（已於 2026-04-18 升級至 Frappe v16.15.0 / ERPNext v16.14.0）
發生環境： 本機開發環境 /home/stanley

---

## 說明

以下兩個魔法指令可在任何 Claude Code session 中直接使用，
Claude 會自動執行對應的完整流程，不需再說明路徑或帳號。

---

## 魔法指令一：匯出 ERPNext 自訂內容並推送

### 指令

```
幫我匯出ERPNext自訂內容並push，這次改了 [說明這次修改內容]
```

### 用途

ERPNext 的自訂腳本（Client Script、Custom Field、Property Setter）
存放在 MariaDB 資料庫中，無法直接用 Git 追蹤。
此指令會先用 export.py 將資料庫內容匯出成 JSON 檔案，再 commit 並推送到 GitHub。

### Claude 執行步驟

```bash
cd /home/stanley/projects/erpnext-customizations
/home/stanley/frappe-bench/env/bin/python export.py
git diff --stat
git add .
git commit -m "update: [你說明的修改內容]"
git push
```

> ⚠️ **注意**：export.py 必須使用 bench 的虛擬環境 Python，
> 使用系統 `python3` 會因缺少 `orjson` 等套件而失敗。

### 相關資訊

| 項目 | 內容 |
|------|------|
| 本機路徑 | `/home/stanley/projects/erpnext-customizations` |
| GitHub Repo | `https://github.com/tskerpnext/erpnext-customizations` |
| 可見性 | Private |
| ERPNext bench 路徑 | `/home/stanley/frappe-bench` |
| 站台名稱 | `site1.local` |
| Git 驗證 | `~/.git-credentials`（token 已儲存，無需手動輸入） |

### 使用範例

```
幫我匯出ERPNext自訂內容並push，這次改了 Work Order 新增 Blanket Order 自訂欄位
```

---

## 魔法指令二：推送流程規劃文件

### 指令

```
幫我 push 流程規劃文件
```

或

```
幫我 push 流程規劃文件，這次新增了 [文件說明]
```

### 用途

將 `/home/stanley/Documents/MyVault/_ERPNEXT/流程規劃/` 資料夾下的
Markdown 文件推送到 GitHub 公開 Repository，方便分享與備份。

### Claude 執行步驟

```bash
cd /home/stanley/Documents/MyVault/_ERPNEXT/流程規劃
git add .
git commit -m "feat/docs: [你說明的內容，或 Claude 自動判斷]"
git push
```

### 相關資訊

| 項目 | 內容 |
|------|------|
| 本機路徑 | `/home/stanley/Documents/MyVault/_ERPNEXT/流程規劃` |
| GitHub Repo | `https://github.com/tskerpnext/erpnext-flow-planning` |
| 可見性 | Public |
| Git 驗證 | `~/.git-credentials`（token 已儲存，無需手動輸入） |

### 使用範例

```
幫我 push 流程規劃文件，這次新增了採購流程說明
```

---

## Commit Message 格式參考

| 情境 | 格式 | 範例 |
|------|------|------|
| 修改自訂腳本 | `update: 說明` | `update: BOM發料倉改為優先選原料倉` |
| 新增自訂腳本 | `feat: 說明` | `feat: 新增Work Order自動填倉腳本` |
| 新增流程文件 | `feat: 說明` | `feat: 新增採購入庫流程規劃文件` |
| 更新流程文件 | `docs: 說明` | `docs: 更新框架訂單流程補充報表設定` |
| 初始建立 | `init: 說明` | `init: 初始匯入ERPNext自訂內容` |

---

## 注意事項

- commit message 可由你指定，或省略讓 Claude 自動判斷（push 前會告知確認）
- token 存於 `~/.git-credentials`，若 push 失敗請確認 token 是否過期
- token 過期處理方式請參考：`0417_ERPNext自訂內容版本控制說明書.md`
