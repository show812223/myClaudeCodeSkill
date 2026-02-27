# 程式碼解析 Patterns

## 一、ASP.NET Core Controller

### Route 解析

**Controller 層級：**
```csharp
[Route("api/[controller]")]        // → /api/{ControllerName 去 Controller 後綴, 轉小寫}
[Route("api/v{version}/[controller]")]
[Route("api/users")]               // 固定路由
[ApiController]                     // 標記為 API Controller
```

**Action 層級：**
```csharp
[HttpGet]                          // GET {controller-route}
[HttpGet("{id}")]                  // GET {controller-route}/{id}
[HttpGet("search")]                // GET {controller-route}/search
[HttpPost]                         // POST {controller-route}
[HttpPut("{id}")]                  // PUT {controller-route}/{id}
[HttpDelete("{id}")]               // DELETE {controller-route}/{id}
```

**組合規則：** `UsersController` + `[Route("api/[controller]")]` → `/api/users`

### 參數來源

```csharp
[FromBody] CreateUserRequest req   // Request Body
[FromQuery] string keyword         // Query Parameter
[FromQuery] int page = 1           // Query（有預設值 = 非必填）
[FromRoute] int id                 // Route Parameter
[FromHeader] string authorization  // Header
[FromForm] IFormFile file          // Form Data / 檔案上傳
CancellationToken ct               // 忽略，不列入文件
```

**無 Attribute 推斷：** 簡單型別 → Query/Route；複雜型別 → Body

### 回傳型別

```csharp
Task<ActionResult<UserDto>>             // → UserDto
Task<IActionResult>                     // → 看方法內部 Ok()/Created()
ActionResult<List<UserDto>>             // → UserDto[]
Task<ActionResult<PagedResult<UserDto>>> // → PagedResult<UserDto>
```

```csharp
[ProducesResponseType(typeof(UserDto), 200)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ProducesResponseType<UserDto>(StatusCodes.Status200OK)]  // .NET 7+
```

### Authorization

```csharp
[Authorize]                                    // 需要認證
[Authorize(Roles = "Admin")]                   // 需要 Admin
[Authorize(Policy = "RequireManager")]         // 需要特定 Policy
[AllowAnonymous]                               // 不需認證（覆蓋 Controller 層級）
```

**判斷順序：** Controller [Authorize] → Action [AllowAnonymous] 覆蓋 → Action [Authorize] 更具體

---

## 二、Minimal API

```csharp
app.MapGet("/api/users", handler)
app.MapPost("/api/users", handler)

// 群組路由
var group = app.MapGroup("/api/users");
group.MapGet("/", handler)           // → GET /api/users
group.MapGet("/{id}", handler)       // → GET /api/users/{id}

// Auth
app.MapGet("/api/users", handler).RequireAuthorization();
app.MapGet("/api/public", handler).AllowAnonymous();
```

---

## 三、Model / DTO / Entity

### 搜尋路徑

```
**/Models/**/*.cs
**/Entities/**/*.cs
**/Dtos/**/*.cs
**/DTOs/**/*.cs
**/ViewModels/**/*.cs
**/Contracts/**/*.cs
**/Requests/**/*.cs
**/Responses/**/*.cs
```

### 屬性解析

```csharp
public int Id { get; set; }                    // int, 必填
public string Name { get; set; } = null!;      // string, 必填
public string? Email { get; set; }             // string, 選填 (nullable)
public required string Role { get; set; }      // string, 必填 (C# 11)
public List<string> Tags { get; set; } = new(); // string[], 選填
```

### Data Annotation 驗證

```csharp
[Required]                    // 必填
[MaxLength(100)]              // 最大長度 100
[MinLength(1)]                // 最小長度 1
[StringLength(100, MinimumLength = 1)]
[Range(1, 100)]               // 數值範圍 1-100
[EmailAddress]                // Email 格式
[Phone]                       // 電話格式
[Url]                         // URL 格式
[RegularExpression(@"...")]   // 正則驗證
```

### 泛型展開

| C# 泛型 | 展開方式 |
|---------|---------|
| `ApiResponse<T>` | 展開外層屬性，T 替換為實際型別 |
| `PagedResult<T>` | `{ data: T[], totalCount, page, pageSize }` |
| `List<T>` / `IEnumerable<T>` | `T[]` |
| `Dictionary<K,V>` | `{ [key: K]: V }` |

### 基本型別對照

| C# | 顯示 |
|----|------|
| int, long | number |
| string | string |
| bool | boolean |
| DateTime, DateTimeOffset | string (ISO 8601) |
| Guid | string (UUID) |
| decimal, double | number |
| IFormFile | file |

---

## 四、Bug 排查相關

### Service 層搜尋

```
**/Services/**/*.cs
**/Services/**/*Service.cs
**/Application/**/*Handler.cs     // CQRS pattern
**/Application/**/*Command.cs
**/Application/**/*Query.cs
```

### Repository / Data Access 層

```
**/Repositories/**/*.cs
**/Data/**/*Repository.cs
**/Infrastructure/**/*Repository.cs
```

### 常見問題 Pattern

**Null Reference：**
```csharp
// 危險：缺少 null check
var entity = await _context.Items.FindAsync(id);
return _mapper.Map<ItemDto>(entity);  // entity 可能為 null
```

**N+1 查詢：**
```csharp
// 危險：迴圈內查詢
foreach (var item in items)
{
    item.Category = await _context.Categories.FindAsync(item.CategoryId);
}
```

**Missing Include：**
```csharp
// 危險：沒有 Include 導航屬性
var order = await _context.Orders.FindAsync(id);
// order.Items 會是 null（沒有 .Include(o => o.Items)）
```

**錯誤的 AutoMapper 映射：**
```csharp
// 檢查 Profile 是否有 ForMember 忽略或自訂映射
CreateMap<Source, Dest>()
    .ForMember(d => d.Name, opt => opt.Ignore());  // 可能導致欄位遺漏
```

---

## 五、Git 變更查詢

```bash
# 特定檔案的近期修改
git log --oneline -10 -- "src/Controllers/UsersController.cs"

# 特定目錄的近期修改
git log --oneline -10 -- "src/Services/"

# 查看特定 commit 的變更
git show --stat abc1234

# 查看兩個日期間的變更
git log --oneline --after="2026-02-01" --before="2026-02-12" -- "src/"
```

---

## 六、排除規則

所有掃描策略都排除以下路徑：

```
**/bin/**
**/obj/**
**/Migrations/**
**/Tests/**
**/test/**
**/*.Tests/**
**/node_modules/**
**/*.g.cs
**/*.Designer.cs
**/wwwroot/**
```
