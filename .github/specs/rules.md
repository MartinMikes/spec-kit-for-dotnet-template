# Development Rules

## Code Review Rules

### 1. Pull Request Requirements
- ✅ All PRs must have a clear description of changes
- ✅ Link related issues using keywords (Fixes #123, Closes #456)
- ✅ At least one approval required before merging
- ✅ All CI/CD checks must pass
- ✅ No merge conflicts
- ✅ Branch must be up-to-date with target branch

### 2. Code Review Checklist
Reviewers must verify:
- [ ] Code follows naming conventions
- [ ] No code duplication
- [ ] Error handling is appropriate
- [ ] Tests are included and passing
- [ ] No security vulnerabilities introduced
- [ ] Performance implications considered
- [ ] Documentation updated if needed
- [ ] No commented-out code committed
- [ ] No debug code or console logs

## Git Workflow Rules

### 1. Branch Naming Convention
```
feature/SHORT-DESCRIPTION      # New features
bugfix/SHORT-DESCRIPTION       # Bug fixes
hotfix/SHORT-DESCRIPTION       # Production hotfixes
release/VERSION               # Release branches
refactor/SHORT-DESCRIPTION    # Code refactoring
docs/SHORT-DESCRIPTION        # Documentation only
```

Examples:
- `feature/user-authentication`
- `bugfix/login-validation-error`
- `hotfix/security-patch`

### 2. Commit Message Format
Follow Conventional Commits:
```
<type>(<scope>): <subject>

<body>

<footer>
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks
- `perf`: Performance improvements

Examples:
```
feat(auth): add JWT token authentication

Implements JWT-based authentication for API endpoints.
Includes refresh token mechanism and token expiration.

Closes #123

---

fix(users): resolve null reference in user profile

The user profile page was throwing NullReferenceException
when the user had no profile picture.

Fixes #456
```

### 3. Branching Strategy
- `main`: Production-ready code
- `develop`: Integration branch for features
- Feature branches off `develop`
- Hotfix branches off `main`
- Merge feature → develop → main

## Code Quality Rules

### 1. No Code Smells
Forbidden patterns:
- ❌ God classes (>500 lines)
- ❌ Long methods (>50 lines)
- ❌ Deep nesting (>3 levels)
- ❌ Magic numbers (use constants)
- ❌ Global state
- ❌ Circular dependencies

### 2. SOLID Principles (Mandatory)
- **S**ingle Responsibility: One class, one reason to change
- **O**pen/Closed: Open for extension, closed for modification
- **L**iskov Substitution: Subtypes must be substitutable
- **I**nterface Segregation: Many specific interfaces > one general
- **D**ependency Inversion: Depend on abstractions, not concretions

### 3. DRY Principle
- No duplicated code blocks
- Extract common logic into reusable methods
- Use inheritance or composition appropriately

### 4. Code Complexity
- Cyclomatic complexity < 10 per method
- Cognitive complexity < 15 per method
- Use SonarQube or similar tools to measure

## Testing Rules

### 1. Test Coverage Requirements
- Minimum 80% code coverage
- 100% coverage for critical business logic
- All public APIs must have tests
- All bug fixes must include regression tests

### 2. Test Naming Convention
```csharp
[MethodName]_[Scenario]_[ExpectedBehavior]

// Examples:
GetUser_WithValidId_ReturnsUser()
GetUser_WithInvalidId_ReturnsNull()
CreateUser_WithDuplicateEmail_ThrowsException()
```

### 3. Test Structure (AAA Pattern)
```csharp
[Fact]
public void TestName()
{
    // Arrange - Setup test data and dependencies
    
    // Act - Execute the method being tested
    
    // Assert - Verify the results
}
```

### 4. Test Categories
- **Unit Tests**: Fast, isolated, no dependencies
- **Integration Tests**: Test component interactions
- **E2E Tests**: Test complete user flows
- **Performance Tests**: Benchmark critical operations

### 5. Mocking Rules
- Mock external dependencies only
- Don't mock value objects or DTOs
- Verify mock interactions when appropriate
- Use `Moq` or `NSubstitute` for mocking

## Security Rules

### 1. Authentication & Authorization
- ✅ Always authenticate users for protected resources
- ✅ Use role-based or policy-based authorization
- ✅ Implement JWT tokens with appropriate expiration
- ✅ Use HTTPS only in production
- ❌ Never store passwords in plain text
- ❌ Never log sensitive information

### 2. Input Validation
- ✅ Validate all user input server-side
- ✅ Sanitize inputs to prevent XSS
- ✅ Use parameterized queries (prevent SQL injection)
- ✅ Implement rate limiting on APIs
- ✅ Validate file uploads (type, size)

### 3. Secret Management
- ✅ Use Azure Key Vault, AWS Secrets Manager, or similar
- ✅ Store secrets in environment variables or secret managers
- ❌ Never commit secrets to version control
- ❌ Never hardcode API keys, connection strings, or passwords
- ✅ Use .gitignore for sensitive files
- ✅ Rotate secrets regularly

### 4. Dependency Management
- ✅ Update dependencies monthly
- ✅ Review security advisories
- ✅ Use Dependabot or similar tools
- ❌ Don't use deprecated packages
- ✅ Audit packages before adding

## Performance Rules

### 1. Database Operations
- ✅ Use async/await for database calls
- ✅ Implement pagination for large datasets
- ✅ Use database indexes appropriately
- ✅ Optimize N+1 query problems
- ❌ Don't load unnecessary data
- ✅ Use projection when possible

### 2. Caching Strategy
- ✅ Cache frequently accessed, rarely changed data
- ✅ Set appropriate cache expiration
- ✅ Use distributed cache for multi-instance apps
- ✅ Implement cache invalidation strategy
- ❌ Don't cache user-specific sensitive data

### 3. API Performance
- ✅ Implement response compression
- ✅ Use ETags for caching
- ✅ Implement request throttling
- ✅ Return minimal data (use DTOs)
- ✅ Use asynchronous operations

## Documentation Rules

### 1. Code Documentation
- ✅ Document public APIs with XML comments
- ✅ Include examples in documentation
- ✅ Document complex algorithms
- ❌ Don't state the obvious
- ✅ Keep documentation up-to-date

```csharp
/// <summary>
/// Retrieves a user by their unique identifier.
/// </summary>
/// <param name="userId">The unique identifier of the user.</param>
/// <returns>The user if found; otherwise, null.</returns>
/// <exception cref="ArgumentException">Thrown when userId is less than 1.</exception>
public async Task<User?> GetUserAsync(int userId)
{
    if (userId < 1)
        throw new ArgumentException("User ID must be greater than 0", nameof(userId));
    
    return await _context.Users.FindAsync(userId);
}
```

### 2. README Requirements
Every project must have:
- Project description
- Prerequisites
- Installation instructions
- Configuration guide
- Usage examples
- API documentation link
- Contributing guidelines
- License information

### 3. Architecture Documentation
- Maintain architecture decision records (ADRs)
- Document system architecture diagrams
- Keep API documentation current (Swagger/OpenAPI)
- Document deployment procedures

## Deployment Rules

### 1. CI/CD Pipeline
- ✅ Automated build on every commit
- ✅ Run all tests in pipeline
- ✅ Automated deployment to staging
- ✅ Manual approval for production
- ✅ Rollback capability

### 2. Environment Configuration
- ✅ Use environment variables
- ✅ Separate configs for dev/staging/prod
- ✅ Never use production data in development
- ✅ Implement feature flags for gradual rollout

### 3. Release Process
- ✅ Semantic versioning (MAJOR.MINOR.PATCH)
- ✅ Tag releases in git
- ✅ Maintain CHANGELOG.md
- ✅ Create release notes
- ✅ Database migration strategy

## Code Formatting Rules

### 1. Use EditorConfig
All projects must include `.editorconfig`:
```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.{cs,csx,vb,vbx}]
indent_size = 4
indent_style = space

