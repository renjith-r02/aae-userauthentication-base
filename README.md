# AuthService3

Enterprise-style tutorial repository for building a secure authentication microservice (Java 17, Maven) with JWT, refresh token rotation, RBAC, Redis-backed token invalidation, and governance-driven delivery.

## Tutorial Objective

Build a production-grade **Authentication Service** that supports:
- User registration
- User login and JWT issuance
- JWT validation
- Refresh token rotation with replay protection
- Logout/session invalidation
- Authenticated profile (`/me`) endpoint
- Role-based access control (RBAC)

Primary source requirements:  
- `.github/docs/requirements/authentication_service_requirements.md`

## Prerequisites

- macOS/Linux/Windows
- JDK 17+
- Maven 3.9+
- PostgreSQL 14+
- Redis 7+
- Docker Desktop (optional; for container workflow)
- Kubernetes CLI (`kubectl`) and local cluster (optional: minikube/kind)
- GitHub account and GitHub Actions access

## Local Setup

### Install and Configure Required Technologies

- Java (OpenJDK 17/21): https://adoptium.net/temurin/releases/
- PostgreSQL: https://www.postgresql.org/download/
- Redis: https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/
- Maven: https://maven.apache.org/install.html

### Optional Technologies

- Docker Desktop: https://docs.docker.com/desktop/
- Kubernetes `kubectl`: https://kubernetes.io/docs/tasks/tools/
- Minikube: https://minikube.sigs.k8s.io/docs/start/

```bash
# 1) Clone
git clone <your-repo-url>
cd AuthService3

# 2) Verify Java/Maven
java -version
mvn -version

# 3) Build current scaffold
mvn clean compile

# 4) Run current app
mvn exec:java -Dexec.mainClass="org.example.Main"
```

## Repository Setup (Recommended)

```bash
# Create baseline branches
git checkout -b develop
git checkout -b feature/auth-foundation

# Optional: enable commit signing and hooks (if your org enforces)
# git config commit.gpgsign true
```

Suggested next repository additions:
- `src/main/resources/application.yml` (externalized config)
- `src/test/...` (unit/integration/security tests)
- `Dockerfile`
- `docker-compose.yml` (app + redis + db)
- `.github/workflows/ci.yml`
- `k8s/` manifests or Helm chart

## Architecture, Security, Validation, and API Instruction Files

The repository includes enterprise guidance under `.github/instructions/`:

- `architecture.instructions.md`
  - Stateless services, loose coupling, Kubernetes-ready, service boundaries
- `security.instructions.md`
  - Zero Trust, least privilege, OWASP Top 10 controls, container/K8s security
- `validation.instructions.md`
  - Quality gates: SonarQube, SAST, dependency and container scanning, test coverage
- `api.instructions.md`
  - OpenAPI 3.x, versioned APIs, consistent error response, JWT + RBAC + rate limiting

Copilot governance baseline:
- `.github/copilot-instructions.md`

## Functional Requirements (Summary)

From `AUTH-FR-001` to `AUTH-FR-007`:

1. **Register User** (`POST /api/v1/auth/register`)
   - Validate profile + strong password policy
   - Hash password with bcrypt
2. **Login + Token Issuance** (`POST /api/v1/auth/login`)
   - Validate credentials, enforce rate limiting
   - Issue short-lived access token + long-lived refresh token
3. **Access Token Validation** (`POST /api/v1/auth/token/validate`)
   - Validate signature, expiry, issuer/audience, blacklist status
4. **Refresh Token Flow** (`POST /api/v1/auth/token/refresh`)
   - Rotate refresh token each use; detect replay
5. **Session Invalidation / Logout** (`POST /api/v1/auth/logout`)
   - Blacklist access token ID in Redis; revoke refresh token/session
6. **Current User Profile** (`GET /api/v1/auth/me`)
   - Return non-sensitive user details and authorization context
7. **RBAC Enforcement**
   - Roles: `USER`, `ADMIN`, `SERVICE`; deny-by-default on protected operations

