# Backend / API Production Checklist

---

## 🔴 CRITICAL

### Authentication & Authorization

- [ ] All endpoints require authentication except explicitly public ones (default-deny; whitelist public routes)
- [ ] Machine-to-machine auth uses OAuth 2.0 with mTLS or `private_key_jwt` — no long-lived static API keys shared over email/Slack
- [ ] JWT access tokens have short expiry ≤ 15 minutes with refresh token rotation (revoke refresh tokens on password change)
- [ ] Role-Based Access Control (RBAC) or Attribute-Based Access Control (ABAC) enforced — principle of least privilege on every endpoint
- [ ] No sensitive data stored in JWT payload — it's Base64-encoded, NOT encrypted (anyone can decode it)
- [ ] Admin/internal endpoints behind separate authentication layer, not just a role check on the same API
- [ ] Password hashing uses bcrypt, scrypt, or Argon2id with appropriate cost factor — never MD5/SHA1/SHA256 for passwords
- [ ] Brute-force protection on login: account lockout or exponential backoff after 5-10 failed attempts
- [ ] Session tokens are invalidated on logout — server-side session store or token blocklist for JWTs
- [ ] Multi-factor authentication (MFA) available for admin and privileged user accounts — enforced for internal tools
- [ ] OAuth/OIDC state parameter validated to prevent CSRF on authorization flows
- [ ] API key scoping: each key has minimum required permissions, IP allowlists where possible, and expiration dates
- [ ] Privilege escalation paths audited — no way for a user to modify their own role or access resources outside their tenant

### Input Validation & Injection Prevention

- [ ] All input validated and sanitized server-side — never trust client-side validation alone
- [ ] SQL injection prevented: use parameterized queries or ORM query builders, never string concatenation for SQL
- [ ] NoSQL injection prevented: validate query operators, reject `$` prefixed keys in MongoDB user input
- [ ] Request body size limits configured (e.g., `body-parser` limit, nginx `client_max_body_size`) to prevent memory exhaustion
- [ ] Content-Type validation on all POST/PUT/PATCH endpoints — reject unexpected content types
- [ ] File upload validation: restrict allowed MIME types, enforce max size, scan for malware, store outside webroot
- [ ] Path traversal prevented: never use user input directly in file system paths (`../../../etc/passwd`)
- [ ] GraphQL: depth limiting, query complexity analysis, and introspection disabled in production (if applicable)
- [ ] Command injection prevented — never pass user input to shell commands (`exec`, `system`, `child_process.exec`)
- [ ] XML External Entity (XXE) prevention: disable DTD processing and external entity resolution in all XML parsers
- [ ] Server-Side Request Forgery (SSRF) prevented: validate and allowlist URLs when the server makes requests based on user input
- [ ] Mass assignment / over-posting prevented: explicitly whitelist allowed fields on all create/update endpoints (never bind request body directly to model)
- [ ] Regular expression denial of service (ReDoS) prevented: audit all user-facing regex for catastrophic backtracking

### Secrets & Configuration