[*.{json,js,jsx,ts,tsx,yml,yaml}]
indent_size = 2
indent_style = space
```

### 2. Linting
- ✅ Use StyleCop or SonarAnalyzer
- ✅ Configure .editorconfig
- ✅ Run linter in CI/CD
- ✅ Fix warnings, not just errors

### 3. Code Style
- ✅ Use `var` when type is obvious
- ✅ Use expression-bodied members when appropriate
- ✅ Place `using` statements inside namespace
- ✅ Order members: fields, constructors, properties, methods

## Forbidden Practices

### Absolutely Prohibited
1. ❌ Committing commented-out code
2. ❌ Using `#region` directives
3. ❌ Catching and swallowing exceptions without logging
4. ❌ Using `async void` (except event handlers)
5. ❌ Blocking async code with `.Result` or `.Wait()`
6. ❌ Using `DateTime.Now` (use `DateTime.UtcNow`)
7. ❌ Hard-coding configuration values
8. ❌ Disabling security features without justification
9. ❌ Copying and pasting code without refactoring
10. ❌ Committing merge conflicts

## Exception Handling

### When exceptions are violations
Rules can be broken only when:
- ✅ Documented with clear justification
- ✅ Approved in code review
- ✅ Added to technical debt backlog
- ✅ Planned for future refactoring

Mark exceptions with:
```csharp
// TODO: Technical Debt - Refactor this [JIRA-123]
// Justification: Performance optimization needed for release
```
