# Naming Conventions

## General Principles

1. **Be Descriptive**: Names should clearly indicate purpose
2. **Be Consistent**: Follow the same patterns throughout
3. **Avoid Abbreviations**: Except common ones (Id, Url, Http, etc.)
4. **Use English**: All names in English
5. **Avoid Prefixes**: No Hungarian notation (strName, intCount)

## C# Naming Conventions

### Namespaces
- **PascalCase**
- Organizational hierarchy
- No underscores

```csharp
namespace CompanyName.ProjectName.Feature;
namespace MyApp.Domain.Models;
namespace MyApp.Application.Services;
namespace MyApp.Infrastructure.Data;
```

### Classes and Records
- **PascalCase**
- Noun or noun phrase
- Descriptive and specific

```csharp
// ✅ Good
public class UserService
public class OrderRepository
public class PaymentProcessor
public record UserDto
public record CreateOrderRequest

// ❌ Bad
public class usr
public class Manager
public class Helper
```

### Interfaces
- **PascalCase**
- Prefix with `I`
- Adjective or noun phrase

```csharp
// ✅ Good
public interface IUserRepository
public interface IEmailService
public interface IPaymentGateway
public interface IDisposable
public interface IConfigurable

// ❌ Bad
public interface UserRepository  // Missing I
public interface IRepository     // Too generic
```

### Methods
- **PascalCase**
- Verb or verb phrase
- Async methods end with `Async`

```csharp
// ✅ Good
public void ProcessOrder()
public async Task<User> GetUserAsync(int id)
public bool IsValid()
public void CalculateTotal()

// ❌ Bad
public void process_order()
public void GetUser()  // Should be GetUserAsync if async
public void DoStuff()  // Not descriptive
```

### Properties
- **PascalCase**
- Noun or noun phrase
- Boolean properties use `Is`, `Has`, `Can`, `Should`

```csharp
// ✅ Good
public string FirstName { get; set; }
public int OrderId { get; set; }
public bool IsActive { get; set; }
public bool HasPermission { get; set; }
public bool CanEdit { get; set; }
public bool ShouldNotify { get; set; }

// ❌ Bad
public string firstName { get; set; }
public bool Active { get; set; }  // Should be IsActive
public bool Permission { get; set; }  // Should be HasPermission
```

### Fields
- **Private fields**: `_camelCase` with underscore prefix
- **Public fields**: PascalCase (rare, prefer properties)
- **Constants**: PascalCase
- **Static readonly**: PascalCase

```csharp
// ✅ Good
private readonly IUserRepository _userRepository;
private string _cachedValue;
private const int MaxRetries = 3;
private static readonly ImmutableArray<string> AllowedTypes = [...];

// ❌ Bad
private IUserRepository userRepository;
private IUserRepository UserRepository;
private const int MAX_RETRIES = 3;  // Use PascalCase, not UPPER_CASE
```

### Parameters and Local Variables
- **camelCase**
- Descriptive names
- Avoid single letters (except in loops: i, j, k)

```csharp
// ✅ Good
public void ProcessUser(int userId, string userName)
{
    var processedData = Transform(userData);
    for (int i = 0; i < items.Length; i++)
    {
        // ...
    }
}

// ❌ Bad
public void ProcessUser(int ID, string n)
{
    var d = Transform(x);
}
```

### Enums
- **PascalCase** for enum name
- **PascalCase** for enum values
- Singular name (unless flags)

```csharp
// ✅ Good
public enum OrderStatus
{
    Pending,
    Processing,
    Completed,
    Cancelled
}

[Flags]
public enum FilePermissions
{
    None = 0,
    Read = 1,
    Write = 2,
    Execute = 4
}

// ❌ Bad
public enum OrderStatuses  // Should be singular
{
    pending,  // Should be PascalCase
    PROCESSING  // Should be PascalCase
}
```

### Generic Type Parameters
- **PascalCase**
- Prefix with `T`
- Descriptive name for multiple parameters

```csharp
// ✅ Good
public class Repository<T> where T : class
public class Cache<TKey, TValue>
public interface IConverter<TInput, TOutput>

// ❌ Bad
public class Repository<t>
public class Cache<K, V>  // Not descriptive enough
```

### Events
- **PascalCase**
- Verb phrase
- Use `EventHandler<T>` pattern

```csharp
// ✅ Good
public event EventHandler<UserEventArgs> UserCreated;
public event EventHandler<OrderEventArgs> OrderProcessed;

// Usage
OnUserCreated(new UserEventArgs(user));

// ❌ Bad
public event EventHandler UserCreate;  // Should be past tense
public event EventHandler OnUserCreated;  // Don't prefix with On
```

### Delegates
- **PascalCase**
- Descriptive name
- Consider using `Action<T>` or `Func<T>` instead

