# Security Specifications

## Security Principles

### Defense in Depth
Multiple layers of security controls throughout the application.

### Least Privilege
Grant minimum necessary permissions for each role and operation.

### Zero Trust
Never trust, always verify - validate all inputs and authenticate all requests.

### Secure by Default
Security features enabled by default, opt-out rather than opt-in.

## Authentication

### JWT Token Authentication

#### Token Structure
```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user@example.com",
    "userId": "123",
    "roles": ["User"],
    "exp": 1735689600,
    "iat": 1735603200
  }
}
```

#### Implementation
```csharp
public class JwtTokenService
{
    private readonly IConfiguration _configuration;

    public string GenerateToken(User user)
    {
        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, user.Email),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            new Claim("userId", user.Id.ToString()),
            new Claim(ClaimTypes.Role, user.Role)
        };

        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_configuration["Jwt:SecretKey"]));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _configuration["Jwt:Issuer"],
            audience: _configuration["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(60),
            signingCredentials: creds
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }

    public string GenerateRefreshToken()
    {
        var randomNumber = new byte[32];
        using var rng = RandomNumberGenerator.Create();
        rng.GetBytes(randomNumber);
        return Convert.ToBase64String(randomNumber);
    }
}
```

#### Token Configuration
```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.SaveToken = true;
    options.RequireHttpsMetadata = true;
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ClockSkew = TimeSpan.Zero,
        ValidIssuer = builder.Configuration["Jwt:Issuer"],
        ValidAudience = builder.Configuration["Jwt:Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(builder.Configuration["Jwt:SecretKey"]))
    };

    options.Events = new JwtBearerEvents
    {
        OnAuthenticationFailed = context =>
        {
            if (context.Exception.GetType() == typeof(SecurityTokenExpiredException))
            {
                context.Response.Headers.Add("Token-Expired", "true");
            }
            return Task.CompletedTask;
        }
    };
});
```

### Refresh Token Flow
```csharp
public class AuthController : ControllerBase
{
    [HttpPost("login")]
    public async Task<ActionResult<AuthResponse>> Login(LoginRequest request)
    {
        var user = await _authService.ValidateUserAsync(request.Email, request.Password);
        if (user == null)
            return Unauthorized();

        var accessToken = _jwtService.GenerateToken(user);
        var refreshToken = _jwtService.GenerateRefreshToken();

        await _authService.SaveRefreshTokenAsync(user.Id, refreshToken);

        return Ok(new AuthResponse
        {
            AccessToken = accessToken,
            RefreshToken = refreshToken,
            ExpiresIn = 3600
        });
    }

    [HttpPost("refresh")]
    public async Task<ActionResult<AuthResponse>> RefreshToken(RefreshTokenRequest request)
    {
        var principal = _jwtService.GetPrincipalFromExpiredToken(request.AccessToken);
        var userId = principal.FindFirst("userId")?.Value;

        var isValid = await _authService.ValidateRefreshTokenAsync(
            int.Parse(userId), request.RefreshToken);

        if (!isValid)
            return Unauthorized();

        var user = await _userService.GetUserAsync(int.Parse(userId));
        var newAccessToken = _jwtService.GenerateToken(user);
        var newRefreshToken = _jwtService.GenerateRefreshToken();

        await _authService.SaveRefreshTokenAsync(user.Id, newRefreshToken);

        return Ok(new AuthResponse
        {
            AccessToken = newAccessToken,
            RefreshToken = newRefreshToken,
            ExpiresIn = 3600
        });
    }
}
```

## Authorization

### Role-Based Access Control (RBAC)
```csharp
public enum UserRole
{
    Admin,
    Manager,
    User,
    Guest
}

[Authorize(Roles = "Admin,Manager")]
[HttpGet("admin/users")]
public async Task<ActionResult<IEnumerable<UserDto>>> GetAllUsers()
{
    // Only Admin and Manager can access
}
```

### Policy-Based Authorization
```csharp
// Define policies
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAdmin", policy =>
        policy.RequireRole("Admin"));

    options.AddPolicy("CanEditUser", policy =>
        policy.Requirements.Add(new CanEditUserRequirement()));

    options.AddPolicy("MinimumAge", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(18)));

    options.AddPolicy("EmailVerified", policy =>
        policy.RequireClaim("EmailVerified", "true"));
});

// Custom requirement
public class CanEditUserRequirement : IAuthorizationRequirement { }

public class CanEditUserHandler : AuthorizationHandler<CanEditUserRequirement, User>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        CanEditUserRequirement requirement,
        User resource)
    {
        var userId = context.User.FindFirst("userId")?.Value;
        var isAdmin = context.User.IsInRole("Admin");

        if (isAdmin || userId == resource.Id.ToString())
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}

// Usage
[Authorize(Policy = "CanEditUser")]
[HttpPut("{id}")]
public async Task<IActionResult> UpdateUser(int id, UpdateUserRequest request)
{
    // Authorization happens before this executes
}
```

