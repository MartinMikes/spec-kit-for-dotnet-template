# Performance Specifications

## Performance Goals

### Response Time Targets
- API endpoints: < 200ms (p95)
- Database queries: < 100ms (p95)
- Page load time: < 2s (p95)
- Time to First Byte (TTFB): < 500ms

### Throughput Targets
- API: 1000 requests/second per instance
- Database: 500 queries/second
- Concurrent users: 10,000+

### Resource Utilization
- CPU: < 70% average
- Memory: < 80% of allocated
- Database connections: < 80% of pool size

## Database Performance

### Query Optimization

#### Use Indexes Effectively
```csharp
// Entity configuration
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        // Single column index
        builder.HasIndex(u => u.Email)
            .IsUnique();

        // Composite index for common queries
        builder.HasIndex(u => new { u.LastName, u.FirstName });

        // Filtered index
        builder.HasIndex(u => u.CreatedAt)
            .HasFilter("[IsActive] = 1");
    }
}
```

#### Avoid N+1 Query Problems
```csharp
// ❌ Bad - N+1 queries
var users = await _context.Users.ToListAsync();
foreach (var user in users)
{
    var orders = await _context.Orders
        .Where(o => o.UserId == user.Id)
        .ToListAsync();
}

// ✅ Good - Single query with Include
var users = await _context.Users
    .Include(u => u.Orders)
    .ToListAsync();

// ✅ Good - Use projection when you don't need full entity
var users = await _context.Users
    .Select(u => new UserWithOrdersDto
    {
        Id = u.Id,
        Name = u.Name,
        OrderCount = u.Orders.Count
    })
    .ToListAsync();
```

#### Use AsNoTracking for Read-Only Queries
```csharp
// ✅ Good - No tracking overhead
var users = await _context.Users
    .AsNoTracking()
    .Where(u => u.IsActive)
    .ToListAsync();

// ✅ Good - For updates, use tracking
var user = await _context.Users
    .FirstOrDefaultAsync(u => u.Id == userId);
user.Name = "Updated Name";
await _context.SaveChangesAsync();
```

#### Implement Pagination
```csharp
public async Task<PagedResult<UserDto>> GetUsersAsync(int pageNumber, int pageSize)
{
    var query = _context.Users.AsNoTracking();

    var totalCount = await query.CountAsync();

    var users = await query
        .OrderBy(u => u.Id)
        .Skip((pageNumber - 1) * pageSize)
        .Take(pageSize)
        .Select(u => new UserDto
        {
            Id = u.Id,
            Name = u.Name
        })
        .ToListAsync();

    return new PagedResult<UserDto>
    {
        Items = users,
        TotalCount = totalCount,
        PageNumber = pageNumber,
        PageSize = pageSize
    };
}
```

#### Use Compiled Queries for Frequent Queries
```csharp
private static readonly Func<ApplicationDbContext, int, Task<User?>> _getUserById =
    EF.CompileAsyncQuery((ApplicationDbContext context, int id) =>
        context.Users.FirstOrDefault(u => u.Id == id));

public async Task<User?> GetUserByIdAsync(int id)
{
    return await _getUserById(_context, id);
}
```

#### Batch Operations
```csharp
// ✅ Good - Bulk insert
await _context.BulkInsertAsync(users);

// ✅ Good - Bulk update
await _context.BulkUpdateAsync(users);

// ❌ Bad - Individual inserts
foreach (var user in users)
{
    await _context.Users.AddAsync(user);
    await _context.SaveChangesAsync(); // Don't save in loop!
}

// ✅ Good - Batch save
await _context.Users.AddRangeAsync(users);
await _context.SaveChangesAsync();
```

### Connection Pooling
```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseNpgsql(connectionString, npgsqlOptions =>
    {
        npgsqlOptions.EnableRetryOnFailure(
            maxRetryCount: 3,
            maxRetryDelay: TimeSpan.FromSeconds(5),
            errorCodesToAdd: null);
        
        npgsqlOptions.CommandTimeout(30);
    });
    
    // Connection pooling settings
    options.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
});

// Connection string with pooling
"Server=localhost;Database=mydb;User Id=user;Password=pass;Pooling=true;MinPoolSize=5;MaxPoolSize=100;"
```

## Caching Strategies