```csharp
// ✅ Good
public delegate void ProcessUserDelegate(User user);
public delegate bool ValidateInputDelegate(string input);

// Better - use built-in
public Action<User> ProcessUser;
public Func<string, bool> ValidateInput;
```

## Project and Solution Structure

### Project Names
- **PascalCase**
- Follow layered architecture naming

```
MyApp.sln
├── src/
│   ├── MyApp.Domain/
│   ├── MyApp.Application/
│   ├── MyApp.Infrastructure/
│   ├── MyApp.Web/
│   └── MyApp.Api/
├── tests/
│   ├── MyApp.UnitTests/
│   ├── MyApp.IntegrationTests/
│   └── MyApp.E2ETests/
└── docs/
```

### File Names
- Match the primary class/interface name
- One class per file (with exceptions for nested classes)

```
UserService.cs
IUserRepository.cs
UserDto.cs
CreateUserRequest.cs
```

## Database Naming Conventions

### Tables
- **PascalCase**
- Plural nouns

```sql
-- ✅ Good
Users
Orders
OrderItems
ProductCategories

-- ❌ Bad
user
tblUsers
order_items
```

### Columns
- **PascalCase**
- Descriptive names
- Foreign keys: `[TableName]Id`

```sql
-- ✅ Good
CREATE TABLE Users (
    Id INT PRIMARY KEY,
    FirstName NVARCHAR(100),
    LastName NVARCHAR(100),
    Email NVARCHAR(255),
    CreatedAt DATETIME,
    IsActive BIT
)

CREATE TABLE Orders (
    Id INT PRIMARY KEY,
    UserId INT,  -- FK to Users
    OrderDate DATETIME,
    TotalAmount DECIMAL(18,2)
)

-- ❌ Bad
CREATE TABLE users (
    user_id INT,
    fname VARCHAR(100),
    active BIT  -- Should be IsActive
)
```

### Stored Procedures
- **PascalCase**
- Prefix: `usp` (user stored procedure)
- Verb + Entity pattern

```sql
-- ✅ Good
uspGetUserById
uspCreateOrder
uspUpdateUserStatus
uspDeleteInactiveUsers

-- ❌ Bad
GetUser
sp_GetUser  -- sp_ is system reserved
user_create
```

### Indexes
- Prefix: `IX_` for non-clustered
- `[TableName]_[ColumnNames]`

```sql
-- ✅ Good
IX_Users_Email
IX_Orders_UserId_OrderDate
IX_Products_CategoryId

-- ❌ Bad
idx_users_email
UserEmailIndex
```

## API Naming Conventions

### Controller Names
- **PascalCase**
- Plural entity name + "Controller"

```csharp
// ✅ Good
public class UsersController : ControllerBase
public class OrdersController : ControllerBase
public class ProductCategoriesController : ControllerBase

// ❌ Bad
public class UserController  // Should be plural
public class UsersAPIController  // Don't add API suffix
```

### API Routes
- **lowercase**
- **kebab-case** for multi-word resources
- Plural nouns for resources

```csharp
// ✅ Good
[Route("api/v{version:apiVersion}/users")]
[Route("api/v{version:apiVersion}/product-categories")]
[Route("api/v{version:apiVersion}/order-items")]

// ❌ Bad
[Route("api/Users")]  // Should be lowercase
[Route("api/user")]   // Should be plural
[Route("api/productCategories")]  // Use kebab-case
```

### API Actions
- **RESTful conventions**

```csharp
// ✅ Good
[HttpGet]                          // GET /api/users
[HttpGet("{id}")]                  // GET /api/users/123
[HttpPost]                         // POST /api/users
[HttpPut("{id}")]                  // PUT /api/users/123
[HttpDelete("{id}")]               // DELETE /api/users/123
[HttpGet("{id}/orders")]           // GET /api/users/123/orders

// ❌ Bad
[HttpGet("GetAllUsers")]           // Don't use verbs in URL
[HttpPost("CreateUser")]           // Use POST /users instead
```

### Query Parameters
- **camelCase**

```csharp
// ✅ Good
GET /api/users?pageNumber=1&pageSize=10&sortBy=name
GET /api/products?categoryId=5&isActive=true

// ❌ Bad
GET /api/users?PageNumber=1  // Should be camelCase
GET /api/users?page_size=10  // Should be camelCase
```

### Request/Response DTOs
- **PascalCase**
- Suffixes: `Request`, `Response`, `Dto`

```csharp
// ✅ Good
public record CreateUserRequest(string FirstName, string LastName, string Email);
public record UserResponse(int Id, string FullName, string Email);
public record UserDto(int Id, string FirstName, string LastName);

// ❌ Bad
public record UserCreateModel
public record UserData
public record User  // Conflicts with domain entity
```

## Frontend Naming Conventions (React/Vue + TypeScript)

### Components (React)
- **PascalCase**
- Descriptive names

