# DevOps Task Skills

Azure DevOps Task 自動化工具組，包含三個指令，覆蓋 Task 完整生命週期。

## 三個指令的差異

| | `/create-task` | `/reply-task` | `/resolve-task` |
|---|---|---|---|
| **做什麼** | 建立新 Task | 回覆說明（唯讀） | 動手解決（改程式碼） |
| **產出** | 新的 Work Item | Task Comment | 修改後的程式碼 |
| **動 Azure DevOps** | ✅ 建立 Work Item | ✅ 寫入 Comment | ❌ 不動 |
| **動程式碼** | ❌ 不動 | ❌ 不動 | ✅ 修改 |
| **典型場景** | 快速開 Task/Bug | 前端問 API 怎麼用 | Bug 修復、實作功能 |

## 安裝

將檔案放到以下位置：

### 方式一：專案層級（僅該專案使用）

```
your-project/
└── .claude/
    ├── commands/
    │   ├── create-task.md          ← Slash Command
    │   ├── reply-task.md           ← Slash Command
    │   └── resolve-task.md         ← Slash Command
    └── skills/
        └── devops-task/
            ├── SKILL.md            ← 主要 Skill 定義
            └── references/
                ├── comment-template.md
                ├── parsing-patterns.md
                ├── resolve-patterns.md
                └── task-template.md
```

### 方式二：全域（所有專案共用）

```
~/.claude/
├── commands/
│   ├── create-task.md
│   ├── reply-task.md
│   └── resolve-task.md
└── skills/
    └── devops-task/
        └── ...（同上）
```

## 使用

```bash
# 建立 Task（自然語言 → Work Item）
claude /create-task 建立使用者登入 API，支援帳號密碼和 OAuth
claude /create-task [bug] 使用者清單 API 分頁參數無效時回傳 500

# 回覆 Task（回寫 Comment 到 Azure DevOps）
claude /reply-task 12345

# 解決 Task（直接改程式碼）
claude /resolve-task 12345
claude /resolve-task 12345 只改 Service 層，不動 Controller
```

## 前置需求

- Azure DevOps MCP Server 已設定
- `claude mcp add azure-devops -- npx -y @azure-devops/mcp YourOrg -d core work work-items repositories`