### Memory Cache
```csharp
public class UserService
{
    private readonly IMemoryCache _cache;
    private readonly IUserRepository _repository;

    public async Task<User?> GetUserAsync(int id)
    {
        var cacheKey = $"user_{id}";

        if (!_cache.TryGetValue(cacheKey, out User user))
        {
            user = await _repository.GetByIdAsync(id);

            if (user != null)
            {
                var cacheOptions = new MemoryCacheEntryOptions()
                    .SetSlidingExpiration(TimeSpan.FromMinutes(5))
                    .SetAbsoluteExpiration(TimeSpan.FromMinutes(30))
                    .SetPriority(CacheItemPriority.Normal)
                    .RegisterPostEvictionCallback((key, value, reason, state) =>
                    {
                        // Log cache eviction
                    });

                _cache.Set(cacheKey, user, cacheOptions);
            }
        }

        return user;
    }
}
```

### Distributed Cache (Redis)
```csharp
public class CacheService
{
    private readonly IDistributedCache _cache;
    private readonly ILogger<CacheService> _logger;

    public async Task<T?> GetAsync<T>(string key)
    {
        var data = await _cache.GetStringAsync(key);
        if (data == null)
            return default;

        return JsonSerializer.Deserialize<T>(data);
    }

    public async Task SetAsync<T>(string key, T value, TimeSpan? expiration = null)
    {
        var options = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = expiration ?? TimeSpan.FromMinutes(10)
        };

        var data = JsonSerializer.Serialize(value);
        await _cache.SetStringAsync(key, data, options);
    }

    public async Task<T> GetOrCreateAsync<T>(string key, Func<Task<T>> factory, TimeSpan? expiration = null)
    {
        var cached = await GetAsync<T>(key);
        if (cached != null)
            return cached;

        var value = await factory();
        await SetAsync(key, value, expiration);
        return value;
    }

    public async Task RemoveAsync(string key)
    {
        await _cache.RemoveAsync(key);
    }
}

// Configuration
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration["Redis:ConnectionString"];
    options.InstanceName = "MyApp_";
});
```

### Response Caching
```csharp
// Add response caching
builder.Services.AddResponseCaching(options =>
{
    options.MaximumBodySize = 1024 * 1024; // 1 MB
    options.UseCaseSensitivePaths = true;
});

app.UseResponseCaching();

// Use in controller
[HttpGet]
[ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any)]
public async Task<IActionResult> GetProducts()
{
    var products = await _productService.GetAllAsync();
    return Ok(products);
}

// Or use cache profiles
builder.Services.AddControllers(options =>
{
    options.CacheProfiles.Add("Default30",
        new CacheProfile
        {
            Duration = 30,
            Location = ResponseCacheLocation.Any
        });
});

[ResponseCache(CacheProfileName = "Default30")]
public async Task<IActionResult> GetProducts()
{
    // ...
}
```

### Output Caching (.NET 7+)
```csharp
builder.Services.AddOutputCache(options =>
{
    options.AddBasePolicy(builder => builder.Expire(TimeSpan.FromSeconds(60)));
    
    options.AddPolicy("Expire30", builder => 
        builder.Expire(TimeSpan.FromSeconds(30)));
    
    options.AddPolicy("NoCache", builder => 
        builder.NoCache());
});

app.UseOutputCache();

// Usage
[OutputCache(Duration = 60)]
public IActionResult GetData()
{
    // ...
}

app.MapGet("/api/products", async (IProductService service) =>
{
    return await service.GetAllAsync();
})
.CacheOutput("Expire30");
```

## API Performance

### Compression
```csharp
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});

builder.Services.Configure<BrotliCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = CompressionLevel.Fastest;
});

app.UseResponseCompression();
```

### Async Operations
```csharp
// ✅ Good - Async all the way
[HttpGet("{id}")]
public async Task<ActionResult<UserDto>> GetUser(int id)
{
    var user = await _userService.GetUserAsync(id);
    if (user == null)
        return NotFound();
    
    return Ok(user);
}

// ❌ Bad - Blocking async code
public ActionResult<UserDto> GetUser(int id)
{
    var user = _userService.GetUserAsync(id).Result; // Deadlock risk!
    return Ok(user);
}

// ❌ Bad - Async void
public async void ProcessUser(int id) // Don't use async void!
{
    await _userService.ProcessAsync(id);
}
```

### Parallel Processing
```csharp
public async Task<CombinedResult> GetCombinedDataAsync(int userId)
{
    // Run multiple async operations in parallel
    var userTask = _userService.GetUserAsync(userId);
    var ordersTask = _orderService.GetUserOrdersAsync(userId);
    var preferencesTask = _preferenceService.GetUserPreferencesAsync(userId);

    await Task.WhenAll(userTask, ordersTask, preferencesTask);

    return new CombinedResult
    {
        User = await userTask,
        Orders = await ordersTask,
        Preferences = await preferencesTask
    };
}

// For collection processing
var tasks = userIds.Select(id => _userService.GetUserAsync(id));
var users = await Task.WhenAll(tasks);
```

