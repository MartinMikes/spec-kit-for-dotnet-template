# Testing Standards

## Testing Strategy

### Test Pyramid
```
        /\
       /  \      E2E Tests (10%)
      /____\     - Full user journeys
     /      \    - Critical business flows
    /________\   Integration Tests (20%)
   /          \  - API tests
  /____________\ - Database tests
 /              \ Unit Tests (70%)
/______________\ - Business logic
                 - Domain models
                 - Services
```

### Test Coverage Requirements
- **Minimum**: 80% overall code coverage
- **Critical Paths**: 100% coverage
- **Business Logic**: 100% coverage
- **Controllers**: 80% coverage
- **Infrastructure**: 60% coverage

## Unit Testing

### Framework Setup
```csharp
// Use xUnit, FluentAssertions, and Moq
<ItemGroup>
  <PackageReference Include="xunit" Version="2.6.0" />
  <PackageReference Include="xunit.runner.visualstudio" Version="2.5.4" />
  <PackageReference Include="FluentAssertions" Version="6.12.0" />
  <PackageReference Include="Moq" Version="4.20.69" />
  <PackageReference Include="AutoFixture" Version="4.18.0" />
  <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
</ItemGroup>
```

### Test Naming Convention
**Pattern**: `MethodName_Scenario_ExpectedBehavior`

```csharp
public class UserServiceTests
{
    [Fact]
    public async Task GetUserAsync_WithValidId_ReturnsUser()
    {
        // ...
    }

    [Fact]
    public async Task GetUserAsync_WithInvalidId_ReturnsNull()
    {
        // ...
    }

    [Fact]
    public async Task CreateUserAsync_WithDuplicateEmail_ThrowsException()
    {
        // ...
    }
}
```

### AAA Pattern (Arrange-Act-Assert)
```csharp
[Fact]
public async Task UpdateUser_WithValidData_UpdatesUser()
{
    // Arrange
    var userId = 1;
    var updateRequest = new UpdateUserRequest("John", "Doe");
    var existingUser = new User { Id = userId, FirstName = "Jane", LastName = "Smith" };
    
    _mockRepository.Setup(r => r.GetByIdAsync(userId))
        .ReturnsAsync(existingUser);
    
    _mockRepository.Setup(r => r.UpdateAsync(It.IsAny<User>()))
        .Returns(Task.CompletedTask);

    // Act
    var result = await _userService.UpdateUserAsync(userId, updateRequest);

    // Assert
    result.Should().NotBeNull();
    result.FirstName.Should().Be("John");
    result.LastName.Should().Be("Doe");
    
    _mockRepository.Verify(
        r => r.UpdateAsync(It.Is<User>(u => 
            u.Id == userId && 
            u.FirstName == "John" && 
            u.LastName == "Doe")), 
        Times.Once);
}
```

### Using AutoFixture
```csharp
public class UserServiceTests
{
    private readonly IFixture _fixture;
    private readonly Mock<IUserRepository> _mockRepository;
    private readonly UserService _userService;

    public UserServiceTests()
    {
        _fixture = new Fixture();
        _mockRepository = new Mock<IUserRepository>();
        _userService = new UserService(_mockRepository.Object);
    }

    [Fact]
    public async Task CreateUser_WithValidData_ReturnsUser()
    {
        // Arrange
        var request = _fixture.Create<CreateUserRequest>();
        var user = _fixture.Build<User>()
            .With(u => u.Email, request.Email)
            .Create();

        _mockRepository.Setup(r => r.AddAsync(It.IsAny<User>()))
            .ReturnsAsync(user);

        // Act
        var result = await _userService.CreateUserAsync(request);

        // Assert
        result.Should().BeEquivalentTo(user);
    }
}
```

