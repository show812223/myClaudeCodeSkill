---
name: devops-task
description: Azure DevOps Task 自動化工具組。包含三個指令：/create-task（自然語言建立 Task）、/reply-task（掃描程式碼回覆 Task Comment）、/resolve-task（直接修改程式碼解決 Task）。觸發詞：create-task、reply-task、resolve-task、建立 task、回覆 task、解決 task。
---

# DevOps Task — Azure DevOps Task 自動化工具組

## 概述

包含三個指令，覆蓋 Task 的完整生命週期：

| | `/create-task` | `/reply-task` | `/resolve-task` |
|---|---|---|---|
| **做什麼** | 建立新 Task | 回覆說明（唯讀） | 動手解決（改程式碼） |
| **產出** | 新的 Work Item | Task Comment | 修改後的程式碼 |
| **動 Azure DevOps** | ✅ 建立 Work Item | ✅ 寫入 Comment | ❌ 不動 |
| **動程式碼** | ❌ 不動 | ❌ 不動 | ✅ 修改 |
| **典型場景** | 快速開 Task/Bug | 前端問 API 怎麼用 | Bug 修復、實作功能 |

## 前置需求

- **Azure DevOps MCP Server** 已設定（`@azure-devops/mcp`）
- MCP Domain 至少啟用：`core`、`work-items`

---

# Part 1：reply-task（回覆說明）

## 使用方式

```bash
claude /reply-task 12345
claude /reply-task 12345,12346,12347
```

## 核心流程

```
讀取 Task → 分類問題 → 掃描程式碼 → 組合回覆 → 回寫 Comment
```

### Step 1：讀取 Task

使用 Azure DevOps MCP `get_work_item` 取得：

- **標題**（Title）
- **描述**（Description，通常是 HTML）
- **類型**（Type：Task / Bug / User Story）
- **狀態**（State）
- **指派對象**（Assigned To）
- **關聯的 PR 或 Commit**（如果有 Links）
- **現有 Comments**（了解上下文，避免重複回答）

### Step 2：分類問題類型

根據 Task 內容智能判斷，屬於以下哪一種（或多種組合）：

| 類型 | 典型關鍵字 / 特徵 | 對應策略 |
|------|-------------------|---------|
| **API 規格** | 「需要 API」「端點」「接口」「怎麼呼叫」「參數」「回傳格式」 | → `reply-strategy-api` |
| **Bug 排查** | Bug 類型的 Work Item、「錯誤」「異常」「500」「回傳不對」「壞掉」 | → `reply-strategy-bug` |
| **Model 定義** | 「欄位」「資料結構」「DTO」「Model」「schema」「型別定義」 | → `reply-strategy-model` |
| **通用問題** | 以上都不符合、或複合型問題 | → `reply-strategy-general` |

**分類規則：**

1. 先看 Work Item Type：如果是 `Bug`，優先進入 Bug 排查策略
2. 再看內容關鍵字匹配
3. 如果同時符合多種，合併執行（例如「這個 API 回傳的欄位定義是什麼」= API + Model）
4. 無法判斷時，走通用策略：盡可能從程式碼中找到相關資訊回覆

---

### reply-strategy-api：API 規格回覆

**觸發條件：** 前端/其他團隊詢問 API 端點、參數用法、回傳格式

**掃描目標：**
```
**/Controllers/**/*Controller.cs
**/Api/**/*Controller.cs
**/Endpoints/**/*.cs
**/Program.cs（Minimal API）
```

**提取資訊：**
- Route 路徑（組合 Controller Route + Action Route）
- HTTP Method（GET/POST/PUT/DELETE/PATCH）
- Request 參數（[FromBody]、[FromQuery]、[FromRoute]、[FromForm]）
- Response 型別（ActionResult<T>、[ProducesResponseType]）
- Authorization 需求（[Authorize]、[AllowAnonymous]）

**同時解析 DTO：**
- 追蹤 Request/Response 涉及的 class/record
- 搜尋路徑：`**/Models/**`、`**/Dtos/**`、`**/Requests/**`、`**/Responses/**`
- 展開屬性列表，標記必填/選填
- 處理泛型包裝（`ApiResponse<T>`、`PagedResult<T>`）

