<!--
  Sync Impact Report:
  Version Change: Initial → 1.0.0
  Modified Principles: N/A (initial constitution)
  Added Sections:
    - All core principles (Code Quality, Test-First, UX Consistency, Performance)
    - Azure & Microservices Architecture Standards
    - Development Workflow & Quality Gates
  Removed Sections: N/A (initial constitution)
  Templates Requiring Updates:
    ✅ plan-template.md - Constitution Check section references this file
    ✅ tasks-template.md - Task categorization aligns with principles
    ✅ spec-template.md - User stories support independent testing
  Follow-up TODOs: None
-->

# .NET Microservices Web Application Constitution

## Core Principles

### I. Code Quality & Standards (NON-NEGOTIABLE)

Every code contribution MUST meet these quality gates:

- **Static Analysis**: Code MUST pass all .NET analyzers (StyleCop, Roslyn analyzers) with zero warnings
- **Code Coverage**: New code MUST achieve minimum 80% branch coverage; critical paths require 100%
- **SOLID Principles**: All services MUST follow SOLID design principles; violations require architectural review
- **Dependency Injection**: All dependencies MUST be injected via constructor; service locator pattern is prohibited
- **Nullability**: Nullable reference types MUST be enabled; all null scenarios MUST be explicitly handled
- **Async/Await**: I/O-bound operations MUST use async/await; blocking calls (.Result, .Wait()) are prohibited

**Rationale**: High code quality prevents defects, improves maintainability, and ensures consistent 
standards across distributed microservices where debugging is inherently more complex.

### II. Test-First Development (NON-NEGOTIABLE)

TDD is mandatory for all feature development following strict Red-Green-Refactor cycle:

- **Test Creation**: Acceptance tests MUST be written and approved by stakeholders BEFORE implementation
- **Red Phase**: Tests MUST fail initially (verify test validity)
- **Green Phase**: Implement minimal code to pass tests
- **Refactor Phase**: Optimize while maintaining green state
- **Test Types Required**:
  - **Unit Tests**: xUnit, minimum 80% coverage, isolated with mocks (Moq/NSubstitute)
  - **Integration Tests**: Test contracts between microservices, database interactions, Azure service integrations
  - **Contract Tests**: Verify API contracts remain stable across service versions (Pact or custom)
  - **Performance Tests**: Load/stress tests for critical endpoints (k6 or Azure Load Testing)

**Rationale**: Test-first development ensures features are testable by design, catches regressions early, 
and provides living documentation. Critical for microservices where service interactions create complexity.

### III. User Experience Consistency

UX MUST be consistent across all microservices and client touchpoints:

- **API Contracts**: All REST APIs MUST follow OpenAPI 3.0 specification with complete documentation
- **Response Format**: Standard JSON envelope: `{ "data": {}, "meta": {}, "errors": [] }` for all responses
- **Error Handling**: Consistent error codes and messages across services using RFC 7807 Problem Details
- **Versioning**: API versioning via URI path (e.g., `/api/v1/resource`); minimum 12-month deprecation notice
- **HTTP Standards**: Proper HTTP verbs (GET, POST, PUT, PATCH, DELETE) and status codes (2xx, 4xx, 5xx)
- **Pagination**: Standard cursor-based pagination for collections with `limit`, `cursor`, `total` metadata
- **Idempotency**: POST/PUT/PATCH operations MUST support idempotency keys for safe retries
- **Rate Limiting**: All public APIs MUST implement rate limiting with clear headers (`X-RateLimit-*`)

**Rationale**: Consistent UX across microservices reduces client integration complexity, improves 
developer experience, and enables API gateway patterns. Predictable behavior builds user trust.

### IV. Performance Requirements

Performance targets aligned with Azure-hosted microservices architecture:

- **Response Time**: P95 latency < 200ms for synchronous API calls; P99 < 500ms
- **Throughput**: Minimum 1,000 requests/second per service instance under normal load
- **Database Queries**: Individual queries MUST complete < 50ms; N+1 query patterns prohibited
- **Memory**: Service memory footprint < 512MB under normal load; < 1GB under peak load
- **Startup Time**: Service MUST be ready for traffic within 30 seconds of container start (health probe)
- **Resilience**: Circuit breakers required for all external dependencies (Polly library)
- **Caching**: Implement distributed caching (Azure Redis) for frequently accessed data (> 100 reads/minute)
- **Monitoring**: Application Insights instrumentation MUST track all requests, dependencies, exceptions

**Rationale**: Performance standards ensure cost-effective Azure hosting, enable horizontal scaling, 
and maintain SLA commitments. Microservices multiply latency; strict limits prevent cascade delays.

## Azure & Microservices Architecture Standards

### Technology Stack

