# Task 描述模板

create-task 使用的描述產生模板。根據需求複雜度自動選用。

---

## 精簡描述（簡單需求）

適用條件：描述 < 20 字、明確單一功能

```
標題：{動詞}{功能名稱}
描述：

{一段自然語言描述需求}
```

**範例：**

```
標題：實作使用者大頭照上傳功能
描述：

新增使用者大頭照上傳功能，允許使用者上傳並更新個人頭像。
```

```
標題：新增 Order 匯出 CSV 功能
描述：

在訂單管理頁面新增匯出功能，將訂單清單匯出為 CSV 格式。
```

---

## 標準描述（中等需求）

適用條件：有具體規格、涉及多個面向

```
標題：{動詞}{功能名稱}（{補充規格}）
描述：

## 需求說明
{功能的完整描述，2-3 段}

## 預期行為
- {行為 1}
- {行為 2}
- {行為 3}

## Acceptance Criteria
- [ ] {驗收條件 1}
- [ ] {驗收條件 2}
- [ ] {驗收條件 3}
```

**範例：**

```
標題：建立使用者登入 API（帳號密碼 + OAuth）
描述：

## 需求說明
建立使用者登入 API，支援以下兩種認證方式：
- 帳號密碼登入
- OAuth 第三方登入

## 預期行為
- 帳號密碼登入：驗證 credentials，成功回傳 JWT Token
- OAuth 登入：支援第三方認證流程，成功回傳 JWT Token
- 登入失敗時回傳適當的錯誤訊息和 HTTP Status Code

## Acceptance Criteria
- [ ] POST /api/auth/login 支援帳號密碼登入
- [ ] POST /api/auth/oauth 支援 OAuth 登入
- [ ] 成功登入回傳標準 JWT Token
- [ ] 帳號不存在回傳 404
- [ ] 密碼錯誤回傳 401
- [ ] 參數缺失回傳 400
```

```
標題：實作訂單分頁查詢 API
描述：

## 需求說明
建立訂單分頁查詢 API，支援依狀態篩選、關鍵字搜尋、日期區間篩選。

## 預期行為
- 支援 page / pageSize 參數進行分頁
- 支援 status 參數篩選訂單狀態
- 支援 keyword 參數搜尋訂單編號、客戶名稱
- 支援 startDate / endDate 篩選日期區間
- 回傳包含 totalCount 的分頁結果

## Acceptance Criteria
- [ ] GET /api/orders 支援分頁查詢
- [ ] 支援 status、keyword、startDate、endDate 篩選
- [ ] 預設 pageSize = 20，最大 100
- [ ] 回傳 PagedResult<OrderDto> 格式
```

---

## 詳細描述（複雜需求）

適用條件：涉及多個元件、系統設計、技術約束

```
標題：{動詞}{功能/模組名稱} — {核心變更描述}
描述：

## 需求說明
{功能的完整描述}

## 技術規格

### {元件/層面 1}
- {具體規格}

### {元件/層面 2}
- {具體規格}

## 影響範圍
- {檔案/模組 1}
- {檔案/模組 2}

## Acceptance Criteria
- [ ] {驗收條件 1}
- [ ] {驗收條件 2}

## 注意事項
- {風險/限制/相依性}
```

**範例：**

```
標題：重構 Order 模組 — 導入 CQRS Pattern 實現讀寫分離
描述：

## 需求說明
將 Order 模組從現有架構重構為 CQRS 模式，實現讀寫分離以提升查詢效能。

## 技術規格

### Command 端（寫入）
- 維持現有 EF Core 架構
- 透過 MediatR 處理 Command
- 包含：CreateOrderCommand、UpdateOrderCommand、DeleteOrderCommand

### Query 端（讀取）
- 使用 Dapper 直接執行 SQL 查詢
- 繞過 EF Core 的 Change Tracking 開銷
- 包含：GetOrderQuery、ListOrdersQuery

### DI 調整
- 註冊 MediatR Handler
- 註冊 Dapper 連線

## 影響範圍
- OrderService（拆分為 Command Handler + Query Handler）
- OrderController（注入 IMediator 替代 IOrderService）
- DI 註冊（Program.cs）

## Acceptance Criteria
- [ ] Command 端：建立/更新/刪除 Order 功能正常
- [ ] Query 端：改用 Dapper，查詢效能提升
- [ ] 既有的 API 行為和回傳格式不變（對前端透明）
- [ ] 所有既有測試通過

## 注意事項
- 需要同步維護 EF Core Model 和 Dapper SQL 的欄位一致性
- 考慮是否需要 Read Model / Projection
- Migration 不受影響（仍由 EF Core 管理）
```

---

## Bug 描述

適用條件：Work Item Type = Bug

```
標題：{功能/端點} {問題現象}
描述：

## 問題描述
{錯誤現象的詳細描述}

## 重現步驟（如果有）
1. {步驟 1}
2. {步驟 2}
3. {步驟 3}

## 預期行為
{正確應該是什麼}

## 實際行為
{目前的錯誤行為}
```

**範例：**

```
標題：使用者清單 API 分頁參數無效時回傳 500
描述：

## 問題描述
GET /api/users 傳入無效的分頁參數（page=0 或 pageSize=-1）時，
API 回傳 500 Internal Server Error，而非 400 Bad Request。

## 重現步驟
1. 呼叫 GET /api/users?page=0&pageSize=20
2. 觀察回傳的 HTTP Status Code

## 預期行為
回傳 400 Bad Request，Body 包含參數驗證錯誤訊息。

## 實際行為
回傳 500 Internal Server Error，無有用的錯誤訊息。
```

```
標題：訂單建立後 Email 通知未發送
描述：

## 問題描述
使用者建立新訂單後，確認 Email 未被發送。後台 Log 無相關錯誤紀錄。

## 預期行為
訂單建立成功後，系統自動發送確認 Email 給使用者。

## 實際行為
訂單建立成功，但使用者未收到確認 Email。
```

---

## 標題產生規則

### Task 標題

| 模式 | 範例 |
|------|------|
| {動詞}{功能} | 實作使用者大頭照上傳功能 |
| {動詞}{功能}（{規格}） | 建立使用者登入 API（帳號密碼 + OAuth） |
| {動詞}{模組} — {描述} | 重構 Order 模組 — 導入 CQRS Pattern |

### Bug 標題

| 模式 | 範例 |
|------|------|
| {端點/功能} {問題現象} | 使用者清單 API 分頁參數無效時回傳 500 |
| {功能} {問題} | 訂單建立後 Email 通知未發送 |

### 常用動詞

| 類型 | 動詞 |
|------|------|
| 新功能 | 建立、實作、新增、開發 |
| 改善 | 優化、改善、提升、調整 |
| 重構 | 重構、整理、遷移、替換 |
| 修復 | 修復、修正、解決 |