**回覆格式：** 參見 `references/comment-template.md` 的 API 模板

---

### reply-strategy-bug：Bug 排查回覆

**觸發條件：** Work Item 類型為 Bug、或描述中包含錯誤現象

**排查步驟：**

1. **定位相關程式碼**
   - 從 Bug 描述提取：API 路徑、錯誤訊息、功能名稱
   - 搜尋對應的 Controller/Service/Repository

2. **分析可能原因**
   - 檢查該端點的邏輯流程
   - 查看 try-catch、validation、null check
   - 檢查 Entity Framework 查詢（N+1、missing Include、錯誤的 Where 條件）
   - 檢查 AutoMapper Profile（是否有遺漏的欄位映射）

3. **檢查近期變更**（如果能取得 git 資訊）
   - `git log --oneline -10 -- {相關檔案}`

**回覆格式：** 參見 `references/comment-template.md` 的 Bug 模板

---

### reply-strategy-model：Model / 欄位定義回覆

**觸發條件：** 詢問資料結構、欄位定義、DTO 規格

**掃描目標：**
```
**/Models/**/*.cs       **/Entities/**/*.cs
**/Dtos/**/*.cs         **/DTOs/**/*.cs
**/ViewModels/**/*.cs   **/Contracts/**/*.cs
**/Requests/**/*.cs     **/Responses/**/*.cs
```

**提取資訊：**
- 類別名稱、命名空間
- 所有屬性：名稱、型別、是否 Nullable、Data Annotation
- 繼承關係（base class 的屬性也要列）
- Entity 與 DTO 的對照關係（透過 AutoMapper Profile）
- Validation 規則

**回覆格式：** 參見 `references/comment-template.md` 的 Model 模板

---

### reply-strategy-general：通用回覆

**觸發條件：** 無法歸類、或複合型問題

**處理方式：**
1. 從 Task 描述提取所有可辨識的技術關鍵字
2. 在專案中全文搜尋相關程式碼
3. 整理找到的相關檔案和程式碼片段
4. 以自然語言回覆，附上程式碼引用

---

### Comment 回寫規則

- 內容簡短（< 500 字）→ 單則 Comment
- 內容較長（500-2000 字）→ 單則 Comment，使用折疊區塊
- 內容很長（> 2000 字）→ 分成「摘要」+「詳細」兩則 Comment

每則 Comment 都包含：
- **開頭**：狀態 emoji + 標題
- **引用**：`> Task #{ID}: {標題}`
- **結尾**：`> 由 Claude Code 自動產生 | {日期}`

使用 Azure DevOps MCP 的 `add_work_item_comment` 工具回寫。

---

# Part 2：resolve-task（程式碼修改）

## 使用方式

```bash
claude /resolve-task 12345
claude /resolve-task 12345,12346
claude /resolve-task 12345 只改 Service 層，不動 Controller
```

**不會做的事：** 不更新 Task 狀態、不加 Comment、不建立 Branch/PR。純粹改程式碼。

## 核心流程

```
讀取 Task → 分類問題 → 分析程式碼 → 擬定方案 → 執行修改 → 驗證 → 回報
```

### Step 1：讀取 Task

使用 Azure DevOps MCP `get_work_item` 取得：

- **標題**（Title）— 快速理解問題
- **描述**（Description）— 詳細需求 / 重現步驟 / 預期行為
- **類型**（Type）— Bug / Task / User Story
- **Acceptance Criteria**（驗收條件，如果有）
- **現有 Comments** — 可能包含補充資訊或討論結果
- **關聯 Work Items** — 理解上下文

### Step 2：分類問題類型

| 類型 | 觸發條件 | 執行策略 |
|------|---------|---------|
| **🐛 Bug 修復** | Type=Bug、描述含錯誤現象、重現步驟 | → `resolve-strategy-bugfix` |
| **✨ 新功能** | Type=Task/Story、描述含需求規格、API 設計 | → `resolve-strategy-feature` |
| **♻️ 重構優化** | 描述含「重構」「優化」「效能」「整理」 | → `resolve-strategy-refactor` |
| **🧪 補寫測試** | 描述含「測試」「test」「coverage」「驗證」 | → `resolve-strategy-test` |
| **🔀 複合型** | 同時符合多種 | 依主要目標排序，依序執行 |

