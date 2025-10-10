# Application Architecture Specification

## Overview

This document defines the architectural patterns and structure for .NET web applications following clean architecture principles with domain-driven design.

## Architecture Layers

### 1. Presentation Layer
**Responsibility**: Handle HTTP requests, user interface, API endpoints

**Components**:
- Controllers/Minimal APIs
- View Models
- Request/Response DTOs
- Middleware
- Filters

**Dependencies**: → Application Layer

```csharp
// Example Controller
namespace MyApp.Api.Controllers;

[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IMediator _mediator;

    public UsersController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<UserResponse>> GetUser(int id)
    {
        var query = new GetUserQuery(id);
        var result = await _mediator.Send(query);
        return result is not null ? Ok(result) : NotFound();
    }
}
```

### 2. Application Layer
**Responsibility**: Business logic orchestration, use cases, application services

**Components**:
- Command/Query handlers (CQRS)
- Application services
- DTOs and mappings
- Validators
- Application interfaces

**Dependencies**: → Domain Layer

**Patterns**:
- CQRS (MediatR)
- Repository pattern interfaces
- Unit of Work pattern

```csharp
// Example Command Handler
namespace MyApp.Application.Users.Commands;

public record CreateUserCommand(string FirstName, string LastName, string Email) 
    : IRequest<UserResponse>;

public class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, UserResponse>
{
    private readonly IUserRepository _userRepository;
    private readonly IMapper _mapper;

    public CreateUserCommandHandler(IUserRepository userRepository, IMapper mapper)
    {
        _userRepository = userRepository;
        _mapper = mapper;
    }

    public async Task<UserResponse> Handle(CreateUserCommand request, CancellationToken cancellationToken)
    {
        var user = new User(request.FirstName, request.LastName, request.Email);
        await _userRepository.AddAsync(user);
        return _mapper.Map<UserResponse>(user);
    }
}
```

### 3. Domain Layer
**Responsibility**: Core business logic, entities, domain events

**Components**:
- Entities
- Value Objects
- Domain Events
- Domain Services
- Specifications
- Enumerations

**Dependencies**: None (pure business logic)

```csharp
// Example Entity
namespace MyApp.Domain.Entities;

public class User : BaseEntity
{
    public string FirstName { get; private set; }
    public string LastName { get; private set; }
    public Email Email { get; private set; }
    public bool IsActive { get; private set; }
    
    private readonly List<Order> _orders = new();
    public IReadOnlyCollection<Order> Orders => _orders.AsReadOnly();

    public User(string firstName, string lastName, Email email)
    {
        FirstName = firstName;
        LastName = lastName;
        Email = email;
        IsActive = true;
    }

    public void Deactivate()
    {
        IsActive = false;
        AddDomainEvent(new UserDeactivatedEvent(Id));
    }

    public void PlaceOrder(Order order)
    {
        if (!IsActive)
            throw new InvalidOperationException("Cannot place order for inactive user");
        
        _orders.Add(order);
        AddDomainEvent(new OrderPlacedEvent(Id, order.Id));
    }
}

// Example Value Object
public class Email : ValueObject
{
    public string Value { get; }

    public Email(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            throw new ArgumentException("Email cannot be empty");
        
        if (!IsValidEmail(value))
            throw new ArgumentException("Invalid email format");
        
        Value = value.ToLowerInvariant();
    }

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return Value;
    }

    private static bool IsValidEmail(string email)
    {
        // Email validation logic
        return Regex.IsMatch(email, @"^[^@\s]+@[^@\s]+\.[^@\s]+$");
    }
}
```

### 4. Infrastructure Layer
**Responsibility**: External concerns, data access, third-party integrations

**Components**:
- Database context (EF Core)
- Repository implementations
- External service clients
- File system access
- Email services
- Caching implementations

**Dependencies**: → Domain Layer, Application Layer interfaces

```csharp
// Example Repository Implementation
namespace MyApp.Infrastructure.Data.Repositories;

public class UserRepository : IUserRepository
{
    private readonly ApplicationDbContext _context;

    public UserRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<User?> GetByIdAsync(int id)
    {
        return await _context.Users
            .Include(u => u.Orders)
            .FirstOrDefaultAsync(u => u.Id == id);
    }

    public async Task AddAsync(User user)
    {
        await _context.Users.AddAsync(user);
        await _context.SaveChangesAsync();
    }
}
```