### Theory Tests (Data-Driven)
```csharp
[Theory]
[InlineData("john@example.com", true)]
[InlineData("invalid-email", false)]
[InlineData("", false)]
[InlineData(null, false)]
public void IsValidEmail_WithVariousInputs_ReturnsExpectedResult(string email, bool expected)
{
    // Act
    var result = EmailValidator.IsValid(email);

    // Assert
    result.Should().Be(expected);
}

[Theory]
[MemberData(nameof(GetUserTestData))]
public async Task GetUser_WithVariousScenarios_ReturnsExpectedResult(
    int userId, User expectedUser, bool shouldSucceed)
{
    // Arrange
    _mockRepository.Setup(r => r.GetByIdAsync(userId))
        .ReturnsAsync(expectedUser);

    // Act
    var result = await _userService.GetUserAsync(userId);

    // Assert
    if (shouldSucceed)
        result.Should().BeEquivalentTo(expectedUser);
    else
        result.Should().BeNull();
}

public static IEnumerable<object[]> GetUserTestData()
{
    yield return new object[] { 1, new User { Id = 1, Name = "John" }, true };
    yield return new object[] { 2, null, false };
    yield return new object[] { 0, null, false };
}
```

### Testing Exceptions
```csharp
[Fact]
public async Task DeleteUser_WithNonExistentUser_ThrowsNotFoundException()
{
    // Arrange
    var userId = 999;
    _mockRepository.Setup(r => r.GetByIdAsync(userId))
        .ReturnsAsync((User)null);

    // Act
    Func<Task> act = async () => await _userService.DeleteUserAsync(userId);

    // Assert
    await act.Should().ThrowAsync<NotFoundException>()
        .WithMessage($"User with ID {userId} was not found");
}

[Fact]
public async Task CreateUser_WithDuplicateEmail_ThrowsValidationException()
{
    // Arrange
    var request = new CreateUserRequest("test@example.com", "John", "Doe");
    _mockRepository.Setup(r => r.EmailExistsAsync(request.Email))
        .ReturnsAsync(true);

    // Act & Assert
    await Assert.ThrowsAsync<ValidationException>(
        () => _userService.CreateUserAsync(request));
}
```

### Mocking Best Practices
```csharp
// ✅ Good - Verify specific interactions
_mockRepository.Verify(
    r => r.AddAsync(It.Is<User>(u => u.Email == "test@example.com")),
    Times.Once);

// ✅ Good - Setup with specific conditions
_mockRepository.Setup(r => r.GetByIdAsync(It.Is<int>(id => id > 0)))
    .ReturnsAsync((int id) => new User { Id = id });

// ✅ Good - Verify no other calls
_mockRepository.Verify(r => r.DeleteAsync(It.IsAny<User>()), Times.Never);
_mockRepository.VerifyNoOtherCalls();

// ❌ Bad - Too generic
_mockRepository.Setup(r => r.GetByIdAsync(It.IsAny<int>()))
    .ReturnsAsync(new User());
```

## Integration Testing

### WebApplicationFactory Setup
```csharp
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Remove real database
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));
            if (descriptor != null)
                services.Remove(descriptor);

            // Add in-memory database
            services.AddDbContext<ApplicationDbContext>(options =>
            {
                options.UseInMemoryDatabase("TestDb");
            });

            // Build service provider
            var sp = services.BuildServiceProvider();

            // Create scope and seed database
            using var scope = sp.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
            db.Database.EnsureCreated();
            SeedTestData(db);
        });
    }

    private static void SeedTestData(ApplicationDbContext db)
    {
        db.Users.AddRange(
            new User { Id = 1, FirstName = "John", LastName = "Doe", Email = "john@example.com" },
            new User { Id = 2, FirstName = "Jane", LastName = "Smith", Email = "jane@example.com" }
        );
        db.SaveChanges();
    }
}
```

