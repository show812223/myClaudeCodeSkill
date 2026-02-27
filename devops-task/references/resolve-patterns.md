# 問題修復與實作 Patterns

resolve-task 專用的程式碼修改參考。涵蓋常見 Bug Pattern、實作範本、重構手法。

---

## 一、Bug 修復 Patterns

### Null Reference

**症狀：** NullReferenceException、500 錯誤

```csharp
// ❌ 問題程式碼
var user = await _context.Users.FindAsync(id);
return _mapper.Map<UserDto>(user);  // user 可能為 null

// ✅ 修復
var user = await _context.Users.FindAsync(id);
if (user is null)
    return NotFound();
return Ok(_mapper.Map<UserDto>(user));
```

**檢查清單：**
- `FindAsync()` / `FirstOrDefaultAsync()` 回傳後有沒有 null check
- Navigation Property 是否可能為 null
- AutoMapper 來源物件是否可能為 null
- 方法參數是否可能為 null

---

### EF Core 查詢問題

**N+1 查詢：**
```csharp
// ❌ N+1
var orders = await _context.Orders.ToListAsync();
foreach (var order in orders)
{
    order.Items = await _context.OrderItems
        .Where(i => i.OrderId == order.Id).ToListAsync();
}

// ✅ 使用 Include
var orders = await _context.Orders
    .Include(o => o.Items)
    .ToListAsync();
```

**Missing Include（Navigation Property 為 null）：**
```csharp
// ❌ 缺少 Include
var order = await _context.Orders.FindAsync(id);
var itemCount = order.Items.Count;  // Items 是 null

// ✅ 加 Include
var order = await _context.Orders
    .Include(o => o.Items)
    .FirstOrDefaultAsync(o => o.Id == id);
```

**過度 Include（效能問題）：**
```csharp
// ❌ 撈太多資料
var users = await _context.Users
    .Include(u => u.Orders)
        .ThenInclude(o => o.Items)
            .ThenInclude(i => i.Product)
    .ToListAsync();

// ✅ 只投影需要的欄位
var users = await _context.Users
    .Select(u => new UserListDto
    {
        Id = u.Id,
        Name = u.Name,
        OrderCount = u.Orders.Count
    })
    .ToListAsync();
```

**Where 條件錯誤：**
```csharp
// ❌ 邏輯錯誤：應該是 AND 不是 OR
.Where(u => u.IsActive || u.Role == "admin")

// ✅ 正確
.Where(u => u.IsActive && u.Role == "admin")
```

---

### AutoMapper 映射問題

**欄位遺漏：**
```csharp
// ❌ 沒有設定映射，目標欄位為 default
CreateMap<User, UserDto>();
// 如果 User.FullName 不存在但 UserDto.FullName 存在 → 永遠是 null

// ✅ 明確映射
CreateMap<User, UserDto>()
    .ForMember(d => d.FullName, opt => opt.MapFrom(s => $"{s.FirstName} {s.LastName}"));
```

**Ignore 造成資料遺失：**
```csharp
// ❌ 過度 Ignore
CreateMap<UpdateRequest, User>()
    .ForMember(d => d.Email, opt => opt.Ignore());  // Email 不會被更新

// ✅ 確認 Ignore 的欄位是刻意的
```

---

### 型別轉換錯誤

```csharp
// ❌ 可能 InvalidCastException
var amount = (decimal)reader["Amount"];  // 如果是 DBNull

// ✅ 安全轉換
var amount = reader["Amount"] as decimal? ?? 0m;

// ❌ Enum 轉換
var status = (OrderStatus)dto.Status;  // 如果值不在 enum 範圍內

// ✅ 安全轉換
if (!Enum.IsDefined(typeof(OrderStatus), dto.Status))
    return BadRequest($"Invalid status: {dto.Status}");
var status = (OrderStatus)dto.Status;
```

---

### 併發問題

```csharp
// ❌ Check-then-act race condition
if (!await _context.Users.AnyAsync(u => u.Email == email))
{
    _context.Users.Add(new User { Email = email });
    await _context.SaveChangesAsync();  // 另一個 request 可能同時插入
}

// ✅ 使用 unique constraint + try-catch
try
{
    _context.Users.Add(new User { Email = email });
    await _context.SaveChangesAsync();
}
catch (DbUpdateException ex) when (ex.InnerException is /* unique constraint */)
{
    return Conflict("Email already exists");
}
```

---

### Validation 不完整