## Non-Functional Requirements (Summary)

- **Security:** TLS, bcrypt/Argon2, signed JWTs, key rotation, no secret hardcoding
- **Performance Targets:** login < 300ms, token validation < 50ms, refresh < 200ms
- **Scalability:** stateless validation, horizontal scale, Redis for shared state
- **Reliability:** fail secure, revoke on suspicious token behavior
- **Observability:** structured logs, audit trails, trace IDs, auth/rate-limit metrics

## Requirement Traceability Map (Starter)

| Requirement ID | API/Component | Tests | Governance Gate |
|---|---|---|---|
| AUTH-FR-001 | Register API + User Service | unit + integration | SAST + quality gate |
| AUTH-FR-002 | Login API + Token Service | unit + integration + rate-limit tests | dependency scan |
| AUTH-FR-003 | Token Validation Filter/API | unit + security tests | SAST |
| AUTH-FR-004 | Refresh Flow + Token Store | integration + replay tests | security validation |
| AUTH-FR-005 | Logout + Redis Blacklist | integration tests | container + dependency scans |
| AUTH-FR-006 | `/me` endpoint + RBAC | integration + authorization tests | API standards validation |
| AUTH-FR-007 | Authorization layer | unit + security tests | governance compliance |

## Dependency Analysis (Initial)

Current `pom.xml` only defines Java 17 compiler settings. Add dependencies incrementally:
- Spring Boot Web
- Spring Security
- Spring Validation
- Spring Data JPA
- Redis client
- JWT library
- Test stack (JUnit, Mockito, Testcontainers)

## Risk Analysis (Initial)

| Risk | Impact | Mitigation |
|---|---|---|
| Weak password policy enforcement | Account compromise | strict validators + security tests |
| Token replay on refresh flow | Session takeover | rotation + token family tracking + replay detection |
| Inadequate rate limiting | Brute-force attacks | per-IP + per-account limits in Redis |
| Secret leakage in code/logs | Credential exposure | env/secret manager + log masking |
| RBAC gaps | Privilege escalation | deny-by-default + authorization tests |

## Copilot Prompting Workflow

1. **Requirements-first prompt**
   - “Implement AUTH-FR-001 registration endpoint with validation and bcrypt hashing.”
2. **Architecture alignment prompt**
   - “Follow `.github/instructions/architecture.instructions.md`; keep service stateless.”
3. **Security prompt**
   - “Apply `.github/instructions/security.instructions.md`; no hardcoded secrets.”
4. **Validation prompt**
   - “Generate unit/integration tests and map each to requirement IDs.”
5. **Governance prompt**
   - “Prepare CI checks for SAST, dependency scan, and coverage threshold.”

Use small, requirement-ID-based prompts and review generated code against instruction files.

## Implementation Plan

### Phase 1: Foundation
- Create Spring Boot app structure
- Configure environment profiles
- Add health/readiness/liveness endpoints

### Phase 2: Core Auth APIs
- Implement register/login/token validate/refresh/logout/me
- Introduce JWT utilities, Redis blacklist, refresh token store

### Phase 3: Security Hardening
- Rate limiting, account lock strategy
- Audit logging and sensitive data masking
- RBAC policy enforcement

### Phase 4: Observability and Governance
- Structured logging, trace IDs, metrics
- OpenAPI documentation
- CI quality gates

### Phase 5: Production Packaging
- Docker image
- Kubernetes manifests
- Deployment runbook

## Testing Plan

### Unit Tests
- Validation logic
- Password hashing and comparison
- JWT generation/verification
- RBAC decision checks

### Integration Tests
- End-to-end API flows for all auth endpoints
- Redis blacklist behavior
- DB persistence and token status transitions

### Security Tests
- Invalid/tampered/expired token rejection
- Refresh token replay detection
- Brute-force/rate-limiting scenarios
- Sensitive fields excluded from responses/logs

Run baseline test command:

```bash
mvn test
```

## Docker Setup (Planned)