- **.NET Version**: .NET 10 (LTS) with latest C# language version
- **Web Framework**: ASP.NET Core Minimal APIs or Web API with controller-based routing
- **Container Runtime**: Docker with Alpine Linux base images for minimal footprint
- **Orchestration**: Azure Kubernetes Service (AKS) or Azure Container Apps
- **API Gateway**: Azure API Management or YARP (reverse proxy)
- **Service Communication**: 
  - Synchronous: HTTP/REST with Refit or HttpClientFactory
  - Asynchronous: Azure Service Bus or Azure Event Grid for event-driven workflows
- **Data Storage**: 
  - Relational: Azure SQL Database or PostgreSQL (Entity Framework Core)
  - NoSQL: Azure Cosmos DB for global distribution requirements
  - Cache: Azure Redis Cache
- **Identity**: Azure AD B2C or Azure AD with JWT bearer tokens (Microsoft.Identity.Web)
- **Secrets Management**: Azure Key Vault; no secrets in configuration files or environment variables
- **CI/CD**: GitHub Actions with Azure deployment integration

### Microservices Design Constraints

- **Service Boundaries**: Each microservice MUST own its data; no shared databases between services
- **Database per Service**: Enforce logical or physical database separation
- **Event Sourcing**: Consider for audit-critical domains; use Azure Event Hubs for event streams
- **Saga Pattern**: Distributed transactions MUST use saga (orchestration or choreography)
- **Service Mesh**: Implement service-to-service authentication and encryption (Linkerd or Azure Service Mesh)
- **Backward Compatibility**: Services MUST support N-1 version compatibility during rolling deployments
- **Health Checks**: Implement `/health/live` (liveness) and `/health/ready` (readiness) endpoints
- **Graceful Shutdown**: Handle SIGTERM signals with connection draining (< 30 seconds)

### Security Requirements

- **Authentication**: All services MUST authenticate using Azure AD JWT tokens
- **Authorization**: Implement role-based access control (RBAC) with Azure AD roles
- **TLS**: All service-to-service communication MUST use TLS 1.3
- **Secrets Rotation**: Automatic secret rotation via Azure Key Vault; secrets refresh < 5 minutes
- **Dependency Scanning**: All NuGet packages MUST pass vulnerability scans (Dependabot, Snyk)
- **OWASP Top 10**: All services MUST address OWASP API Security Top 10 vulnerabilities

## Development Workflow & Quality Gates

### Code Review Requirements

- **Peer Review**: All PRs MUST receive approval from minimum 2 reviewers
- **Constitution Compliance**: Reviewers MUST verify adherence to all constitutional principles
- **Architecture Review**: Changes affecting service contracts require architecture team approval
- **Performance Review**: Changes to critical paths require performance benchmark comparison
- **Security Review**: Authentication/authorization changes require security team approval

### Quality Gates (CI Pipeline)

1. **Build**: Solution MUST compile with zero errors and zero warnings
2. **Unit Tests**: All unit tests MUST pass with ≥ 80% branch coverage
3. **Integration Tests**: All integration tests MUST pass
4. **Static Analysis**: SonarQube quality gate MUST pass (A rating on reliability, security, maintainability)
5. **Dependency Check**: No high/critical severity vulnerabilities
6. **Container Build**: Docker image MUST build successfully
7. **Performance Baseline**: Critical endpoints MUST not regress > 10% from baseline

### Deployment Gates (CD Pipeline)

1. **Infrastructure as Code**: All Azure resources defined in Bicep or Terraform
2. **Blue-Green Deployment**: Zero-downtime deployments with automated rollback
3. **Smoke Tests**: Post-deployment health checks MUST pass
4. **Monitoring**: Application Insights alerts configured for new services
5. **Chaos Engineering**: Production services MUST pass quarterly chaos experiments

## Governance

This constitution supersedes all other development practices and policies. All team members, pull requests, 
and design reviews MUST verify compliance with these principles.

**Amendment Process**:
1. Proposed changes MUST be documented with rationale and impact analysis
2. Architecture team MUST approve all amendments
3. Major version bump for backward-incompatible governance changes
4. All affected templates and documentation MUST be updated before amendment takes effect
5. Team communication and training MUST occur within 2 weeks of amendment

**Compliance Review**:
- Weekly: Automated compliance checks via CI/CD pipelines
- Monthly: Architecture review of constitution adherence across services
- Quarterly: Constitution effectiveness review and amendment consideration

**Complexity Justification**: Any violation of constitutional principles MUST include written justification 
approved by architecture team and documented in the feature's implementation plan.

**Runtime Guidance**: Use `.github/prompts/speckit.*.prompt.md` files for agent-specific development workflows.

**Version**: 1.0.0 | **Ratified**: 2025-10-11 | **Last Amended**: 2025-10-11