### Minimize Payload Size
```csharp
// ✅ Good - Return only needed fields
public record UserDto(int Id, string Name, string Email);

[HttpGet]
public async Task<ActionResult<IEnumerable<UserDto>>> GetUsers()
{
    var users = await _context.Users
        .Select(u => new UserDto(u.Id, u.Name, u.Email))
        .ToListAsync();
    
    return Ok(users);
}

// ✅ Good - Use OData for flexible queries
[EnableQuery]
[HttpGet]
public IQueryable<User> GetUsers()
{
    return _context.Users.AsQueryable();
}

// Client can request: /api/users?$select=id,name&$filter=isActive eq true
```

## Memory Management

### Use Span<T> and Memory<T>
```csharp
// ✅ Good - Zero-allocation substring
public ReadOnlySpan<char> GetFileExtension(string filename)
{
    var lastDot = filename.LastIndexOf('.');
    return lastDot >= 0 ? filename.AsSpan(lastDot) : ReadOnlySpan<char>.Empty;
}

// ✅ Good - Efficient byte operations
public void ProcessData(ReadOnlySpan<byte> data)
{
    // Process without allocations
    foreach (var b in data)
    {
        // ...
    }
}
```

### StringBuilder for String Concatenation
```csharp
// ❌ Bad - Multiple allocations
string result = "";
foreach (var item in items)
{
    result += item.ToString() + ",";
}

// ✅ Good - Single allocation
var sb = new StringBuilder();
foreach (var item in items)
{
    sb.Append(item.ToString()).Append(',');
}
var result = sb.ToString();
```

### Object Pooling
```csharp
public class ObjectPoolService
{
    private readonly ObjectPool<StringBuilder> _stringBuilderPool;

    public ObjectPoolService()
    {
        var policy = new StringBuilderPooledObjectPolicy();
        var provider = new DefaultObjectPoolProvider();
        _stringBuilderPool = provider.Create(policy);
    }

    public string ProcessData(IEnumerable<string> items)
    {
        var sb = _stringBuilderPool.Get();
        try
        {
            foreach (var item in items)
            {
                sb.Append(item).Append(',');
            }
            return sb.ToString();
        }
        finally
        {
            _stringBuilderPool.Return(sb);
        }
    }
}
```

### Reduce Allocations
```csharp
// ✅ Good - Use ValueTask for hot paths
public ValueTask<int> GetCountAsync()
{
    if (_cache.TryGetValue("count", out int count))
        return new ValueTask<int>(count);
    
    return new ValueTask<int>(GetCountFromDbAsync());
}

// ✅ Good - Use struct for small types
public readonly struct Point
{
    public int X { get; init; }
    public int Y { get; init; }
}

// ✅ Good - Use record struct
public readonly record struct Rectangle(int Width, int Height);
```

## Background Processing

### Hosted Services
```csharp
public class DataCleanupService : BackgroundService
{
    private readonly ILogger<DataCleanupService> _logger;
    private readonly IServiceProvider _serviceProvider;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Data cleanup service started");

        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                using (var scope = _serviceProvider.CreateScope())
                {
                    var dbContext = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
                    
                    // Cleanup old records
                    var cutoffDate = DateTime.UtcNow.AddDays(-30);
                    var oldRecords = await dbContext.Logs
                        .Where(l => l.CreatedAt < cutoffDate)
                        .ToListAsync(stoppingToken);
                    
                    dbContext.Logs.RemoveRange(oldRecords);
                    await dbContext.SaveChangesAsync(stoppingToken);
                }

                await Task.Delay(TimeSpan.FromHours(24), stoppingToken);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error in data cleanup service");
            }
        }
    }
}

// Register
builder.Services.AddHostedService<DataCleanupService>();
```

### Channels for Producer-Consumer
```csharp
public class NotificationService
{
    private readonly Channel<Notification> _channel;

    public NotificationService()
    {
        _channel = Channel.CreateBounded<Notification>(new BoundedChannelOptions(100)
        {
            FullMode = BoundedChannelFullMode.Wait
        });

        // Start consumer
        Task.Run(ProcessNotificationsAsync);
    }

    public async Task EnqueueAsync(Notification notification)
    {
        await _channel.Writer.WriteAsync(notification);
    }

    private async Task ProcessNotificationsAsync()
    {
        await foreach (var notification in _channel.Reader.ReadAllAsync())
        {
            // Process notification
            await SendNotificationAsync(notification);
        }
    }
}
```