```bash
# Build app jar
mvn clean package

# Build image
docker build -t authservice3:local .

# Run container (example)
docker run --rm -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=local \
  authservice3:local
```

## Kubernetes Setup (Planned)

```bash
# Apply manifests (after creating k8s/ files)
kubectl apply -f k8s/

# Check rollout
kubectl get pods
kubectl get svc
```

Minimum K8s expectations:
- Deployment + Service + ConfigMap/Secret
- Liveness/readiness probes
- Resource requests/limits
- NetworkPolicy and non-root container policy

## CI/CD Setup (Planned)

Typical pipeline stages:
1. Build + unit tests
2. Static analysis (SAST + lint + SonarQube)
3. Dependency and container scan
4. Package and publish artifact/image
5. Deploy to dev/stage with approvals

Example local build command used by CI:

```bash
mvn -B clean verify
```

## Live Tutorial Flow

1. Start from requirements file and define API contracts.
2. Scaffold service and dependency stack.
3. Implement registration and login.
4. Add JWT validation and protected `/me` endpoint.
5. Add refresh token rotation and replay handling.
6. Add logout + Redis blacklist.
7. Add RBAC and audit logging.
8. Finalize tests, scans, and deployment assets.

## Run Commands (Quick Reference)

```bash
# Build
mvn clean compile

# Test
mvn test

# Package
mvn clean package

# Run current entrypoint
mvn exec:java -Dexec.mainClass="org.example.Main"
```

## Expected Outcomes

By completion of this tutorial implementation, the repository should provide:
- Versioned and documented auth APIs
- Secure credential and token lifecycle management
- Redis-backed blacklist and rate limiting
- RBAC-protected endpoints
- Passing unit/integration/security tests
- Containerized and Kubernetes-deployable service
- CI/CD with enterprise quality and security gates

## Enterprise `.github` Governance and Agents

### Governance Documents
- `.github/copilot-instructions.md`
- `.github/instructions/architecture.instructions.md`
- `.github/instructions/security.instructions.md`
- `.github/instructions/validation.instructions.md`
- `.github/instructions/api.instructions.md`

### Agent Documents
- `.github/agents/requirements-analysis.agent.md`
  - functional/non-functional extraction, security requirements, traceability
- `.github/agents/architecture-planning.agent.md`
  - service boundaries, topology, resilience, architecture outputs
- `.github/agents/repository-analysis.agent.md`
  - structure/anti-pattern/dependency/governance review
- `.github/agents/validation-governance.agent.md`
  - architecture/security/API/CI compliance validation
- `.github/agents/workspace-persistence.agent.md`
  - design continuity and drift tracking
- `.github/agents/diagram-generation.agent.md`
  - component/deployment/sequence/data-flow diagrams with trust boundaries

### Artifact-Generation Prompts

The following prompt is documented for reference only and should not be executed directly from the README.

1. **Architecture artifact generation**

   ```text
   #file:diagram-generation.agent.md #file:architecture-planning.agent.md Using the requirements defined in #file:authentication_service_requirements.md create component diagram, sequence diagram, class diagram, deployment diagram in mermaid format and create md files in .github/docs/architecture folder . Create api contracts in swagger format inside the same folder. Follow the naming conventions as defined in #file:architecture-planning.agent.md
   ```

2. **Spring Boot implementation and test artifact generation**

   ```text
   #file:architecture-planning.agent.md Refer the project requirements in the #file:authentication_service_requirements.md architecture and coding standards in #file:architecture-planning.agent.md Refer the following architecture diagram #file:class-diagram.md #file:sequence-diagram.md #file:deployment-diagram.md to create a Spring Boot application with Unit , Integration and Jmeter testing scripts included for deployment to Kubernetes cluster. Application uses PostGres as DB, Redis for caching
   ```

## Current State Note

The repository currently contains a minimal Java scaffold (`org.example.Main`) and instruction/requirements documentation. The service implementation described above is the target outcome of the tutorial-driven build.
