# AuthService3

Enterprise-style tutorial repository for building a secure authentication microservice (Java 17, Maven) with JWT, refresh token rotation, RBAC, Redis-backed token invalidation, and governance-driven delivery.

## Software Installation and Tooling (Start Here)

### Required
- **Java (OpenJDK 17/21):** https://adoptium.net/temurin/releases/
- **Maven 3.9+:** https://maven.apache.org/install.html
- **PostgreSQL 14+:** https://www.postgresql.org/download/
- **Redis 7+:** https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/
- **Git:** https://git-scm.com/downloads
- **GitHub account:** https://github.com/

### Optional (for container/orchestration workflows)
- **Docker Desktop:** https://docs.docker.com/desktop/
- **Kubernetes CLI (`kubectl`):** https://kubernetes.io/docs/tasks/tools/
- **Minikube:** https://minikube.sigs.k8s.io/docs/start/

## Clone and Initial Setup

Base repository:
`https://github.com/renjith-r02/aae-userauthentication-base`

```bash
git clone https://github.com/renjith-r02/aae-userauthentication-base.git
cd aae-userauthentication-base
```

Recommended local verification:

```bash
java -version
mvn -version
mvn clean compile
mvn test
```

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

## Enterprise Governance Context (Concise)

### Core instruction files
- `.github/copilot-instructions.md`
- `.github/instructions/architecture.instructions.md`
- `.github/instructions/security.instructions.md`
- `.github/instructions/validation.instructions.md`
- `.github/instructions/api.instructions.md`

### Agent files
- `.github/agents/requirements-analysis.agent.md`
- `.github/agents/architecture-planning.agent.md`
- `.github/agents/repository-analysis.agent.md`
- `.github/agents/validation-governance.agent.md`
- `.github/agents/workspace-persistence.agent.md`
- `.github/agents/diagram-generation.agent.md`

## Functional Requirements (Summary)

From `AUTH-FR-001` to `AUTH-FR-007`:
1. **Register User** (`POST /api/v1/auth/register`) with profile/password validation and bcrypt hashing
2. **Login + Token Issuance** (`POST /api/v1/auth/login`) with rate limiting
3. **Access Token Validation** (`POST /api/v1/auth/token/validate`) including signature/expiry/issuer/audience/blacklist checks
4. **Refresh Token Flow** (`POST /api/v1/auth/token/refresh`) with rotation and replay detection
5. **Session Invalidation / Logout** (`POST /api/v1/auth/logout`) with Redis blacklist + refresh revocation
6. **Current User Profile** (`GET /api/v1/auth/me`) with non-sensitive user context
7. **RBAC Enforcement** for `USER`, `ADMIN`, `SERVICE` (deny-by-default)

## Non-Functional and Security Requirements (Summary)

- **Security:** TLS, bcrypt/Argon2, signed JWTs, key rotation, no hardcoded secrets
- **Performance:** login < 300ms, token validation < 50ms, refresh < 200ms
- **Scalability:** stateless service design + Redis shared token state
- **Reliability:** fail secure and revoke suspicious sessions/tokens
- **Observability:** structured logging, audit trails, trace IDs, auth/rate-limit metrics

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

Current `pom.xml` is minimal. Add dependencies incrementally:
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

## Implementation Plan (Concise)

1. **Foundation:** Spring Boot structure, profiles, health endpoints
2. **Core Auth APIs:** register/login/validate/refresh/logout/me + JWT + Redis
3. **Security Hardening:** rate limits, lock strategy, audit masking, RBAC
4. **Observability & Governance:** logs, traces, metrics, OpenAPI, CI gates
5. **Packaging:** Docker image and Kubernetes manifests

## Testing Plan (Concise)

- **Unit:** validation, hashing, JWT logic, RBAC decisions
- **Integration:** endpoint flows, Redis blacklist, DB state transitions
- **Security:** tampered token rejection, replay detection, brute-force scenarios

## Optional Docker Setup (Planned)

```bash
mvn clean package
docker build -t authservice3:local .
docker run --rm -p 8080:8080 -e SPRING_PROFILES_ACTIVE=local authservice3:local
```

## Optional Kubernetes Setup (Planned)

```bash
kubectl apply -f k8s/
kubectl get pods
kubectl get svc
```

Minimum K8s expectations:
- Deployment + Service + ConfigMap/Secret
- Liveness/readiness probes
- Resource requests/limits
- NetworkPolicy and non-root container policy

## Prompt Library (Documentation Only)

The following prompts are reference content for artifact generation.

### 1) Architecture artifact generation prompt

```text
#file:diagram-generation.agent.md #file:architecture-planning.agent.md Using the requirements defined in #file:authentication_service_requirements.md create component diagram, sequence diagram, class diagram, deployment diagram in mermaid format and create md files in .github/docs/architecture folder . Create api contracts in swagger format inside the same folder. Follow the naming conventions as defined in #file:architecture-planning.agent.md
```

### 2) Spring Boot implementation and test artifact generation prompt

```text
#file:architecture-planning.agent.md Refer the project requirements in the #file:authentication_service_requirements.md architecture and coding standards in #file:architecture-planning.agent.md Refer the following architecture diagram #file:class-diagram.md #file:sequence-diagram.md #file:deployment-diagram.md to create a Spring Boot application with Unit , Integration and Jmeter testing scripts included for deployment to Kubernetes cluster. Application uses PostGres as DB, Redis for caching
```

## Current State Note

The repository currently contains a minimal Java scaffold (`org.example.Main`) and documentation/guidance assets. The service implementation above is the target outcome.