## Project Structure

```
src/
├── MyApp.Domain/
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Events/
│   ├── Interfaces/
│   ├── Specifications/
│   └── Exceptions/
│
├── MyApp.Application/
│   ├── Common/
│   │   ├── Interfaces/
│   │   ├── Mappings/
│   │   └── Models/
│   ├── Users/
│   │   ├── Commands/
│   │   ├── Queries/
│   │   └── Validators/
│   └── Orders/
│       ├── Commands/
│       ├── Queries/
│       └── Validators/
│
├── MyApp.Infrastructure/
│   ├── Data/
│   │   ├── Configurations/
│   │   ├── Repositories/
│   │   └── ApplicationDbContext.cs
│   ├── Services/
│   │   ├── EmailService.cs
│   │   └── CacheService.cs
│   └── Identity/
│
├── MyApp.Api/
│   ├── Controllers/
│   ├── Middleware/
│   ├── Filters/
│   └── Program.cs
│
└── MyApp.Web/ (if using MVC/Razor)
    ├── Pages/
    ├── ViewModels/
    └── Program.cs

tests/
├── MyApp.UnitTests/
│   ├── Domain/
│   ├── Application/
│   └── Infrastructure/
│
├── MyApp.IntegrationTests/
│   └── Api/
│
└── MyApp.E2ETests/
    └── Scenarios/
```

## Design Patterns

### 1. Repository Pattern
Abstracts data access logic

```csharp
public interface IRepository<T> where T : BaseEntity
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
}
```

### 2. Unit of Work Pattern
Manages transactions across repositories

```csharp
public interface IUnitOfWork : IDisposable
{
    IUserRepository Users { get; }
    IOrderRepository Orders { get; }
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitAsync();
    Task RollbackAsync();
}
```

### 3. CQRS Pattern
Separates read and write operations

```csharp
// Command (Write)
public record CreateUserCommand(string Name, string Email) : IRequest<int>;

// Query (Read)
public record GetUserQuery(int UserId) : IRequest<UserDto>;
```

### 4. Mediator Pattern
Decouples request/response handling

```csharp
// In controller
var command = new CreateUserCommand(name, email);
var userId = await _mediator.Send(command);
```

### 5. Specification Pattern
Encapsulates query logic

```csharp
public class ActiveUsersSpecification : Specification<User>
{
    public override Expression<Func<User, bool>> ToExpression()
    {
        return user => user.IsActive;
    }
}

// Usage
var activeUsers = await _userRepository.FindAsync(new ActiveUsersSpecification());
```

## API Design

### Versioning Strategy
Use URL versioning

```csharp
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]")]
public class UsersController : ControllerBase
```

### Response Format
Consistent API responses

```csharp
public class ApiResponse<T>
{
    public bool Success { get; set; }
    public T? Data { get; set; }
    public string? Message { get; set; }
    public List<string> Errors { get; set; } = new();
}

// Success response
return Ok(new ApiResponse<UserDto>
{
    Success = true,
    Data = userDto
});

// Error response
return BadRequest(new ApiResponse<object>
{
    Success = false,
    Message = "Validation failed",
    Errors = validationErrors
});
```

### Pagination
Standard pagination for list endpoints

```csharp
public class PagedResult<T>
{
    public List<T> Items { get; set; } = new();
    public int PageNumber { get; set; }
    public int PageSize { get; set; }
    public int TotalPages { get; set; }
    public int TotalCount { get; set; }
    public bool HasPrevious => PageNumber > 1;
    public bool HasNext => PageNumber < TotalPages;
}

[HttpGet]
public async Task<ActionResult<PagedResult<UserDto>>> GetUsers(
    [FromQuery] int pageNumber = 1,
    [FromQuery] int pageSize = 10)
{
    var result = await _mediator.Send(new GetUsersQuery(pageNumber, pageSize));
    return Ok(result);
}
```

## Data Access

### Entity Framework Core Configuration
Use Fluent API for entity configuration