---

### resolve-strategy-bugfix：Bug 修復

**目標：** 定位問題根因，修改程式碼修復，確保不影響既有功能

**Step 1 — 定位**
1. 從 Bug 描述提取線索：API 路徑 / 錯誤訊息 / 堆疊追蹤
2. 搜尋相關程式碼（參見 `references/parsing-patterns.md`）
3. 建立呼叫鏈：Controller → Service → Repository → Entity

**Step 2 — 分析**
1. 追蹤有問題的程式碼路徑
2. 檢查常見問題 Pattern（參見 `references/resolve-patterns.md`）：
   - Null Reference / 缺少 null check
   - EF Core 查詢問題（N+1、missing Include、錯誤的 Where）
   - AutoMapper 映射遺漏
   - 型別轉換錯誤、併發問題、Validation 不完整
3. 確認根因，評估影響範圍

**Step 3 — 修復**
1. 以最小變更修復問題（不附帶不相關的重構）
2. 修改時遵循專案既有的 coding style
3. 加上必要的 null check、validation、error handling

**Step 4 — 驗證**
1. `dotnet build` — 確認編譯通過
2. `dotnet test` — 確認既有測試不壞
3. 列出所有被修改的檔案

---

### resolve-strategy-feature：新功能實作

**目標：** 根據 Task 需求實作完整功能，遵循專案架構慣例

**Step 1 — 需求分析**
1. 整理要實作的功能、涉及的 Entity / DTO、需要的 API 端點
2. 如果需求不夠清楚，列出假設並提示使用者確認

**Step 2 — 架構分析**
1. 掃描專案結構，理解既有的分層架構
2. 找到類似功能作為參考（例如要實作 Order CRUD，先看 User CRUD 怎麼寫）
3. 確認使用的 Pattern（Repository Pattern? CQRS? 直接用 DbContext?）

**Step 3 — 實作**

按照以下順序實作（由底層往上）：

1. **Entity / Model** — 建立 Entity、EF Configuration、Request/Response DTO、AutoMapper Profile
2. **DbContext** — 加入 `DbSet<T>`（**不自動產生 Migration**，提醒使用者手動執行）
3. **Service 層** — Interface + 實作 + 商業邏輯
4. **Controller** — Route、HTTP Method、注入 Service、Authorization、`[ProducesResponseType]`
5. **DI 註冊** — 在 `Program.cs` 註冊新 Service

**Step 4 — 驗證**
1. `dotnet build`、確認 DI 註冊完整
2. 列出新增/修改的檔案
3. 提醒手動步驟（Migration、設定檔等）

---

### resolve-strategy-refactor：重構 / 優化

**目標：** 改善程式碼品質或效能，不改變外部行為

**Step 1 — 範圍確認**
- 效能優化（查詢、快取、演算法）
- 程式碼品質（重複程式碼、過長方法、命名不佳）
- 架構調整（抽 Service、引入 Pattern）

**Step 2 — 影響分析**
- 分析所有呼叫端
- 確認測試覆蓋率
- 評估風險

**Step 3 — 重構執行**

遵循原則：
- 一次只做一件事
- 保持行為不變
- 小步前進

常見手法（詳見 `references/resolve-patterns.md`）：
- 提取方法 / 提取 Service
- 消除重複（DRY）
- EF Core 查詢優化（Select projection、AsNoTracking）

**Step 4 — 驗證**
- `dotnet build` + `dotnet test`（**所有既有測試必須通過**）

---

### resolve-strategy-test：補寫測試

**目標：** 為指定功能補上測試覆蓋

**Step 1 — 分析測試基礎設施**
- 找到測試專案（`*.Tests.csproj`）
- 確認框架：xUnit / NUnit / MSTest
- 確認 Mock 框架：Moq / NSubstitute / FakeItEasy
- 找到現有測試作為範本

**Step 2 — 撰寫測試**

命名慣例依循專案既有風格，若無則用 `MethodName_Scenario_ExpectedResult`。

