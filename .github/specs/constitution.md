# Constitution

## Core Principles

This template establishes the foundational principles for building robust, maintainable, and scalable .NET web applications.

### 1. Code Quality First
- **Maintainability**: Write code that is easy to understand, modify, and extend
- **Readability**: Prioritize clear, self-documenting code over clever solutions
- **Testability**: Design with testing in mind from the start
- **Simplicity**: Follow KISS (Keep It Simple, Stupid) principle

### 2. Performance & Scalability
- Design for horizontal scalability
- Optimize database queries and API calls
- Implement caching strategies appropriately
- Monitor and measure performance metrics

### 3. Security by Design
- Never trust user input - validate and sanitize everything
- Implement proper authentication and authorization
- Follow OWASP Top 10 security guidelines
- Keep dependencies up to date
- Use secrets management (never commit secrets)

### 4. Modern Development Practices
- Follow Domain-Driven Design (DDD) principles
- Apply SOLID principles consistently
- Use dependency injection throughout
- Implement proper error handling and logging
- Version all APIs

### 5. Team Collaboration
- Write comprehensive documentation
- Conduct thorough code reviews
- Follow consistent naming conventions
- Maintain backward compatibility when possible
- Communicate changes effectively

### 6. Continuous Improvement
- Refactor continuously
- Update dependencies regularly
- Monitor and act on metrics
- Learn from production issues
- Stay current with .NET ecosystem

## Technology Stack Preferences

### Backend
- **.NET 10**: Latest LTS version
- **ASP.NET Core**: Web framework
- **Entity Framework Core**: ORM
- **MediatR**: CQRS pattern implementation
- **FluentValidation**: Input validation

### Frontend
- **React** or **Vue.js**: UI framework
- **FluentUI (Microsoft)**: Component library
- **TypeScript**: Type-safe JavaScript

### Database
- **PostgreSQL** or **SQL Server**: Primary database
- **Redis**: Caching and session storage

### DevOps
- **Docker**: Containerization
- **GitHub Actions**: CI/CD
- **Azure** or **AWS**: Cloud hosting

## Project Structure Philosophy

Every project should follow a clean architecture pattern:

1. **Presentation Layer**: Controllers, Views, API endpoints
2. **Application Layer**: Business logic, use cases, DTOs
3. **Domain Layer**: Entities, value objects, domain events
4. **Infrastructure Layer**: Data access, external services, file system

## Quality Gates

All code must pass these gates before merging:

1. ✅ All unit tests pass
2. ✅ Code coverage > 80%
3. ✅ No critical security vulnerabilities
4. ✅ Linting and formatting checks pass
5. ✅ Peer review approval
6. ✅ Integration tests pass
7. ✅ Performance benchmarks met

## Change Management

- All changes must go through pull requests
- Breaking changes require major version bump
- Deprecations must be communicated 2 versions in advance
- Migration guides must be provided for breaking changes
