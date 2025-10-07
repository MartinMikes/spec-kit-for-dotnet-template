# Contributing Guide

Thank you for considering contributing to this .NET template project! This guide will help you understand our development process and standards.

## Getting Started

1. **Fork the repository** and clone it locally
2. **Read the specifications** in `.github/specs/`
3. **Set up your development environment** following the README
4. **Create a feature branch** from `develop`

## Code Standards

All contributions must adhere to:

- **[Constitution](.github/specs/constitution.md)** - Core principles
- **[Best Practices](.github/specs/best-practices.md)** - .NET 10 coding standards
- **[Rules](.github/specs/rules.md)** - Development rules and Git workflow
- **[Naming Conventions](.github/specs/naming-conventions.md)** - Naming standards

## Development Workflow

### 1. Create a Feature Branch

```bash
git checkout develop
git pull origin develop
git checkout -b feature/your-feature-name
```

Branch naming:
- `feature/` - New features
- `bugfix/` - Bug fixes
- `hotfix/` - Production hotfixes
- `docs/` - Documentation only
- `refactor/` - Code refactoring

### 2. Make Changes

- Follow the coding standards in `.editorconfig`
- Write tests for all new functionality
- Update documentation as needed
- Keep changes focused and minimal

### 3. Commit Your Changes

Use conventional commit messages:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation changes
- `style` - Code style/formatting
- `refactor` - Code refactoring
- `test` - Adding tests
- `chore` - Maintenance tasks

**Example:**
```
feat(auth): add JWT token authentication

Implements JWT-based authentication for API endpoints.
Includes refresh token mechanism and token expiration.

Closes #123
```

### 4. Test Your Changes

```bash
# Restore dependencies
dotnet restore

# Build
dotnet build

# Run tests
dotnet test

# Check formatting
dotnet format --verify-no-changes
```

### 5. Push and Create Pull Request

```bash
git push origin feature/your-feature-name
```

Then create a Pull Request on GitHub.

## Pull Request Guidelines

### PR Title
Use the same format as commit messages:
```
feat(auth): add JWT token authentication
```

### PR Description Template
```markdown
## Description
Brief description of the changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-reviewed the code
- [ ] Commented complex code
- [ ] Updated documentation
- [ ] Added tests
- [ ] All tests passing
- [ ] No new warnings

## Related Issues
Closes #123

## Screenshots (if applicable)
```

### Code Review Process

Your PR will be reviewed for:

1. **Code Quality**
   - Follows naming conventions
   - No code duplication
   - Proper error handling
   - Performance considerations

2. **Testing**
   - Unit tests included
   - Integration tests where appropriate
   - All tests passing
   - Code coverage maintained (>80%)

3. **Security**
   - No vulnerabilities introduced
   - Input validation present
   - No secrets committed
   - Security headers configured

4. **Documentation**
   - Code comments where needed
   - README updated if needed
   - API documentation current

## Testing Requirements

### Unit Tests
- Minimum 80% code coverage
- Follow AAA pattern (Arrange-Act-Assert)
- Use descriptive test names
- Mock external dependencies

Example:
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
Test API endpoints and database interactions:
```csharp
[Fact]
public async Task CreateUser_WithValidData_ReturnsCreated()
{
    var request = new CreateUserRequest("John", "Doe", "john@example.com");
    var response = await _client.PostAsJsonAsync("/api/users", request);
    
    response.StatusCode.Should().Be(HttpStatusCode.Created);
}
```

## Documentation Standards

- Use XML comments for public APIs
- Include examples in documentation
- Keep documentation up-to-date
- Document complex algorithms

Example:
```csharp
/// <summary>
/// Retrieves a user by their unique identifier.
/// </summary>
/// <param name="userId">The unique identifier of the user.</param>
/// <returns>The user if found; otherwise, null.</returns>
/// <exception cref="ArgumentException">Thrown when userId is less than 1.</exception>
public async Task<User?> GetUserAsync(int userId)
{
    // Implementation
}
```

## Security Considerations

### What to Check
- [ ] No hardcoded secrets
- [ ] Input validation implemented
- [ ] SQL injection prevention
- [ ] XSS protection
- [ ] CSRF tokens where needed
- [ ] Proper authentication/authorization
- [ ] Secure password handling
- [ ] HTTPS enforced

### What NOT to Commit
- ❌ API keys or secrets
- ❌ Connection strings
- ❌ Passwords
- ❌ Private keys
- ❌ Environment-specific configs

Use environment variables or secret managers instead.

## Performance Guidelines

- Use async/await properly
- Implement caching where appropriate
- Avoid N+1 query problems
- Use pagination for large datasets
- Optimize database queries
- Profile and benchmark critical paths

## Common Issues

### Build Failures
```bash
# Clean and rebuild
dotnet clean
dotnet restore
dotnet build
```

### Test Failures
```bash
# Run specific test
dotnet test --filter "FullyQualifiedName~YourTestName"

# Run with verbose output
dotnet test --logger "console;verbosity=detailed"
```

### Merge Conflicts
```bash
# Update your branch with latest changes
git checkout develop
git pull origin develop
git checkout feature/your-feature
git merge develop
# Resolve conflicts
git add .
git commit
```

## Getting Help

- Check [Best Practices](.github/specs/best-practices.md)
- Review [Architecture Guidelines](.github/specs/specifications/architecture.md)
- Look at existing code for examples
- Ask questions in pull request comments
- Open an issue for clarification

## Quality Checklist

Before submitting your PR, ensure:

- [ ] Code follows all style guidelines
- [ ] All tests pass locally
- [ ] Code coverage is maintained
- [ ] No linting errors
- [ ] Documentation is updated
- [ ] Commit messages follow convention
- [ ] PR description is complete
- [ ] No merge conflicts
- [ ] Self-review completed

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Thank You!

Your contributions make this template better for everyone! 🎉
