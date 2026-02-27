# Comment 模板

每種問題類型對應不同的回覆模板。Claude 應根據分類結果選用對應模板，複合型問題則合併多個模板。

---

## API 規格模板

### 標準版（4~10 個端點）

```markdown
## ✅ API 端點資訊

> Task #{TaskID}: {Task 標題}

### 端點清單

| Method | Route | 說明 | Auth |
|---|---|---|---|
| GET | /api/users | 取得使用者清單 | Bearer Token |
| POST | /api/users | 建立使用者 | Admin |

---

### `GET /api/users`

**Query Parameters:**
| 參數 | 型別 | 必填 | 說明 |
|---|---|---|---|
| page | int | | 頁碼，預設 1 |
| pageSize | int | | 每頁筆數，預設 20 |
| keyword | string | | 關鍵字搜尋 |

**Response:** `200 OK`
```json
{
  "data": [
    { "id": 1, "name": "string", "email": "string" }
  ],
  "totalCount": 100,
  "page": 1,
  "pageSize": 20
}
```

**Auth:** Bearer Token

---

### `POST /api/users`

**Request Body:** `CreateUserRequest`
```json
{
  "name": "string (必填, max 100)",
  "email": "string (必填, email 格式)",
  "role": "string (選填, 預設 'user')"
}
```

**Response:** `201 Created`
```json
{
  "id": 1,
  "name": "string",
  "email": "string"
}
```

**Auth:** 需要 Admin 角色

---
> 由 Claude Code 自動產生 | 2026-02-12
```

### 精簡版（≤ 3 個端點）

```markdown
## ✅ API 端點資訊

> Task #{TaskID}: {Task 標題}

| Method | Route | 說明 | Auth |
|---|---|---|---|
| GET | /api/users/{id} | 取得單一使用者 | Bearer Token |

**`GET /api/users/{id}`**
- **Route Param:** `id` (int, 必填)
- **Response:** `200 OK` → `UserDto { id, name, email, role, createdAt }`
- **Auth:** Bearer Token

> 由 Claude Code 自動產生 | 2026-02-12
```

### 摘要版（> 10 個端點，第一則 Comment）

```markdown
## ✅ API 端點資訊（共 {N} 個）

> Task #{TaskID}: {Task 標題}

| # | Method | Route | 說明 |
|---|---|---|---|
| 1 | GET | /api/users | 使用者清單 |
| 2 | POST | /api/users | 建立使用者 |
| ... | ... | ... | ... |

> ⬇️ 各端點詳細規格請見下方 Comment

> 由 Claude Code 自動產生 | 2026-02-12
```

---

## Bug 排查模板

```markdown
## 🔍 Bug 排查結果

> Task #{TaskID}: {Task 標題}

### 問題定位

**相關檔案：**
- `src/Controllers/UsersController.cs` — `GetUserById()` (Line 42-68)
- `src/Services/UserService.cs` — `FindByIdAsync()` (Line 25-40)

### 可能原因

**1. {原因標題}**（可能性：高/中/低）
- 位置：`src/Services/UserService.cs:33`
- 分析：{詳細說明為什麼認為這是原因}
- 程式碼：
```csharp
// 這裡缺少 null check
var user = await _context.Users.FindAsync(id);
return _mapper.Map<UserDto>(user); // user 可能為 null
```

**2. {原因標題}**（可能性：中）
- 位置：{檔案:行號}
- 分析：{說明}

### 建議修復

```csharp
var user = await _context.Users.FindAsync(id);
if (user == null)
    return NotFound($"User {id} not found");
return Ok(_mapper.Map<UserDto>(user));
```

### 近期相關變更

| Commit | 日期 | 作者 | 說明 |
|---|---|---|---|
| abc1234 | 2026-02-10 | Mark | refactor: 重構 User 查詢邏輯 |
| def5678 | 2026-02-08 | John | feat: 新增角色篩選功能 |

> 由 Claude Code 自動產生 | 2026-02-12
```

---

## Model 定義模板

```markdown
## 📋 資料結構定義

> Task #{TaskID}: {Task 標題}

### `UserDto`

**命名空間：** `Project.Core.Models.Dtos`
**檔案：** `src/Core/Models/Dtos/UserDto.cs`
**對應 Entity：** `User`

| 屬性 | 型別 | 必填 | 驗證規則 | 說明 |
|---|---|---|---|---|
| Id | int | ✅ | — | 主鍵 |
| Name | string | ✅ | MaxLength(100) | 使用者名稱 |
| Email | string? | | Email 格式 | 電子郵件 |
| Role | string | ✅ | — | 角色，預設 "user" |
| CreatedAt | DateTime | ✅ | — | 建立時間 |

**JSON 範例：**
```json
{
  "id": 1,
  "name": "Mark",
  "email": "mark@example.com",
  "role": "admin",
  "createdAt": "2026-02-12T10:30:00Z"
}
```

### Mapping 關係

| 來源 | 目標 | Profile | 備註 |
|------|------|---------|------|
| User → UserDto | UserProfile.cs | 全欄位映射 | — |
| CreateUserRequest → User | UserProfile.cs | 排除 Id, CreatedAt | — |

### 相關 DTO

- `CreateUserRequest` — 建立使用者的 Request Body
- `UpdateUserRequest` — 更新使用者的 Request Body
- `UserListDto` — 清單用的精簡版（不含 Email）

> 由 Claude Code 自動產生 | 2026-02-12
```

---

## 通用回覆模板

```markdown
## 💬 回覆

> Task #{TaskID}: {Task 標題}

{以自然語言回答 Task 中的問題}

### 相關程式碼

**`src/Services/UserService.cs`** (Line 25-40)
```csharp
{相關程式碼片段}
```

{補充說明}

### 參考檔案

- `{檔案路徑}` — {簡述這個檔案的用途}
- `{檔案路徑}` — {簡述}

> 由 Claude Code 自動產生 | 2026-02-12
```

---

## 狀態 Emoji 速查

| Emoji | 用途 |
|-------|------|
| ✅ | API 規格（已完成、可使用） |
| 🔍 | Bug 排查結果 |
| 📋 | Model / 資料結構定義 |
| 💬 | 通用回覆 |
| 🚧 | 開發中、尚未完成 |
| ⚠️ | 有注意事項 |
| 🔒 | 需要特定權限 |