```csharp
// ❌ 只檢查 null，沒檢查空字串
if (request.Name != null) { ... }

// ✅ 完整檢查
if (string.IsNullOrWhiteSpace(request.Name))
    return BadRequest("Name is required");

// ❌ 沒有檢查陣列/集合
var items = request.Items;  // 可能是空陣列

// ✅ 檢查集合
if (request.Items is not { Count: > 0 })
    return BadRequest("At least one item is required");
```

---

## 二、新功能實作範本

### CRUD Controller 範本

偵測專案風格後，參考此結構（具體風格跟隨專案）：

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class ItemsController : ControllerBase
{
    private readonly IItemService _service;

    public ItemsController(IItemService service)
    {
        _service = service;
    }

    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<ItemDto>), 200)]
    public async Task<IActionResult> GetAll([FromQuery] ItemQueryRequest request)
    {
        var result = await _service.GetAllAsync(request);
        return Ok(result);
    }

    [HttpGet("{id}")]
    [ProducesResponseType(typeof(ItemDto), 200)]
    [ProducesResponseType(404)]
    public async Task<IActionResult> GetById(int id)
    {
        var item = await _service.GetByIdAsync(id);
        if (item is null) return NotFound();
        return Ok(item);
    }

    [HttpPost]
    [ProducesResponseType(typeof(ItemDto), 201)]
    [ProducesResponseType(typeof(ValidationProblemDetails), 400)]
    public async Task<IActionResult> Create([FromBody] CreateItemRequest request)
    {
        var item = await _service.CreateAsync(request);
        return CreatedAtAction(nameof(GetById), new { id = item.Id }, item);
    }

    [HttpPut("{id}")]
    [ProducesResponseType(typeof(ItemDto), 200)]
    [ProducesResponseType(404)]
    public async Task<IActionResult> Update(int id, [FromBody] UpdateItemRequest request)
    {
        var item = await _service.UpdateAsync(id, request);
        if (item is null) return NotFound();
        return Ok(item);
    }

    [HttpDelete("{id}")]
    [ProducesResponseType(204)]
    [ProducesResponseType(404)]
    public async Task<IActionResult> Delete(int id)
    {
        var success = await _service.DeleteAsync(id);
        if (!success) return NotFound();
        return NoContent();
    }
}
```

### Service 層範本

```csharp
public interface IItemService
{
    Task<PagedResult<ItemDto>> GetAllAsync(ItemQueryRequest request);
    Task<ItemDto?> GetByIdAsync(int id);
    Task<ItemDto> CreateAsync(CreateItemRequest request);
    Task<ItemDto?> UpdateAsync(int id, UpdateItemRequest request);
    Task<bool> DeleteAsync(int id);
}

public class ItemService : IItemService
{
    private readonly AppDbContext _context;
    private readonly IMapper _mapper;

    public ItemService(AppDbContext context, IMapper mapper)
    {
        _context = context;
        _mapper = mapper;
    }

    // 實作各方法...
}
```

### DI 註冊

```csharp
// Program.cs 或 ServiceCollectionExtensions
builder.Services.AddScoped<IItemService, ItemService>();
```

### AutoMapper Profile

```csharp
public class ItemProfile : Profile
{
    public ItemProfile()
    {
        CreateMap<Item, ItemDto>();
        CreateMap<CreateItemRequest, Item>();
        CreateMap<UpdateItemRequest, Item>()
            .ForAllMembers(opt => opt.Condition((src, dest, srcMember) => srcMember != null));
    }
}
```

---

## 三、重構手法

### 提取方法（Extract Method）

```csharp
// ❌ 過長方法
public async Task<IActionResult> ProcessOrder(OrderRequest request)
{
    // 30 行 validation
    // 20 行 計算折扣
    // 15 行 建立訂單
    // 10 行 發送通知
}

// ✅ 拆分
public async Task<IActionResult> ProcessOrder(OrderRequest request)
{
    var validationResult = ValidateOrder(request);
    if (!validationResult.IsValid) return BadRequest(validationResult.Errors);

    var discount = CalculateDiscount(request);
    var order = await CreateOrder(request, discount);
    await SendOrderNotification(order);

    return Ok(_mapper.Map<OrderDto>(order));
}
```

### 提取 Service（Extract Service）

**辨識時機：** Controller 裡有超過 5 行的商業邏輯

```csharp
// ❌ Controller 做太多事
[HttpPost]
public async Task<IActionResult> Register(RegisterRequest request)
{
    var existingUser = await _context.Users.FirstOrDefaultAsync(u => u.Email == request.Email);
    if (existingUser != null) return Conflict();

    var hashedPassword = BCrypt.HashPassword(request.Password);
    var user = new User { Email = request.Email, Password = hashedPassword };
    _context.Users.Add(user);
    await _context.SaveChangesAsync();

    await _emailService.SendWelcome(user.Email);
    return CreatedAtAction(nameof(GetProfile), new { id = user.Id }, _mapper.Map<UserDto>(user));
}