### Resource-Based Authorization
```csharp
[HttpPut("documents/{id}")]
public async Task<IActionResult> UpdateDocument(int id, UpdateDocumentRequest request)
{
    var document = await _documentService.GetByIdAsync(id);
    
    var authResult = await _authorizationService.AuthorizeAsync(
        User, document, "CanEditDocument");

    if (!authResult.Succeeded)
        return Forbid();

    // Update document
    await _documentService.UpdateAsync(id, request);
    return NoContent();
}
```

## Input Validation & Sanitization

### Model Validation with FluentValidation
```csharp
public class CreateUserRequestValidator : AbstractValidator<CreateUserRequest>
{
    public CreateUserRequestValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(255);

        RuleFor(x => x.Password)
            .NotEmpty()
            .MinimumLength(8)
            .Matches(@"[A-Z]").WithMessage("Password must contain uppercase letter")
            .Matches(@"[a-z]").WithMessage("Password must contain lowercase letter")
            .Matches(@"[0-9]").WithMessage("Password must contain number")
            .Matches(@"[^a-zA-Z0-9]").WithMessage("Password must contain special character");

        RuleFor(x => x.FirstName)
            .NotEmpty()
            .MaximumLength(100)
            .Matches(@"^[a-zA-Z\s]+$").WithMessage("First name can only contain letters");
    }
}
```

### SQL Injection Prevention
```csharp
// ✅ Good - Parameterized query (EF Core)
var users = await _context.Users
    .Where(u => u.Email == email)
    .ToListAsync();

// ✅ Good - Parameterized raw SQL
var users = await _context.Users
    .FromSqlRaw("SELECT * FROM Users WHERE Email = {0}", email)
    .ToListAsync();

// ❌ Bad - String concatenation
var users = await _context.Users
    .FromSqlRaw($"SELECT * FROM Users WHERE Email = '{email}'")
    .ToListAsync();
```

### XSS Prevention
```csharp
// Encode output in views
@Html.Encode(Model.UserInput)

// Sanitize HTML input
public class HtmlSanitizer
{
    private static readonly HtmlSanitizer _sanitizer = new();

    public static string Sanitize(string html)
    {
        return _sanitizer.Sanitize(html);
    }
}

// Usage
var sanitizedHtml = HtmlSanitizer.Sanitize(userInput);
```

### CSRF Protection
```csharp
// Automatically enabled in ASP.NET Core
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});

// In Razor views
<form method="post">
    @Html.AntiForgeryToken()
    <!-- form fields -->
</form>
```

## Password Security

### Password Hashing
```csharp
public class PasswordHasher
{
    private const int SaltSize = 16;
    private const int HashSize = 32;
    private const int Iterations = 100000;

    public static string Hash(string password)
    {
        using var rng = RandomNumberGenerator.Create();
        var salt = new byte[SaltSize];
        rng.GetBytes(salt);

        using var pbkdf2 = new Rfc2898DeriveBytes(
            password,
            salt,
            Iterations,
            HashAlgorithmName.SHA256);

        var hash = pbkdf2.GetBytes(HashSize);

        var hashBytes = new byte[SaltSize + HashSize];
        Array.Copy(salt, 0, hashBytes, 0, SaltSize);
        Array.Copy(hash, 0, hashBytes, SaltSize, HashSize);

        return Convert.ToBase64String(hashBytes);
    }

    public static bool Verify(string password, string hashedPassword)
    {
        var hashBytes = Convert.FromBase64String(hashedPassword);

        var salt = new byte[SaltSize];
        Array.Copy(hashBytes, 0, salt, 0, SaltSize);

        using var pbkdf2 = new Rfc2898DeriveBytes(
            password,
            salt,
            Iterations,
            HashAlgorithmName.SHA256);

        var hash = pbkdf2.GetBytes(HashSize);

        for (int i = 0; i < HashSize; i++)
        {
            if (hashBytes[i + SaltSize] != hash[i])
                return false;
        }

        return true;
    }
}
```

### Password Policy
```csharp
public class PasswordPolicy
{
    public const int MinimumLength = 8;
    public const int MaximumLength = 128;
    public const bool RequireUppercase = true;
    public const bool RequireLowercase = true;
    public const bool RequireDigit = true;
    public const bool RequireSpecialChar = true;
    public const int PasswordHistoryCount = 5; // Remember last 5 passwords
    public const int PasswordExpiryDays = 90;
}
```

