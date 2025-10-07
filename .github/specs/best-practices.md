# Best Practices for .NET 10 Development

## General Coding Practices

### 1. Use Modern C# Features

#### Nullable Reference Types
```csharp
// Enable in project file
<Nullable>enable</Nullable>

// Use nullable annotations
public string? OptionalValue { get; set; }
public string RequiredValue { get; set; } = string.Empty;
```

#### File-Scoped Namespaces
```csharp
namespace MyApp.Domain.Models;

public class User
{
    // Class implementation
}
```

#### Record Types for DTOs
```csharp
public record UserDto(int Id, string Name, string Email);
public record CreateUserRequest(string Name, string Email);
```

#### Pattern Matching
```csharp
public string GetUserType(User user) => user switch
{
    { IsAdmin: true } => "Administrator",
    { IsPremium: true } => "Premium User",
    _ => "Regular User"
};
```

### 2. Asynchronous Programming

#### Always Use Async/Await Properly
```csharp
// ✅ Good
public async Task<User> GetUserAsync(int id)
{
    return await _context.Users.FindAsync(id);
}

// ❌ Bad - async void (except event handlers)
public async void ProcessUser(int id) { }

// ❌ Bad - blocking on async
public User GetUser(int id)
{
    return _context.Users.FindAsync(id).Result; // Deadlock risk
}
```

#### Use ConfigureAwait When Appropriate
```csharp
// In library code
var data = await _httpClient.GetAsync(url).ConfigureAwait(false);
```

### 3. Dependency Injection

#### Constructor Injection (Preferred)
```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    private readonly ILogger<UserService> _logger;

    public UserService(IUserRepository userRepository, ILogger<UserService> logger)
    {
        _userRepository = userRepository;
        _logger = logger;
    }
}
```

#### Service Registration
```csharp
// Program.cs
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddSingleton<ICacheService, CacheService>();
builder.Services.AddTransient<IEmailService, EmailService>();
```

### 4. Error Handling

#### Use Specific Exceptions
```csharp
// ✅ Good
public class UserNotFoundException : Exception
{
    public UserNotFoundException(int userId) 
        : base($"User with ID {userId} was not found") { }
}

// Use custom exceptions
throw new UserNotFoundException(userId);
```

#### Global Exception Handling
```csharp
// Program.cs
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/json";
        
        var error = context.Features.Get<IExceptionHandlerFeature>();
        if (error != null)
        {
            var logger = context.RequestServices.GetRequiredService<ILogger<Program>>();
            logger.LogError(error.Error, "Unhandled exception");
            
            await context.Response.WriteAsJsonAsync(new { error = "An error occurred" });
        }
    });
});
```

### 5. Logging

#### Structured Logging
```csharp
// ✅ Good - structured
_logger.LogInformation("User {UserId} performed action {Action}", userId, action);

// ❌ Bad - string interpolation
_logger.LogInformation($"User {userId} performed action {action}");
```

#### Log Levels
- **Trace**: Very detailed debugging
- **Debug**: Development debugging
- **Information**: General flow
- **Warning**: Abnormal but expected
- **Error**: Errors and exceptions
- **Critical**: Fatal errors

### 6. Entity Framework Core

#### Use Async Methods
```csharp
var users = await _context.Users
    .Where(u => u.IsActive)
    .ToListAsync();
```

#### Avoid N+1 Queries
```csharp
// ✅ Good - eager loading
var users = await _context.Users
    .Include(u => u.Orders)
    .ToListAsync();

// ❌ Bad - causes N+1 queries
var users = await _context.Users.ToListAsync();
foreach (var user in users)
{
    var orders = await _context.Orders.Where(o => o.UserId == user.Id).ToListAsync();
}
```

#### Use AsNoTracking for Read-Only Queries
```csharp
var users = await _context.Users
    .AsNoTracking()
    .ToListAsync();
```