// ✅ 提取到 Service
[HttpPost]
public async Task<IActionResult> Register(RegisterRequest request)
{
    var result = await _authService.RegisterAsync(request);
    if (result.IsConflict) return Conflict();
    return CreatedAtAction(nameof(GetProfile), new { id = result.User.Id }, result.User);
}
```

### EF Core 查詢優化

```csharp
// ❌ 撈全部欄位
var users = await _context.Users.ToListAsync();
return _mapper.Map<List<UserListDto>>(users);

// ✅ Select projection（只查需要的欄位）
var users = await _context.Users
    .Select(u => new UserListDto
    {
        Id = u.Id,
        Name = u.Name,
        Email = u.Email
    })
    .ToListAsync();

// ❌ 沒有用 AsNoTracking
var user = await _context.Users.FirstOrDefaultAsync(u => u.Id == id);

// ✅ 唯讀查詢加 AsNoTracking
var user = await _context.Users
    .AsNoTracking()
    .FirstOrDefaultAsync(u => u.Id == id);
```

### 消除重複（DRY）

```csharp
// ❌ 多個 Controller 重複的分頁邏輯
// UsersController, OrdersController, ProductsController 都有一樣的分頁程式碼

// ✅ 提取為 Extension Method
public static class QueryableExtensions
{
    public static async Task<PagedResult<T>> ToPagedResultAsync<T>(
        this IQueryable<T> query, int page, int pageSize)
    {
        var totalCount = await query.CountAsync();
        var items = await query
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();

        return new PagedResult<T>(items, totalCount, page, pageSize);
    }
}
```

---

## 四、測試 Patterns

### Unit Test — Service 層

```csharp
public class UserServiceTests
{
    private readonly Mock<AppDbContext> _mockContext;
    private readonly Mock<IMapper> _mockMapper;
    private readonly UserService _sut;

    public UserServiceTests()
    {
        _mockContext = new Mock<AppDbContext>(new DbContextOptions<AppDbContext>());
        _mockMapper = new Mock<IMapper>();
        _sut = new UserService(_mockContext.Object, _mockMapper.Object);
    }

    [Fact]
    public async Task GetByIdAsync_UserExists_ReturnsUserDto()
    {
        // Arrange
        var user = new User { Id = 1, Name = "Test" };
        var expectedDto = new UserDto { Id = 1, Name = "Test" };
        // setup mock...

        // Act
        var result = await _sut.GetByIdAsync(1);

        // Assert
        Assert.NotNull(result);
        Assert.Equal(expectedDto.Name, result.Name);
    }

    [Fact]
    public async Task GetByIdAsync_UserNotFound_ReturnsNull()
    {
        // Arrange — setup mock to return null

        // Act
        var result = await _sut.GetByIdAsync(999);

        // Assert
        Assert.Null(result);
    }
}
```

### Integration Test — Controller 層

```csharp
public class UsersControllerTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public UsersControllerTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetById_ExistingUser_Returns200()
    {
        var response = await _client.GetAsync("/api/users/1");
        response.EnsureSuccessStatusCode();

        var user = await response.Content.ReadFromJsonAsync<UserDto>();
        Assert.NotNull(user);
    }

    [Fact]
    public async Task GetById_NonExistent_Returns404()
    {
        var response = await _client.GetAsync("/api/users/99999");
        Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
    }

    [Fact]
    public async Task Create_InvalidRequest_Returns400()
    {
        var request = new CreateUserRequest { Name = "" };  // invalid
        var response = await _client.PostAsJsonAsync("/api/users", request);
        Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
    }
}
```

### Bug 回歸測試

當修復 Bug 時，先寫一個能重現 Bug 的測試，再修復：

```csharp
[Fact]
public async Task GetUserById_NullUser_ShouldReturn404_NotThrow500()
{
    // 這個測試重現 Task #12345 的 Bug：
    // user 不存在時應該回傳 404，不應拋出 NullReferenceException

    var response = await _client.GetAsync("/api/users/99999");

    // 修復前這裡會是 500，修復後應該是 404
    Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
}
```