## HTTP Client Performance

### Use HttpClientFactory
```csharp
// Register
builder.Services.AddHttpClient<IExternalApiClient, ExternalApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.example.com");
    client.Timeout = TimeSpan.FromSeconds(30);
    client.DefaultRequestHeaders.Add("User-Agent", "MyApp/1.0");
});

// Use
public class ExternalApiClient : IExternalApiClient
{
    private readonly HttpClient _httpClient;

    public ExternalApiClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<T> GetAsync<T>(string endpoint)
    {
        var response = await _httpClient.GetAsync(endpoint);
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<T>();
    }
}
```

### Connection Pooling
```csharp
builder.Services.AddHttpClient<IApiClient, ApiClient>()
    .ConfigurePrimaryHttpMessageHandler(() =>
        new SocketsHttpHandler
        {
            PooledConnectionLifetime = TimeSpan.FromMinutes(2),
            PooledConnectionIdleTimeout = TimeSpan.FromMinutes(1),
            MaxConnectionsPerServer = 10
        });
```

## Monitoring & Profiling

### Application Insights
```csharp
builder.Services.AddApplicationInsightsTelemetry();

// Custom metrics
public class MetricsService
{
    private readonly TelemetryClient _telemetry;

    public void TrackPerformance(string operation, TimeSpan duration)
    {
        _telemetry.TrackMetric($"{operation}_Duration", duration.TotalMilliseconds);
    }

    public void TrackDependency(string dependency, TimeSpan duration, bool success)
    {
        _telemetry.TrackDependency(dependency, "", DateTime.UtcNow, duration, success);
    }
}
```

### Performance Logging
```csharp
public class PerformanceMonitoringMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<PerformanceMonitoringMiddleware> _logger;

    public async Task InvokeAsync(HttpContext context)
    {
        var sw = Stopwatch.StartNew();

        try
        {
            await _next(context);
        }
        finally
        {
            sw.Stop();
            
            if (sw.ElapsedMilliseconds > 1000) // Log slow requests
            {
                _logger.LogWarning(
                    "Slow request: {Method} {Path} took {Duration}ms",
                    context.Request.Method,
                    context.Request.Path,
                    sw.ElapsedMilliseconds);
            }
        }
    }
}
```

### Database Query Logging
```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseNpgsql(connectionString)
        .EnableSensitiveDataLogging(builder.Environment.IsDevelopment())
        .LogTo(
            Console.WriteLine,
            new[] { DbLoggerCategory.Database.Command.Name },
            LogLevel.Information);
});
```

## Performance Testing

### Load Testing with K6
```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 100 }, // Ramp up to 100 users
    { duration: '3m', target: 100 }, // Stay at 100 users
    { duration: '1m', target: 0 },   // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
  },
};

export default function () {
  const res = http.get('https://api.myapp.com/api/users');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  sleep(1);
}
```

### BenchmarkDotNet
```csharp
[MemoryDiagnoser]
public class UserServiceBenchmarks
{
    private UserService _userService;

    [GlobalSetup]
    public void Setup()
    {
        // Setup dependencies
    }

    [Benchmark]
    public async Task GetUser()
    {
        await _userService.GetUserAsync(1);
    }

    [Benchmark]
    public async Task GetUsers()
    {
        await _userService.GetUsersAsync(1, 100);
    }
}
```

## Performance Checklist

### Development
- [ ] Use async/await properly
- [ ] Implement caching strategy
- [ ] Optimize database queries
- [ ] Use pagination for lists
- [ ] Enable response compression
- [ ] Minimize payload sizes
- [ ] Use appropriate data structures
- [ ] Avoid N+1 query problems

### Pre-Production
- [ ] Load testing completed
- [ ] Performance benchmarks met
- [ ] Database indexes optimized
- [ ] Caching configured
- [ ] CDN configured for static assets
- [ ] Connection pooling configured
- [ ] Memory profiling done
- [ ] No memory leaks detected

### Production
- [ ] APM tool configured
- [ ] Performance monitoring active
- [ ] Alerts configured for slow requests
- [ ] Auto-scaling configured
- [ ] Database performance monitored
- [ ] Cache hit ratio monitored
- [ ] Resource utilization tracked