#### Database Transactions
```csharp
using var transaction = await _context.Database.BeginTransactionAsync();
try
{
    await _context.Users.AddAsync(user);
    await _context.SaveChangesAsync();
    
    await _context.Orders.AddAsync(order);
    await _context.SaveChangesAsync();
    
    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

### 7. API Design

#### Use Proper HTTP Status Codes
```csharp
[HttpGet("{id}")]
public async Task<ActionResult<UserDto>> GetUser(int id)
{
    var user = await _userService.GetUserAsync(id);
    if (user == null)
        return NotFound(); // 404
    
    return Ok(user); // 200
}

[HttpPost]
public async Task<ActionResult<UserDto>> CreateUser(CreateUserRequest request)
{
    var user = await _userService.CreateUserAsync(request);
    return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user); // 201
}
```

#### API Versioning
```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});

// Controller
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class UsersController : ControllerBase
{
}
```

### 8. Validation

#### Use FluentValidation
```csharp
public class CreateUserRequestValidator : AbstractValidator<CreateUserRequest>
{
    public CreateUserRequestValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty()
            .MaximumLength(100);
        
        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress();
    }
}
```

### 9. Configuration

#### Use Options Pattern
```csharp
public class EmailSettings
{
    public string SmtpServer { get; set; } = string.Empty;
    public int Port { get; set; }
    public string FromAddress { get; set; } = string.Empty;
}

// Program.cs
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));

// Usage
public class EmailService
{
    private readonly EmailSettings _settings;
    
    public EmailService(IOptions<EmailSettings> settings)
    {
        _settings = settings.Value;
    }
}
```

### 10. Performance

#### Use Span<T> and Memory<T>
```csharp
public void ProcessData(ReadOnlySpan<byte> data)
{
    // Efficient memory operations
}
```

#### String Interpolation vs String.Format
```csharp
// ✅ Good - use interpolation for simple cases
var message = $"Hello, {name}!";

// Use StringBuilder for loops
var sb = new StringBuilder();
foreach (var item in items)
{
    sb.AppendLine($"Item: {item}");
}
```

#### Cache Appropriately
```csharp
public class CacheService
{
    private readonly IMemoryCache _cache;
    
    public async Task<T> GetOrCreateAsync<T>(string key, Func<Task<T>> factory)
    {
        if (!_cache.TryGetValue(key, out T value))
        {
            value = await factory();
            _cache.Set(key, value, TimeSpan.FromMinutes(10));
        }
        return value;
    }
}
```

## Testing Best Practices

### Unit Tests
```csharp
[Fact]
public async Task GetUser_WithValidId_ReturnsUser()
{
    // Arrange
    var userId = 1;
    var expectedUser = new User { Id = userId, Name = "Test" };
    _mockRepository.Setup(r => r.GetByIdAsync(userId))
        .ReturnsAsync(expectedUser);
    
    // Act
    var result = await _userService.GetUserAsync(userId);
    
    // Assert
    result.Should().NotBeNull();
    result.Id.Should().Be(userId);
}
```

### Integration Tests
```csharp
public class UserApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    
    public UserApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task GetUsers_ReturnsSuccessStatusCode()
    {
        var response = await _client.GetAsync("/api/v1/users");
        response.EnsureSuccessStatusCode();
    }
}
```

## Security Best Practices

### Input Validation
```csharp
// Always validate and sanitize
public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);
    
    // Additional validation
    if (await _userService.EmailExistsAsync(request.Email))
        return Conflict("Email already exists");
    
    // Process...
}
```

### Authentication & Authorization
```csharp
[Authorize(Roles = "Admin")]
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteUser(int id)
{
    await _userService.DeleteUserAsync(id);
    return NoContent();
}
```

### Use HTTPS
```csharp
// Program.cs
app.UseHsts();
app.UseHttpsRedirection();
```

## Code Organization

### Follow Clean Architecture
```
src/
├── Domain/           # Entities, value objects, domain events
├── Application/      # Use cases, DTOs, interfaces
├── Infrastructure/   # Data access, external services
└── Presentation/     # API controllers, views
```

### Use Minimal APIs for Simple Endpoints
```csharp
app.MapGet("/api/health", () => Results.Ok(new { status = "healthy" }));

app.MapGet("/api/users/{id}", async (int id, IUserService service) =>
{
    var user = await service.GetUserAsync(id);
    return user is not null ? Results.Ok(user) : Results.NotFound();
});
```