## Data Protection

### Encryption at Rest
```csharp
public class EncryptionService
{
    private readonly IConfiguration _configuration;

    public string Encrypt(string plainText)
    {
        var key = _configuration["Encryption:Key"];
        using var aes = Aes.Create();
        aes.Key = Convert.FromBase64String(key);
        aes.GenerateIV();

        using var encryptor = aes.CreateEncryptor();
        using var ms = new MemoryStream();
        
        ms.Write(aes.IV, 0, aes.IV.Length);
        
        using (var cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Write))
        using (var sw = new StreamWriter(cs))
        {
            sw.Write(plainText);
        }

        return Convert.ToBase64String(ms.ToArray());
    }

    public string Decrypt(string cipherText)
    {
        var key = _configuration["Encryption:Key"];
        var buffer = Convert.FromBase64String(cipherText);
        
        using var aes = Aes.Create();
        aes.Key = Convert.FromBase64String(key);

        var iv = new byte[aes.IV.Length];
        Array.Copy(buffer, iv, iv.Length);
        aes.IV = iv;

        using var decryptor = aes.CreateDecryptor();
        using var ms = new MemoryStream(buffer, iv.Length, buffer.Length - iv.Length);
        using var cs = new CryptoStream(ms, decryptor, CryptoStreamMode.Read);
        using var sr = new StreamReader(cs);
        
        return sr.ReadToEnd();
    }
}
```

### Sensitive Data in Logs
```csharp
public class SensitiveDataFilter
{
    private static readonly string[] SensitiveKeys = 
    {
        "password", "token", "secret", "apikey", "creditcard", "ssn"
    };

    public static object FilterSensitiveData(object data)
    {
        if (data == null) return null;

        var json = JsonSerializer.Serialize(data);
        var doc = JsonDocument.Parse(json);
        var filtered = FilterElement(doc.RootElement);
        
        return JsonSerializer.Deserialize<object>(filtered.ToString());
    }

    private static JsonElement FilterElement(JsonElement element)
    {
        if (element.ValueKind == JsonValueKind.Object)
        {
            var obj = new Dictionary<string, object>();
            foreach (var prop in element.EnumerateObject())
            {
                var key = prop.Name.ToLower();
                if (SensitiveKeys.Any(k => key.Contains(k)))
                {
                    obj[prop.Name] = "***REDACTED***";
                }
                else
                {
                    obj[prop.Name] = FilterElement(prop.Value);
                }
            }
            return JsonSerializer.SerializeToElement(obj);
        }
        return element;
    }
}
```

## HTTPS & TLS

### Force HTTPS
```csharp
// Program.cs
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
    app.UseHttpsRedirection();
}

// Strict Transport Security
app.Use(async (context, next) =>
{
    context.Response.Headers.Add(
        "Strict-Transport-Security", 
        "max-age=31536000; includeSubDomains");
    await next();
});
```

### Security Headers
```csharp
public class SecurityHeadersMiddleware
{
    private readonly RequestDelegate _next;

    public SecurityHeadersMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Prevent clickjacking
        context.Response.Headers.Add("X-Frame-Options", "DENY");
        
        // Prevent MIME type sniffing
        context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
        
        // XSS Protection
        context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
        
        // Content Security Policy
        context.Response.Headers.Add("Content-Security-Policy",
            "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';");
        
        // Referrer Policy
        context.Response.Headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");
        
        // Permissions Policy
        context.Response.Headers.Add("Permissions-Policy", 
            "geolocation=(), microphone=(), camera=()");

        await _next(context);
    }
}

// Register middleware
app.UseMiddleware<SecurityHeadersMiddleware>();
```

## Rate Limiting & Throttling

### IP-Based Rate Limiting
```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(context =>
    {
        var ipAddress = context.Connection.RemoteIpAddress?.ToString() ?? "unknown";
        
        return RateLimitPartition.GetFixedWindowLimiter(ipAddress, _ =>
            new FixedWindowRateLimiterOptions
            {
                PermitLimit = 100,
                Window = TimeSpan.FromMinutes(1),
                QueueProcessingOrder = QueueProcessingOrder.OldestFirst,
                QueueLimit = 2
            });
    });

    options.RejectionStatusCode = StatusCodes.Status429TooManyRequests;
});

app.UseRateLimiter();
```