覆蓋要點：
- 正常路徑（Happy Path）
- 邊界條件（Null、Empty、Max/Min）
- 錯誤處理（Not Found、Unauthorized、Validation Error）
- Bug 回歸測試（如果是修 Bug 的延伸）

**Step 3 — 驗證**
- `dotnet test` — 新測試全部通過，既有測試不壞

---

---

# Part 3：create-task（建立 Task）

## 使用方式

```bash
# 自然語言描述需求
claude /create-task 建立使用者登入 API，支援帳號密碼和 OAuth

# 建立 Bug
claude /create-task [bug] 使用者清單 API 分頁參數無效時回傳 500

# 簡短需求
claude /create-task 加上使用者大頭照上傳功能

# 指定專案
claude /create-task [project:MyProject] 重構 Order Service 的查詢效能
```

**不會做的事：** 不修改程式碼、不指派人員、不設定 Sprint。純粹建立 Work Item。

## 核心流程

```
解析輸入 → 判斷類型 → 分析複雜度 → 產生標題+描述 → 確認 → 建立 Work Item
```

### Step 1：解析輸入

從使用者的自然語言輸入中提取：

- **Work Item Type 標記**（可選）：
  - `[bug]` → Bug
  - `[task]` → Task（預設）
  - `[story]` 或 `[us]` → User Story
  - 未標記 → 從內容自動判斷（描述錯誤現象 → Bug，其他 → Task）
- **專案標記**（可選）：`[project:ProjectName]`，未指定則使用預設專案
- **需求描述**：去除標記後的剩餘文字

### Step 2：判斷 Work Item Type

如果使用者沒有明確標記，根據內容自動判斷：

| 判斷為 Bug | 判斷為 Task |
|-----------|-----------|
| 包含「錯誤」「異常」「壞掉」「500」「crash」 | 包含「建立」「實作」「新增」「加上」 |
| 包含「回傳不對」「資料錯誤」「無法正常」 | 包含「重構」「優化」「調整」「更新」 |
| 描述的是「目前的問題」 | 描述的是「要做的事」 |

### Step 3：分析複雜度，決定描述深度

根據輸入內容的複雜度，自動選擇描述詳細程度：

**簡單需求**（描述 < 20 字、明確單一功能）→ 精簡描述（1-2 段自然語言）

**中等需求**（有具體規格或多個面向）→ 標準描述（需求說明 + 預期行為 + Acceptance Criteria）

**複雜需求**（涉及多元件、系統設計、技術約束）→ 詳細描述（需求 + 技術規格 + 影響範圍 + AC + 注意事項）

描述格式模板參見 `references/task-template.md`。

### Step 4：確認內容

**在建立之前，必須先輸出產生的內容讓使用者確認：**

```
══════════════════════════════════════════
  準備建立 Work Item
══════════════════════════════════════════

📋 類型: Task
📁 專案: MyProject
📌 標題: 建立使用者登入 API（帳號密碼 + OAuth）

📝 描述:
   {完整描述內容}

══════════════════════════════════════════
確認建立？(Y/n)
```

使用者確認後才執行建立。

### Step 5：建立 Work Item

使用 Azure DevOps MCP `create_work_item` 建立：

- **Type**：Task / Bug / User Story
- **Title**：產生的標題
- **Description**：產生的描述（HTML 格式）

建立成功後輸出：

```
✅ 已建立 Task #12345: 建立使用者登入 API（帳號密碼 + OAuth）
🔗 https://dev.azure.com/YourOrg/Project/_workitems/edit/12345
```

---

## 描述產生規則

### 標題規則

- **簡潔明確**：不超過 80 字
- **動詞開頭**：「建立」「實作」「修復」「重構」「新增」「調整」
- **包含範圍**：涉及的模組或功能名稱
- **Bug 標題**：描述問題現象，不是原因

### 智能展開

當使用者提到技術關鍵字時，自動展開相關細節：

| 關鍵字 | 自動展開 |
|--------|---------|
| API | 建議的 HTTP Method + Route |
| CRUD | 列出 Create/Read/Update/Delete 四個端點 |
| 分頁 | 提及 PagedResult、page/pageSize 參數 |
| 上傳 | 提及 IFormFile、檔案大小限制 |
| 認證/登入 | 提及 JWT Token、OAuth、Authorization |
| 權限 | 提及 Role-based、Policy-based |
| 快取 | 提及 IMemoryCache / Redis |