```typescript
// ✅ Good
UserProfile.tsx
OrderList.tsx
ProductCard.tsx
NavigationBar.tsx

// Component
export const UserProfile: React.FC<UserProfileProps> = ({ userId }) => {
  // ...
}

// ❌ Bad
userProfile.tsx
profile.tsx  // Not descriptive
```

### Components (Vue)
- **PascalCase** or **kebab-case** in template
- PascalCase in script

```vue
<!-- ✅ Good -->
<template>
  <user-profile :user-id="userId" />
  <!-- or -->
  <UserProfile :userId="userId" />
</template>

<script setup lang="ts">
// UserProfile.vue
defineProps<{
  userId: number
}>()
</script>

<!-- ❌ Bad -->
<!-- user_profile.vue -->
```

### TypeScript Interfaces/Types
- **PascalCase**
- Prefix interface with `I` (optional, community is divided)
- Use descriptive names

```typescript
// ✅ Good
interface User {
  id: number;
  firstName: string;
  lastName: string;
}

type UserFormData = {
  firstName: string;
  lastName: string;
  email: string;
}

interface IUserService {
  getUser(id: number): Promise<User>;
}

// ❌ Bad
interface user  // Should be PascalCase
type FormData  // Too generic
```

### Functions and Variables
- **camelCase**
- Verb for functions
- Noun for variables

```typescript
// ✅ Good
const userName = 'John';
const isActive = true;
const userList: User[] = [];

function getUserById(id: number): Promise<User> {
  // ...
}

const handleSubmit = (data: FormData) => {
  // ...
}

// ❌ Bad
const UserName = 'John';
const get_user = (id) => { };
function submit() { }  // Not descriptive
```

### CSS/SCSS Classes
- **kebab-case**
- BEM methodology (optional but recommended)

```scss
// ✅ Good
.user-profile { }
.user-profile__header { }
.user-profile__avatar { }
.user-profile--compact { }

.btn-primary { }
.card-container { }

// ❌ Bad
.UserProfile { }
.user_profile { }
.userProfile { }
```

### Constants
- **UPPER_SNAKE_CASE** for true constants
- **PascalCase** for configuration objects

```typescript
// ✅ Good
const MAX_RETRY_ATTEMPTS = 3;
const API_BASE_URL = 'https://api.example.com';

const AppConfig = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
} as const;

// ❌ Bad
const max_retry_attempts = 3;
const apiBaseUrl = 'https://api.example.com';  // Should be UPPER_CASE
```

## Configuration Files

### Environment Variables
- **UPPER_SNAKE_CASE**

```bash
# ✅ Good
DATABASE_CONNECTION_STRING
API_KEY
JWT_SECRET
SMTP_HOST

# ❌ Bad
databaseConnectionString
api-key
JwtSecret
```

### Configuration Sections
- **PascalCase**

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "..."
  },
  "JwtSettings": {
    "SecretKey": "...",
    "ExpirationMinutes": 60
  },
  "EmailSettings": {
    "SmtpHost": "smtp.example.com"
  }
}
```

## Test Naming Conventions

### Test Classes
- Class name + `Tests` suffix

```csharp
// ✅ Good
public class UserServiceTests
public class OrderRepositoryTests
public class PaymentControllerTests

// ❌ Bad
public class TestUserService
public class UserServiceTest  // Should be plural
```

### Test Methods
- **Pattern**: `MethodName_Scenario_ExpectedBehavior`
- **PascalCase**

```csharp
// ✅ Good
[Fact]
public void GetUser_WithValidId_ReturnsUser()

[Fact]
public void GetUser_WithInvalidId_ReturnsNull()

[Fact]
public void CreateUser_WithDuplicateEmail_ThrowsException()

// ❌ Bad
[Fact]
public void Test1()

[Fact]
public void GetUserTest()

[Fact]
public void get_user_with_valid_id()  // Should be PascalCase
```

## Summary Cheat Sheet

| Element | Convention | Example |
|---------|-----------|---------|
| Namespace | PascalCase | `MyApp.Domain.Models` |
| Class | PascalCase | `UserService` |
| Interface | PascalCase + I prefix | `IUserRepository` |
| Method | PascalCase | `GetUserAsync()` |
| Property | PascalCase | `FirstName` |
| Private Field | _camelCase | `_userRepository` |
| Parameter | camelCase | `userId` |
| Local Variable | camelCase | `orderTotal` |
| Constant | PascalCase | `MaxRetries` |
| Enum | PascalCase (singular) | `OrderStatus` |
| Database Table | PascalCase (plural) | `Users` |
| Database Column | PascalCase | `FirstName` |
| API Route | lowercase kebab-case | `/api/users` |
| DTO | PascalCase + suffix | `CreateUserRequest` |
| React Component | PascalCase | `UserProfile` |
| CSS Class | kebab-case | `user-profile` |
| Environment Var | UPPER_SNAKE_CASE | `DATABASE_URL` |
| Test Method | PascalCase | `GetUser_WithValidId_ReturnsUser` |