### API Integration Tests
```csharp
public class UsersApiTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly HttpClient _client;
    private readonly CustomWebApplicationFactory _factory;

    public UsersApiTests(CustomWebApplicationFactory factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetUsers_ReturnsSuccessAndCorrectContentType()
    {
        // Act
        var response = await _client.GetAsync("/api/v1/users");

        // Assert
        response.EnsureSuccessStatusCode();
        response.Content.Headers.ContentType?.ToString()
            .Should().Be("application/json; charset=utf-8");
    }

    [Fact]
    public async Task GetUser_WithValidId_ReturnsUser()
    {
        // Act
        var response = await _client.GetAsync("/api/v1/users/1");
        var user = await response.Content.ReadFromJsonAsync<UserDto>();

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        user.Should().NotBeNull();
        user!.Id.Should().Be(1);
        user.Email.Should().Be("john@example.com");
    }

    [Fact]
    public async Task CreateUser_WithValidData_ReturnsCreated()
    {
        // Arrange
        var request = new CreateUserRequest(
            "New User",
            "newuser@example.com",
            "password123");

        // Act
        var response = await _client.PostAsJsonAsync("/api/v1/users", request);
        var user = await response.Content.ReadFromJsonAsync<UserDto>();

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        user.Should().NotBeNull();
        response.Headers.Location.Should().NotBeNull();
    }

    [Fact]
    public async Task CreateUser_WithInvalidData_ReturnsBadRequest()
    {
        // Arrange
        var request = new CreateUserRequest("", "invalid-email", "");

        // Act
        var response = await _client.PostAsJsonAsync("/api/v1/users", request);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
    }
}
```

### Database Integration Tests
```csharp
public class UserRepositoryTests : IDisposable
{
    private readonly ApplicationDbContext _context;
    private readonly UserRepository _repository;

    public UserRepositoryTests()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;

        _context = new ApplicationDbContext(options);
        _repository = new UserRepository(_context);

        SeedDatabase();
    }

    private void SeedDatabase()
    {
        _context.Users.AddRange(
            new User { Id = 1, FirstName = "John", LastName = "Doe" },
            new User { Id = 2, FirstName = "Jane", LastName = "Smith" }
        );
        _context.SaveChanges();
    }

    [Fact]
    public async Task GetByIdAsync_WithExistingId_ReturnsUser()
    {
        // Act
        var user = await _repository.GetByIdAsync(1);

        // Assert
        user.Should().NotBeNull();
        user!.FirstName.Should().Be("John");
    }

    [Fact]
    public async Task AddAsync_WithNewUser_AddsToDatabase()
    {
        // Arrange
        var newUser = new User { FirstName = "Bob", LastName = "Johnson" };

        // Act
        await _repository.AddAsync(newUser);

        // Assert
        var users = await _context.Users.ToListAsync();
        users.Should().HaveCount(3);
        users.Should().Contain(u => u.FirstName == "Bob");
    }

    public void Dispose()
    {
        _context.Database.EnsureDeleted();
        _context.Dispose();
    }
}
```

### Testing with Real Database (TestContainers)
```csharp
public class UserRepositoryIntegrationTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _postgres = new PostgreSqlBuilder()
        .WithDatabase("testdb")
        .WithUsername("testuser")
        .WithPassword("testpass")
        .Build();

    private ApplicationDbContext _context;
    private UserRepository _repository;

    public async Task InitializeAsync()
    {
        await _postgres.StartAsync();

        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseNpgsql(_postgres.GetConnectionString())
            .Options;

        _context = new ApplicationDbContext(options);
        await _context.Database.MigrateAsync();

        _repository = new UserRepository(_context);
    }

    [Fact]
    public async Task GetByEmailAsync_WithExistingEmail_ReturnsUser()
    {
        // Arrange
        var user = new User { Email = "test@example.com", FirstName = "Test" };
        await _repository.AddAsync(user);

        // Act
        var result = await _repository.GetByEmailAsync("test@example.com");

        // Assert
        result.Should().NotBeNull();
        result!.Email.Should().Be("test@example.com");
    }

    public async Task DisposeAsync()
    {
        await _context.DisposeAsync();
        await _postgres.DisposeAsync();
    }
}
```