### Endpoint-Specific Rate Limiting
```csharp
[EnableRateLimiting("api")]
[HttpPost("login")]
public async Task<IActionResult> Login(LoginRequest request)
{
    // ...
}

// Policy configuration
builder.Services.AddRateLimiter(options =>
{
    options.AddPolicy("api", context =>
        RateLimitPartition.GetSlidingWindowLimiter(
            context.Connection.RemoteIpAddress?.ToString() ?? "unknown",
            _ => new SlidingWindowRateLimiterOptions
            {
                PermitLimit = 10,
                Window = TimeSpan.FromSeconds(10),
                SegmentsPerWindow = 2
            }));
});
```

## Secrets Management

### Azure Key Vault
```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{builder.Configuration["KeyVault:Name"]}.vault.azure.net/"),
    new DefaultAzureCredential());

// Usage
var apiKey = builder.Configuration["ApiKey"]; // Retrieved from Key Vault
```

### User Secrets (Development Only)
```bash
# Initialize user secrets
dotnet user-secrets init

# Set secret
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=..."

# List secrets
dotnet user-secrets list
```

```csharp
// Access in code
if (builder.Environment.IsDevelopment())
{
    builder.Configuration.AddUserSecrets<Program>();
}
```

### Environment Variables
```csharp
// appsettings.json - no secrets!
{
  "ConnectionStrings": {
    "DefaultConnection": ""
  }
}

// Set via environment variable
// ConnectionStrings__DefaultConnection=Server=...

// Access
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

## API Security

### CORS Configuration
```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigins", policy =>
    {
        policy.WithOrigins("https://myapp.com", "https://admin.myapp.com")
            .AllowAnyMethod()
            .AllowAnyHeader()
            .AllowCredentials();
    });
});

app.UseCors("AllowSpecificOrigins");
```

### API Key Authentication
```csharp
public class ApiKeyAuthenticationHandler : AuthenticationHandler<ApiKeyAuthenticationOptions>
{
    private const string ApiKeyHeaderName = "X-API-Key";

    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        if (!Request.Headers.TryGetValue(ApiKeyHeaderName, out var apiKeyHeaderValues))
        {
            return AuthenticateResult.Fail("API Key not found");
        }

        var apiKey = apiKeyHeaderValues.FirstOrDefault();
        var validApiKey = Configuration["ApiKey"];

        if (apiKey != validApiKey)
        {
            return AuthenticateResult.Fail("Invalid API Key");
        }

        var claims = new[] { new Claim(ClaimTypes.Name, "API Client") };
        var identity = new ClaimsIdentity(claims, Scheme.Name);
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, Scheme.Name);

        return AuthenticateResult.Success(ticket);
    }
}
```

## Security Logging & Monitoring

### Security Event Logging
```csharp
public class SecurityEventLogger
{
    private readonly ILogger<SecurityEventLogger> _logger;

    public void LogFailedLogin(string email, string ipAddress)
    {
        _logger.LogWarning(
            "Failed login attempt for {Email} from {IpAddress}",
            email, ipAddress);
    }

    public void LogSuccessfulLogin(int userId, string ipAddress)
    {
        _logger.LogInformation(
            "Successful login for user {UserId} from {IpAddress}",
            userId, ipAddress);
    }

    public void LogPasswordChange(int userId)
    {
        _logger.LogInformation(
            "Password changed for user {UserId}",
            userId);
    }

    public void LogUnauthorizedAccess(int userId, string resource)
    {
        _logger.LogWarning(
            "Unauthorized access attempt by user {UserId} to {Resource}",
            userId, resource);
    }
}
```

## Security Checklist

### Development Phase
- [ ] Enable nullable reference types
- [ ] Use parameterized queries
- [ ] Implement input validation
- [ ] Hash passwords with strong algorithm
- [ ] Use HTTPS everywhere
- [ ] Implement authentication & authorization
- [ ] Sanitize all user inputs
- [ ] Set security headers
- [ ] Implement rate limiting
- [ ] Use secure session management

### Pre-Production
- [ ] Security code review completed
- [ ] Dependency vulnerability scan passed
- [ ] No secrets in source code
- [ ] HTTPS certificates configured
- [ ] Security headers tested
- [ ] Authentication tested
- [ ] Authorization tested
- [ ] Input validation tested
- [ ] Error handling doesn't leak info
- [ ] Logging configured properly

### Production
- [ ] Web Application Firewall (WAF) enabled
- [ ] DDoS protection enabled
- [ ] Regular security updates scheduled
- [ ] Monitoring and alerting configured
- [ ] Incident response plan ready
- [ ] Regular security audits scheduled
- [ ] Backup and recovery tested
- [ ] Access logs retention configured