```csharp
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("Users");
        
        builder.HasKey(u => u.Id);
        
        builder.Property(u => u.FirstName)
            .IsRequired()
            .HasMaxLength(100);
        
        builder.Property(u => u.Email)
            .IsRequired()
            .HasMaxLength(255);
        
        builder.HasIndex(u => u.Email)
            .IsUnique();
        
        builder.HasMany(u => u.Orders)
            .WithOne(o => o.User)
            .HasForeignKey(o => o.UserId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}
```

### Database Migrations
Manage schema changes

```bash
# Add migration
dotnet ef migrations add InitialCreate --project src/MyApp.Infrastructure

# Update database
dotnet ef database update --project src/MyApp.Infrastructure
```

## Authentication & Authorization

### JWT Authentication
```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = configuration["Jwt:Issuer"],
            ValidAudience = configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(configuration["Jwt:Key"]))
        };
    });
```

### Policy-Based Authorization
```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAdminRole", policy => 
        policy.RequireRole("Admin"));
    
    options.AddPolicy("RequireEmailVerified", policy =>
        policy.RequireClaim("EmailVerified", "true"));
});

[Authorize(Policy = "RequireAdminRole")]
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteUser(int id)
{
    // ...
}
```

## Error Handling

### Global Exception Handler
```csharp
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;

    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "An unhandled exception occurred");

        var (statusCode, message) = exception switch
        {
            ValidationException => (400, "Validation failed"),
            NotFoundException => (404, "Resource not found"),
            UnauthorizedAccessException => (401, "Unauthorized"),
            _ => (500, "An error occurred")
        };

        httpContext.Response.StatusCode = statusCode;
        await httpContext.Response.WriteAsJsonAsync(new
        {
            error = message,
            details = exception.Message
        }, cancellationToken);

        return true;
    }
}
```

## Caching Strategy

### Multi-Level Caching
1. **Memory Cache**: Fast, in-process
2. **Distributed Cache**: Redis for multi-instance
3. **HTTP Caching**: Client-side with ETags

```csharp
public class CacheService : ICacheService
{
    private readonly IMemoryCache _memoryCache;
    private readonly IDistributedCache _distributedCache;

    public async Task<T?> GetOrCreateAsync<T>(
        string key,
        Func<Task<T>> factory,
        TimeSpan? expiration = null)
    {
        // Try memory cache first
        if (_memoryCache.TryGetValue(key, out T value))
            return value;

        // Try distributed cache
        var cached = await _distributedCache.GetStringAsync(key);
        if (cached != null)
        {
            value = JsonSerializer.Deserialize<T>(cached);
            _memoryCache.Set(key, value, expiration ?? TimeSpan.FromMinutes(5));
            return value;
        }

        // Execute factory and cache
        value = await factory();
        await SetAsync(key, value, expiration);
        
        return value;
    }
}
```

## Performance Optimization

### Database Query Optimization
```csharp
// Use projection to select only needed fields
var users = await _context.Users
    .Select(u => new UserDto
    {
        Id = u.Id,
        Name = u.FirstName + " " + u.LastName,
        Email = u.Email
    })
    .ToListAsync();

// Use AsNoTracking for read-only queries
var users = await _context.Users
    .AsNoTracking()
    .ToListAsync();

// Batch operations
await _context.BulkInsertAsync(users);
```

### Response Compression
```csharp
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});
```

## Health Checks

```csharp
builder.Services.AddHealthChecks()
    .AddDbContextCheck<ApplicationDbContext>()
    .AddRedis(configuration["Redis:ConnectionString"])
    .AddUrlGroup(new Uri(configuration["ExternalApi:Url"]), "External API");

app.MapHealthChecks("/health");
```

## Dependency Injection Container Setup

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Database
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

// MediatR
builder.Services.AddMediatR(cfg => 
    cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));

// AutoMapper
builder.Services.AddAutoMapper(typeof(Program).Assembly);

// Validators
builder.Services.AddValidatorsFromAssembly(typeof(Program).Assembly);

// Repositories
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IOrderRepository, OrderRepository>();

// Services
builder.Services.AddScoped<IEmailService, EmailService>();
builder.Services.AddSingleton<ICacheService, CacheService>();

// Caching
builder.Services.AddMemoryCache();
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration["Redis:ConnectionString"];
});

var app = builder.Build();

// Configure middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```