## End-to-End Testing

### Playwright Setup
```csharp
[Parallelizable(ParallelScope.Self)]
public class E2ETests : PageTest
{
    private const string BaseUrl = "https://localhost:5001";

    [SetUp]
    public async Task Setup()
    {
        await Context.Tracing.StartAsync(new()
        {
            Screenshots = true,
            Snapshots = true,
            Sources = true
        });
    }

    [TearDown]
    public async Task TearDown()
    {
        await Context.Tracing.StopAsync(new()
        {
            Path = Path.Combine(
                TestContext.CurrentContext.WorkDirectory,
                $"trace_{TestContext.CurrentContext.Test.Name}.zip"
            )
        });
    }

    [Test]
    public async Task UserCanLoginSuccessfully()
    {
        // Navigate to login page
        await Page.GotoAsync($"{BaseUrl}/login");

        // Fill login form
        await Page.FillAsync("#email", "user@example.com");
        await Page.FillAsync("#password", "password123");

        // Click login button
        await Page.ClickAsync("button[type='submit']");

        // Assert redirect to dashboard
        await Expect(Page).ToHaveURLAsync($"{BaseUrl}/dashboard");

        // Assert user name is displayed
        await Expect(Page.Locator(".user-name")).ToHaveTextAsync("John Doe");
    }

    [Test]
    public async Task UserCanCreateNewOrder()
    {
        // Login first
        await LoginAsUser();

        // Navigate to orders
        await Page.ClickAsync("text=Orders");
        await Page.ClickAsync("text=Create Order");

        // Fill order form
        await Page.FillAsync("#product", "Product A");
        await Page.FillAsync("#quantity", "5");
        await Page.ClickAsync("button[type='submit']");

        // Assert success message
        await Expect(Page.Locator(".success-message"))
            .ToHaveTextAsync("Order created successfully");

        // Assert order appears in list
        await Expect(Page.Locator(".order-list"))
            .ToContainTextAsync("Product A");
    }
}
```

## Performance Testing

### Load Testing with NBomber
```csharp
[Test]
public void LoadTest_GetUsers_HandlesExpectedLoad()
{
    var httpClient = new HttpClient();

    var scenario = Scenario.Create("get_users", async context =>
    {
        var response = await httpClient.GetAsync("https://localhost:5001/api/users");
        
        return response.IsSuccessStatusCode
            ? Response.Ok()
            : Response.Fail();
    })
    .WithWarmUpDuration(TimeSpan.FromSeconds(5))
    .WithLoadSimulations(
        Simulation.Inject(rate: 100, interval: TimeSpan.FromSeconds(1), during: TimeSpan.FromMinutes(1))
    );

    var stats = NBomberRunner
        .RegisterScenarios(scenario)
        .Run();

    var getUsers = stats.ScenarioStats[0];
    getUsers.Ok.Request.RPS.Should().BeGreaterThan(90);
    getUsers.Ok.Latency.Percent95.Should().BeLessThan(200); // 200ms p95
}
```

### Benchmarking with BenchmarkDotNet
```csharp
[MemoryDiagnoser]
[RankColumn]
public class ServiceBenchmarks
{
    private UserService _userService;
    private IUserRepository _repository;

    [GlobalSetup]
    public void Setup()
    {
        // Setup dependencies
        _repository = new UserRepository(/*...*/);
        _userService = new UserService(_repository);
    }

    [Benchmark]
    public async Task GetUserById()
    {
        await _userService.GetUserAsync(1);
    }

    [Benchmark]
    public async Task GetAllUsers()
    {
        await _userService.GetAllUsersAsync();
    }

    [Benchmark]
    [Arguments(10)]
    [Arguments(100)]
    [Arguments(1000)]
    public async Task GetUsersPaginated(int pageSize)
    {
        await _userService.GetUsersAsync(1, pageSize);
    }
}
```

