# .NET Web Application Template with Spec-Kit

A comprehensive template for .NET 10 web applications using GitHub's spec-kit framework with best practices, coding standards, and architectural guidelines.

## 📋 Overview

This repository serves as a foundational template for building modern .NET web applications. It includes comprehensive specifications, best practices, and guidelines following:

- **.NET 10** best practices
- **Clean Architecture** principles
- **Domain-Driven Design (DDD)**
- **Security-first** approach
- **Performance optimization** strategies
- **Comprehensive testing** standards

## 🏗️ Technology Stack

### Backend
- **.NET 10** - Latest LTS version
- **ASP.NET Core** - Web framework
- **Entity Framework Core** - ORM
- **MediatR** - CQRS pattern
- **FluentValidation** - Input validation

### Frontend
- **React** or **Vue.js** - UI framework (your choice)
- **FluentUI (Microsoft)** - Component library
- **TypeScript** - Type-safe JavaScript

### Database
- **PostgreSQL** or **SQL Server** - Primary database
- **Redis** - Caching and session storage

### DevOps
- **Docker** - Containerization
- **GitHub Actions** - CI/CD
- **Azure** or **AWS** - Cloud hosting

## 📁 Repository Structure

```
.github/
└── specs/                          # Spec-kit specifications
    ├── constitution.md             # Core principles and guidelines
    ├── best-practices.md           # .NET 10 coding standards
    ├── rules.md                    # Development rules and conventions
    ├── naming-conventions.md       # Standardized naming conventions
    └── specifications/
        ├── architecture.md         # Application architecture guidelines
        ├── security.md             # Security best practices
        ├── performance.md          # Performance optimization
        ├── testing.md              # Testing standards
        └── ui-guidelines.md        # React/Vue + FluentUI guidelines
```

## 🚀 Getting Started

### Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
- [Node.js 20+](https://nodejs.org/) (for frontend)
- [Docker](https://www.docker.com/) (optional)
- [PostgreSQL](https://www.postgresql.org/) or SQL Server

### Using This Template

1. **Click "Use this template"** button on GitHub
2. **Clone your new repository**
   ```bash
   git clone https://github.com/yourusername/your-project-name.git
   cd your-project-name
   ```

3. **Review the specifications**
   - Read [Constitution](.github/specs/constitution.md) for core principles
   - Check [Architecture](.github/specs/specifications/architecture.md) for structure
   - Review [Best Practices](.github/specs/best-practices.md) for coding standards

4. **Create your .NET project structure**
   ```bash
   # Create solution
   dotnet new sln -n YourProjectName

   # Create projects following clean architecture
   dotnet new classlib -n YourProjectName.Domain
   dotnet new classlib -n YourProjectName.Application
   dotnet new classlib -n YourProjectName.Infrastructure
   dotnet new webapi -n YourProjectName.Api

   # Add projects to solution
   dotnet sln add **/*.csproj
   ```

5. **Set up frontend** (React or Vue.js)
   ```bash
   # React with TypeScript
   npx create-react-app client --template typescript
   cd client
   npm install @fluentui/react @fluentui/react-icons

   # OR Vue.js with TypeScript
   npm create vue@latest
   cd client
   npm install @fluentui/web-components
   ```

## 📚 Documentation

### Core Specifications

- **[Constitution](.github/specs/constitution.md)** - Foundational principles and technology choices
- **[Best Practices](.github/specs/best-practices.md)** - .NET 10 coding standards and patterns
- **[Rules](.github/specs/rules.md)** - Development rules, Git workflow, and quality gates
- **[Naming Conventions](.github/specs/naming-conventions.md)** - Comprehensive naming standards

### Technical Specifications

- **[Architecture](.github/specs/specifications/architecture.md)** - Clean architecture, CQRS, DDD patterns
- **[Security](.github/specs/specifications/security.md)** - Authentication, authorization, data protection
- **[Performance](.github/specs/specifications/performance.md)** - Optimization strategies and caching
- **[Testing](.github/specs/specifications/testing.md)** - Unit, integration, and E2E testing standards
- **[UI Guidelines](.github/specs/specifications/ui-guidelines.md)** - React/Vue.js with FluentUI implementation

## 🏛️ Architecture

This template follows **Clean Architecture** with **Domain-Driven Design**:

```
src/
├── Domain/           # Business entities, value objects, domain events
├── Application/      # Use cases, DTOs, CQRS handlers
├── Infrastructure/   # Data access, external services
└── API/              # Controllers, middleware, API endpoints

tests/
├── UnitTests/        # Unit tests (70% of tests)
├── IntegrationTests/ # Integration tests (20% of tests)
└── E2ETests/         # End-to-end tests (10% of tests)
```

## 🔒 Security

Security is built-in from the start:

- ✅ JWT authentication with refresh tokens
- ✅ Role-based and policy-based authorization
- ✅ Input validation and sanitization
- ✅ SQL injection prevention
- ✅ XSS protection
- ✅ HTTPS enforcement
- ✅ Security headers
- ✅ Rate limiting
- ✅ Secrets management

See [Security Specifications](.github/specs/specifications/security.md) for details.

## 🚦 Quality Gates

All code must pass these gates before merging:

- ✅ All unit tests pass
- ✅ Code coverage > 80%
- ✅ No critical security vulnerabilities
- ✅ Linting and formatting checks pass
- ✅ Peer review approval
- ✅ Integration tests pass
- ✅ Performance benchmarks met

## 🧪 Testing Strategy

Following the **Test Pyramid**:

- **70% Unit Tests** - Fast, isolated, business logic
- **20% Integration Tests** - API and database integration
- **10% E2E Tests** - Critical user journeys

See [Testing Standards](.github/specs/specifications/testing.md) for details.

## 📊 Performance Targets

- API endpoints: < 200ms (p95)
- Database queries: < 100ms (p95)
- Page load time: < 2s (p95)
- Concurrent users: 10,000+

See [Performance Specifications](.github/specs/specifications/performance.md) for optimization strategies.

## 🎨 UI Development

### FluentUI Integration

**React:**
```typescript
import { ThemeProvider } from '@fluentui/react';
import { PrimaryButton } from '@fluentui/react/lib/Button';
```

**Vue.js:**
```vue
<template>
  <fluent-button appearance="primary">Click me</fluent-button>
</template>
```

See [UI Guidelines](.github/specs/specifications/ui-guidelines.md) for complete setup.

## 🔄 Development Workflow

1. **Create feature branch** from `develop`
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Follow naming conventions** from [Naming Conventions](.github/specs/naming-conventions.md)

3. **Write tests first** (TDD approach)

4. **Implement feature** following [Best Practices](.github/specs/best-practices.md)

5. **Run quality checks**
   ```bash
   dotnet build
   dotnet test
   dotnet format
   ```

6. **Create Pull Request** with clear description

7. **Pass code review** and quality gates

8. **Merge to develop** → then to main for release

## 📝 Code Style

This template uses `.editorconfig` for consistent formatting:

- **C# files**: 4 spaces, PascalCase for public members
- **TypeScript/JavaScript**: 2 spaces, camelCase for variables
- **JSON/YAML**: 2 spaces
- **Line endings**: LF (Unix-style)

## 🤝 Contributing

1. Read the [Constitution](.github/specs/constitution.md)
2. Follow the [Rules](.github/specs/rules.md)
3. Adhere to [Best Practices](.github/specs/best-practices.md)
4. Use [Naming Conventions](.github/specs/naming-conventions.md)
5. Write comprehensive tests per [Testing Standards](.github/specs/specifications/testing.md)

## 📄 License

This template is available under the MIT License. See LICENSE file for details.

## 🔗 Resources

- [.NET Documentation](https://docs.microsoft.com/dotnet/)
- [ASP.NET Core Documentation](https://docs.microsoft.com/aspnet/core/)
- [FluentUI React](https://developer.microsoft.com/fluentui)
- [React Documentation](https://react.dev/)
- [Vue.js Documentation](https://vuejs.org/)

## 💡 Quick Tips

- **Always use async/await** for I/O operations
- **Enable nullable reference types** in all projects
- **Use file-scoped namespaces** (C# 10+)
- **Implement proper logging** with structured logging
- **Cache appropriately** with memory and distributed caching
- **Validate all inputs** using FluentValidation
- **Never commit secrets** - use secure secret management

## 🎯 Next Steps

1. Review all specifications in `.github/specs/`
2. Set up your project following the architecture guidelines
3. Configure CI/CD pipeline using GitHub Actions
4. Implement authentication and authorization
5. Set up database with migrations
6. Create your first feature following TDD
7. Deploy to your preferred cloud platform

---

**Happy Coding!** 🚀

For questions or issues, please refer to the specifications or open an issue in this repository.
