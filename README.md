# spec-kit-for-dotnet-template

A specification-driven template for building .NET 10 microservices web applications on Azure, following best practices for code quality, testing standards, user experience consistency, and performance.

## Overview

This template provides a comprehensive framework for developing Azure-hosted .NET microservices applications using the spec-kit methodology. It includes:

- **Constitutional Governance**: Core principles ensuring code quality, test-first development, UX consistency, and performance
- **Specification Templates**: Structured formats for features, plans, tasks, and checklists
- **Development Workflows**: Agent-driven commands for specification, planning, implementation, and analysis
- **Azure Best Practices**: Optimized for AKS/Container Apps deployment with integrated monitoring and security

## Key Features

- ✅ .NET 10 with latest C# language features
- ✅ Microservices architecture with clean separation of concerns
- ✅ Test-Driven Development (TDD) mandatory workflow
- ✅ Azure-native integrations (Service Bus, Key Vault, Application Insights, Redis Cache)
- ✅ OpenAPI 3.0 compliant REST APIs with versioning
- ✅ Performance-focused (P95 < 200ms, 1000+ req/s per instance)
- ✅ Infrastructure as Code (Bicep/Terraform)
- ✅ CI/CD with GitHub Actions

## Getting Started

1. Review the [Constitution](.specify/memory/constitution.md) to understand core principles
2. Use `.github/prompts/speckit.*.prompt.md` commands to drive development workflows
3. Follow the specification templates in `.specify/templates/` for consistent documentation

## Architecture

This template supports microservices architecture with:
- Domain-Driven Design (DDD) layering
- Database per service pattern
- Event-driven communication via Azure Service Bus
- API Gateway (Azure API Management or YARP)
- Distributed caching and resilience patterns

## License

[Specify your license here]