### 掃描專案輔助（可選）

如果在專案目錄下執行，可以掃描專案程式碼來豐富描述：

- 找到相關的 Entity/Model → 在描述中引用現有的資料結構
- 找到類似功能 → 參考其架構在描述中建議實作方式
- 找到相關的 Controller → 確認 Route 命名慣例

**這是可選的輔助，不是必要步驟。**

---

## 通用規則（三個指令共用）

### 搜尋路徑

依照以下順序尋找專案根目錄：
1. 當前工作目錄下的 `*.sln` 或 `*.csproj`
2. `src/` 目錄
3. 直接在當前目錄搜尋

### 排除路徑

```
**/bin/**          **/obj/**
**/Migrations/**   **/Tests/**
**/test/**         **/*.Tests/**
**/node_modules/**
**/*.g.cs          **/*.Designer.cs
**/wwwroot/**
```

### 程式碼風格（resolve-task 適用）

自動偵測專案風格，而非強加自己的風格：
1. 檢查 `.editorconfig`
2. 觀察既有程式碼（花括號、var vs 明確型別、file-scoped namespace 等）
3. 新增的程式碼必須與既有風格一致

### 錯誤處理慣例（resolve-task 適用）

偵測專案使用的 Error Handling Pattern：
- Exception-based → 使用相同的自訂 Exception
- Result Pattern → 用相同的 Result wrapper
- ProblemDetails → 產生一致的 ProblemDetails

### 不自動做的事（resolve-task 適用）

| 動作 | 原因 |
|------|------|
| `dotnet ef migrations add` | Migration 名稱需要人工確認 |
| 修改 `appsettings.json` | 涉及環境設定 |
| 修改 CI/CD Pipeline | 影響部署流程 |
| 刪除檔案 | 需要人工確認 |
| 升級 NuGet 套件版本 | 可能影響其他功能 |

### resolve-task 執行完成後的輸出

在終端機輸出執行摘要（不回寫 Azure DevOps）：

```
══════════════════════════════════════════
  resolve-task 執行結果
══════════════════════════════════════════

📋 Task #12345: 使用者登入 API 回傳 500 錯誤
🏷️  類型: Bug 修復

📝 修改內容:
   ✅ src/Services/AuthService.cs (Line 42)
      — 加入 null check
   ✅ src/Controllers/AuthController.cs (Line 28)
      — 回傳 404 而非讓 exception 冒出

🔨 建置: ✅ 通過
🧪 測試: ✅ 12 passed, 0 failed

⚠️  提醒:
   — 無需 Migration
   — 建議補上 AuthService.LoginAsync 的 unit test

══════════════════════════════════════════
```

---

## 安全守則（resolve-task 適用）

1. **修改前先理解** — 不盲目改程式碼，確認理解問題再動手
2. **最小變更原則** — Bug 修復不附帶重構，新功能不改既有邏輯
3. **不確定就問** — 需求模糊時詢問使用者，不自行猜測
4. **保護既有功能** — 修改後跑測試，確認不破壞既有行為
5. **不碰機密** — 不修改 connection string、API key、密碼等設定值
6. **提醒手動步驟** — Migration、設定檔修改等需要明確提醒

---

## 邊界情況處理（三個指令共用）

| 情況 | 處理方式 |
|------|---------|
| Task 描述為空 | 提示使用者，詢問需求 |
| 無法判斷問題類型 | reply → 走通用策略；resolve → 詢問使用者 |
| 找不到相關程式碼 | 回覆/提示說明未找到 |
| Task 已關閉 | 警告但仍允許執行 |
| MCP 連線失敗 | 提示檢查 MCP 設定和認證狀態 |

---

## 參考檔案

- Comment 回覆模板：`references/comment-template.md`（reply-task 用）
- Task 描述模板：`references/task-template.md`（create-task 用）
- 程式碼解析 Pattern：`references/parsing-patterns.md`（共用）
- 問題修復與實作 Pattern：`references/resolve-patterns.md`（resolve-task 用）