- [ ] No secrets in codebase, Dockerfiles, or container images — scan with `git-secrets`, `truffleHog`, or `gitleaks`
- [ ] Secrets managed via dedicated vault or secrets manager (AWS Secrets Manager, HashiCorp Vault, Doppler, 1Password Secrets Automation)
- [ ] Environment-specific configs strictly separated: dev/staging/prod use different databases, keys, and service URLs
- [ ] Database credentials rotated before going to production (don't launch with the same password used during development)
- [ ] Default credentials changed on all services: databases, admin panels, message queues, cache servers
- [ ] `.env` files excluded from Docker images and Git (verify `.gitignore` and `.dockerignore`)
- [ ] Secret rotation automated on a schedule — credentials and API keys rotate without manual intervention or downtime
- [ ] Secrets injected at runtime via environment or mounted volumes — never baked into container images at build time
- [ ] Git history scanned for accidentally committed secrets — rotate any found immediately (they persist in history even after deletion)

### Rate Limiting & DoS Protection

- [ ] Rate limiting on all public endpoints with stricter limits on auth routes: login, signup, OTP, password reset (e.g., 5/min for login)
- [ ] WAF (Web Application Firewall) configured in front of the API (Cloudflare, AWS WAF, Fastly) to block common attack patterns
- [ ] Request timeout configured on all routes to prevent slowloris and slow-read attacks (e.g., 30s max)
- [ ] Payload size limits enforced at the reverse proxy / load balancer level as an additional layer
- [ ] API abuse detection: monitor for credential stuffing patterns, automated scraping, enumeration attacks
- [ ] Per-tenant / per-user rate limits for multi-tenant APIs — prevent one tenant from exhausting shared resources
- [ ] Rate limit headers returned to clients: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`, `Retry-After`

### Data Integrity & Transactions

- [ ] All critical mutations wrapped in database transactions with appropriate isolation levels (prevent phantom reads, lost updates)
- [ ] Foreign key constraints and NOT NULL constraints enforced at the database level — never rely solely on application-level validation
- [ ] Idempotency keys required on all payment and financial mutation endpoints — retries must not cause duplicate side effects
- [ ] Unique constraints on business-critical fields (email, username, order ID) enforced at DB level, not just application code
- [ ] Soft delete implemented for user-facing data where recovery may be needed — hard delete only after retention period

---

## 🟡 IMPORTANT

### Observability & Monitoring

- [ ] Structured logging in JSON format with consistent fields: `timestamp`, `level`, `message`, `correlation_id`, `service` (use pino, winston, structlog, slog)
- [ ] Every request assigned a correlation/trace ID — propagated across service boundaries in headers (e.g., `X-Request-ID`)
- [ ] Distributed tracing configured if running multiple services (OpenTelemetry, Jaeger, Datadog APM, AWS X-Ray)
- [ ] Four golden signals monitored: latency (p50/p95/p99), traffic (req/s), error rate (%), saturation (CPU/memory/connections)
- [ ] Alerting configured: error rate spike > 1%, p99 latency > threshold, 5xx rate > 0.5% triggers Slack/PagerDuty notification
- [ ] Dashboards created for real-time API health: request volume, error rates, latency percentiles, active connections
- [ ] Log retention configured: logs persist beyond container/pod restarts (ship to Datadog, ELK, Loki, CloudWatch)
- [ ] PII scrubbed from logs — never log passwords, tokens, credit card numbers, SSNs, or full email addresses
- [ ] Health check endpoint implemented: `/health` for liveness, `/ready` for readiness (include dependency checks in readiness)
- [ ] SLOs defined for critical user journeys (e.g., 99.9% of login requests complete in < 500ms) — tracked with error budget burn rate alerts
- [ ] Anomaly detection on key metrics — alert on deviation from baseline, not just static thresholds (catches slow degradation)
- [ ] Dependency health monitored — track latency and error rates for every external service call (database, cache, third-party APIs)
- [ ] Audit logging for all privileged actions: admin operations, data exports, permission changes, config modifications (append-only, tamper-evident)

### Performance & Scaling

- [ ] Database queries profiled — N+1 queries identified and eliminated (use query logging, Django Debug Toolbar, Prisma query events)
- [ ] Database indexes created for all columns used in WHERE, JOIN, ORDER BY clauses in production query patterns
- [ ] Caching layer configured for hot read paths: Redis, Memcached, or application-level cache with TTL and invalidation strategy
- [ ] API response times validated under load: p50 < 100ms, p99 < 500ms for standard endpoints (run load test before launch)
- [ ] Load test run with realistic traffic patterns and data volume (k6, Artillery, Locust, or Grafana k6 Cloud)
- [ ] Service is stateless — no in-memory session storage, no local file dependencies, can scale horizontally without sticky sessions
- [ ] Database connection pooling configured with appropriate min/max pool sizes (PgBouncer for Postgres; HikariCP for JVM)
- [ ] Pagination implemented on all list endpoints — never return unbounded result sets (use cursor-based pagination for large datasets)
- [ ] Expensive operations offloaded to background jobs/queues (email sending, image processing, report generation)
- [ ] Database has read replicas configured for read-heavy workloads (if applicable — confirm replication lag is acceptable)
- [ ] Query result caching with cache invalidation strategy — avoid thundering herd on cache expiry (use stale-while-revalidate or locking)
- [ ] Slow query log enabled — queries exceeding threshold (e.g., > 200ms) are logged and reviewed regularly
- [ ] Auto-scaling policies configured and tested — service scales out under load and scales in during low traffic (verify both directions)
- [ ] Connection limits set on all external dependencies — prevent a single tenant or burst from exhausting connection pools

### Reliability & Resilience

- [ ] Graceful shutdown handling: drain in-flight requests on SIGTERM, close database connections cleanly (critical for zero-downtime deploys)
- [ ] Retry logic with exponential backoff and jitter for external service calls — never retry on 4xx client errors
- [ ] Circuit breaker pattern implemented for critical external dependencies (prevent cascade failures)
- [ ] Database migration rollback tested — can you reverse the latest migration without data loss?
- [ ] Automated database backup schedule configured and restore procedure tested at least once (verify backups are not corrupted)
- [ ] Timeouts configured for all external HTTP calls, database queries, and queue operations (default unlimited timeout = memory leak risk)
- [ ] Dead letter queues configured for failed async jobs — failed messages are not silently dropped
- [ ] Idempotency handled for critical mutation endpoints (payments, transfers) — retry-safe with idempotency keys
- [ ] Bulkhead pattern implemented — failures in non-critical services don't bring down critical paths (isolate thread pools/connection pools)
- [ ] Graceful degradation configured — if a dependency is down, serve cached/default data instead of failing entirely
- [ ] Leader election or distributed locking in place for operations that must run on exactly one instance (cron jobs, migrations, queue consumers)
- [ ] Poison message handling — malformed messages in queues are detected and routed to DLQ after max retries, not retried infinitely
- [ ] RTO and RPO defined for each data store — backup frequency and failover time align with business requirements

### Data Management & Compliance

- [ ] Database schema versioned with migration tool (Flyway, Alembic, Prisma Migrate, Knex) — never apply DDL manually in production
- [ ] Schema migrations are backward-compatible — old app version can run against new schema during rolling deploy (expand-then-contract pattern)
- [ ] Data retention policies defined and automated — PII is purged or anonymized after retention period expires
- [ ] GDPR compliance: right to access (data export), right to erasure (data deletion), right to portability implemented
- [ ] CCPA compliance: "Do Not Sell" opt-out mechanism, data disclosure on request, deletion on request
- [ ] Data classification applied — PII, financial data, health data (PHI) identified and protected with appropriate encryption and access controls
- [ ] Personally identifiable information (PII) encrypted at rest using AES-256-GCM or platform-managed encryption
- [ ] Data anonymization or pseudonymization applied to non-production environments — never use real user data in staging/dev
- [ ] Cross-border data transfer compliance verified — data residency requirements met for EU (GDPR), China (PIPL), etc.
- [ ] Data backup encryption enabled — backups are encrypted at rest and access-controlled independently from production data
- [ ] Soft-delete and data recovery workflow tested — accidentally deleted data can be restored within the retention window
- [ ] Database connection encryption enforced — TLS required for all database connections (no plaintext database traffic)

### Documentation & Operations

- [ ] OpenAPI / Swagger spec is up-to-date and matches actual API behavior (auto-generate from code where possible)
- [ ] Runbooks written for top 5 most likely incidents: database full, service down, spike in errors, memory leak, dependency outage
- [ ] On-call rotation assigned — someone is responsible for production issues and knows how to respond
- [ ] Incident response plan documented: who gets paged, escalation path, communication channel, postmortem process
- [ ] Service ownership documented: team name, Slack channel, PagerDuty policy, repo link
- [ ] API changelog maintained for breaking changes — consumers know what changed and when
- [ ] Architecture decision records (ADRs) maintained for significant technical decisions (why, alternatives considered, trade-offs)
- [ ] Dependency map documented — all upstream and downstream services, databases, queues, and third-party APIs visualized
- [ ] Capacity planning reviewed — current resource utilization documented with projections for 6-12 months growth
- [ ] Postmortem template and process defined — blameless postmortems required for all SEV1/SEV2 incidents within 48 hours

### Security (continued)

- [ ] TLS 1.2+ enforced on all endpoints; TLS 1.0 and 1.1 disabled (check with SSL Labs test)
- [ ] Sensitive data encrypted at rest: PII, financial data, health data (AES-256-GCM or platform-managed encryption)
- [ ] SAST (Static Application Security Testing) scan run with zero critical/high findings unresolved (Snyk, Semgrep, CodeQL)
- [ ] Dependency vulnerabilities audited — no critical CVEs in production dependencies (`npm audit`, `pip audit`, `cargo audit`, `go vuln check`)
- [ ] CORS policy explicitly configured with allowed origins list — never `Access-Control-Allow-Origin: *` on authenticated APIs
- [ ] Security headers set: `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy`
- [ ] API responses don't leak internal details: no stack traces, no framework version headers, no database error messages in 5xx responses
- [ ] Sensitive data has retention policies — old data is automatically purged or anonymized per compliance requirements
- [ ] DAST (Dynamic Application Security Testing) scan run against staging environment — OWASP ZAP or Burp Suite with zero critical findings
- [ ] Container images scanned for vulnerabilities (Trivy, Snyk Container, Grype) — base images updated to latest patched versions
- [ ] Least privilege IAM roles for all cloud resources — no wildcards (`*`) in production IAM policies
- [ ] Network segmentation enforced — databases and internal services not accessible from the public internet (private subnets, security groups)
- [ ] API authentication bypass tested — pen test confirms no endpoint is accessible without valid credentials
- [ ] Supply chain security: signed commits, verified base images, SBOM (Software Bill of Materials) generated for production artifacts

### Distributed Systems (if microservices)

- [ ] Service discovery configured and tested — services can find each other without hardcoded addresses (Consul, Kubernetes DNS, AWS Cloud Map)
- [ ] Distributed tracing spans cover the full request lifecycle across all services (OpenTelemetry collector deployed)
- [ ] Inter-service communication secured with mTLS or service mesh (Istio, Linkerd, Consul Connect)
- [ ] Event-driven communication uses durable message broker (Kafka, RabbitMQ, SQS) — not direct HTTP calls for async workflows
- [ ] Contract testing between services — producer and consumer contracts verified in CI (Pact, Specmatic)
- [ ] Saga pattern or compensating transactions implemented for distributed workflows that span multiple services
- [ ] Service mesh traffic policies configured: retries, timeouts, circuit breakers, and canary routing at the mesh level
- [ ] Cross-service schema compatibility verified — Protobuf/Avro schema registry enforces backward compatibility for event schemas

---

## 🟢 NICE-TO-HAVE

- [ ] API versioning strategy implemented and documented (`/v1/`, header-based, or query param — pick one and be consistent)
- [ ] Canary deployment or blue-green deployment configured for zero-downtime releases
- [ ] Feature flags integrated for safe rollouts and instant kill switches (LaunchDarkly, Flagsmith, Unleash, or config-based)
- [ ] Idempotency keys supported on all mutation endpoints (not just payments — any POST that creates resources)
- [ ] Webhook delivery includes retry with exponential backoff and HMAC signature verification for consumers
- [ ] Shadow traffic / traffic replay testing (GoReplay) run before major refactors to compare old vs new behavior
- [ ] Request/response examples in OpenAPI spec for every endpoint (improves developer experience for API consumers)
- [ ] Chaos engineering: tested behavior under dependency failure, network partition, high latency (Chaos Monkey, Litmus, Toxiproxy)
- [ ] Automated canary analysis: new deployments are auto-rolled-back if error rate or latency degrades vs baseline
- [ ] Data migration scripts are idempotent and can be re-run safely
- [ ] API deprecation strategy documented: how much notice, sunset headers, migration guides for consumers
- [ ] GraphQL persisted queries or query allowlisting for production (prevent arbitrary query abuse)
- [ ] Cost allocation tags on all cloud resources — team/service/environment traceable for FinOps
- [ ] Synthetic monitoring: automated tests simulate key user journeys from multiple regions every N minutes (Checkly, Datadog Synthetics)
- [ ] Request tracing includes business context (user ID, tenant ID, feature flag state) for debugging production issues
- [ ] Database query plan analysis automated — CI warns when a migration introduces a sequential scan on large tables
- [ ] Multi-region failover tested — traffic routes to secondary region within RTO if primary region goes down
- [ ] Write-ahead logging or event sourcing for critical business operations — enables audit trail and temporal queries
- [ ] gRPC or binary protocol used for high-throughput internal service communication (lower overhead than JSON over HTTP)
- [ ] API gateway configured for cross-cutting concerns: auth, rate limiting, request transformation, response caching (Kong, Apigee, AWS API Gateway)