## Test Organization

### Project Structure
```
tests/
├── MyApp.UnitTests/
│   ├── Domain/
│   │   ├── Entities/
│   │   └── ValueObjects/
│   ├── Application/
│   │   ├── Services/
│   │   └── Handlers/
│   └── Infrastructure/
│       └── Repositories/
│
├── MyApp.IntegrationTests/
│   ├── Api/
│   │   └── Controllers/
│   ├── Data/
│   │   └── Repositories/
│   └── Common/
│       └── WebApplicationFactory.cs
│
└── MyApp.E2ETests/
    ├── Scenarios/
    │   ├── UserJourney/
    │   └── OrderFlow/
    └── Pages/
        ├── LoginPage.cs
        └── DashboardPage.cs
```

### Shared Test Utilities
```csharp
public static class TestDataBuilder
{
    public static User CreateUser(string email = "test@example.com")
    {
        return new User
        {
            Id = 1,
            Email = email,
            FirstName = "Test",
            LastName = "User"
        };
    }

    public static CreateUserRequest CreateUserRequest()
    {
        return new CreateUserRequest(
            "test@example.com",
            "Test",
            "User");
    }
}

public static class MockExtensions
{
    public static Mock<IUserRepository> SetupGetByIdAsync(
        this Mock<IUserRepository> mock,
        int userId,
        User user)
    {
        mock.Setup(r => r.GetByIdAsync(userId))
            .ReturnsAsync(user);
        return mock;
    }
}
```

## Test Best Practices

### DO's
- ✅ Write tests for all business logic
- ✅ Test edge cases and error scenarios
- ✅ Use meaningful test names
- ✅ Keep tests simple and focused
- ✅ Use AAA pattern consistently
- ✅ Mock external dependencies
- ✅ Clean up resources (IDisposable)
- ✅ Run tests in isolation
- ✅ Use test data builders

### DON'Ts
- ❌ Don't test framework code
- ❌ Don't test third-party libraries
- ❌ Don't share state between tests
- ❌ Don't use Thread.Sleep for timing
- ❌ Don't test private methods directly
- ❌ Don't ignore failing tests
- ❌ Don't write tests that depend on order
- ❌ Don't use real external services

## CI/CD Integration

### GitHub Actions
```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '8.0.x'
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build
      run: dotnet build --no-restore
    
    - name: Unit Tests
      run: dotnet test tests/MyApp.UnitTests --no-build --verbosity normal
    
    - name: Integration Tests
      run: dotnet test tests/MyApp.IntegrationTests --no-build --verbosity normal
    
    - name: Code Coverage
      run: |
        dotnet test --collect:"XPlat Code Coverage" --results-directory ./coverage
        reportgenerator -reports:./coverage/**/coverage.cobertura.xml -targetdir:./coverage/report
    
    - name: Upload Coverage
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage/**/coverage.cobertura.xml
```

## Test Coverage Reports

### Coverlet Configuration
```xml
<ItemGroup>
  <PackageReference Include="coverlet.collector" Version="6.0.0" />
  <PackageReference Include="coverlet.msbuild" Version="6.0.0" />
</ItemGroup>
```

### Generate Coverage Report
```bash
# Run tests with coverage
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura

# Generate HTML report
reportgenerator -reports:coverage.cobertura.xml -targetdir:coverage-report
```

## Testing Checklist

### Before Commit
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Code coverage meets minimum (80%)
- [ ] No ignored/skipped tests
- [ ] Test names are descriptive
- [ ] Tests follow AAA pattern
- [ ] Mocks are properly configured
- [ ] No test dependencies on external services

### Before Release
- [ ] All E2E tests pass
- [ ] Performance tests pass
- [ ] Load tests meet requirements
- [ ] Security tests pass
- [ ] Regression tests complete
- [ ] Browser compatibility verified
- [ ] Mobile responsiveness tested
