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

### Input Validation & Injection Prevention

- [ ] All input validated and sanitized server-side — never trust client-side validation alone
- [ ] SQL injection prevented: use parameterized queries or ORM query builders, never string concatenation for SQL
- [ ] NoSQL injection prevented: validate query operators, reject `$` prefixed keys in MongoDB user input
- [ ] Request body size limits configured (e.g., `body-parser` limit, nginx `client_max_body_size`) to prevent memory exhaustion
- [ ] Content-Type validation on all POST/PUT/PATCH endpoints — reject unexpected content types
- [ ] File upload validation: restrict allowed MIME types, enforce max size, scan for malware, store outside webroot
- [ ] Path traversal prevented: never use user input directly in file system paths (`../../../etc/passwd`)
- [ ] GraphQL: depth limiting, query complexity analysis, and introspection disabled in production (if applicable)

### Secrets & Configuration

- [ ] No secrets in codebase, Dockerfiles, or container images — scan with `git-secrets`, `truffleHog`, or `gitleaks`
- [ ] Secrets managed via dedicated vault or secrets manager (AWS Secrets Manager, HashiCorp Vault, Doppler, 1Password Secrets Automation)
- [ ] Environment-specific configs strictly separated: dev/staging/prod use different databases, keys, and service URLs
- [ ] Database credentials rotated before going to production (don't launch with the same password used during development)
- [ ] Default credentials changed on all services: databases, admin panels, message queues, cache servers
- [ ] `.env` files excluded from Docker images and Git (verify `.gitignore` and `.dockerignore`)

### Rate Limiting & DoS Protection

- [ ] Rate limiting on all public endpoints with stricter limits on auth routes: login, signup, OTP, password reset (e.g., 5/min for login)
- [ ] WAF (Web Application Firewall) configured in front of the API (Cloudflare, AWS WAF, Fastly) to block common attack patterns
- [ ] Request timeout configured on all routes to prevent slowloris and slow-read attacks (e.g., 30s max)
- [ ] Payload size limits enforced at the reverse proxy / load balancer level as an additional layer
- [ ] API abuse detection: monitor for credential stuffing patterns, automated scraping, enumeration attacks

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

### Reliability & Resilience

- [ ] Graceful shutdown handling: drain in-flight requests on SIGTERM, close database connections cleanly (critical for zero-downtime deploys)
- [ ] Retry logic with exponential backoff and jitter for external service calls — never retry on 4xx client errors
- [ ] Circuit breaker pattern implemented for critical external dependencies (prevent cascade failures)
- [ ] Database migration rollback tested — can you reverse the latest migration without data loss?
- [ ] Automated database backup schedule configured and restore procedure tested at least once (verify backups are not corrupted)
- [ ] Timeouts configured for all external HTTP calls, database queries, and queue operations (default unlimited timeout = memory leak risk)
- [ ] Dead letter queues configured for failed async jobs — failed messages are not silently dropped
- [ ] Idempotency handled for critical mutation endpoints (payments, transfers) — retry-safe with idempotency keys

### Documentation & Operations

- [ ] OpenAPI / Swagger spec is up-to-date and matches actual API behavior (auto-generate from code where possible)
- [ ] Runbooks written for top 5 most likely incidents: database full, service down, spike in errors, memory leak, dependency outage
- [ ] On-call rotation assigned — someone is responsible for production issues and knows how to respond
- [ ] Incident response plan documented: who gets paged, escalation path, communication channel, postmortem process
- [ ] Service ownership documented: team name, Slack channel, PagerDuty policy, repo link
- [ ] API changelog maintained for breaking changes — consumers know what changed and when

### Security (continued)

- [ ] TLS 1.2+ enforced on all endpoints; TLS 1.0 and 1.1 disabled (check with SSL Labs test)
- [ ] Sensitive data encrypted at rest: PII, financial data, health data (AES-256-GCM or platform-managed encryption)
- [ ] SAST (Static Application Security Testing) scan run with zero critical/high findings unresolved (Snyk, Semgrep, CodeQL)
- [ ] Dependency vulnerabilities audited — no critical CVEs in production dependencies (`npm audit`, `pip audit`, `cargo audit`, `go vuln check`)
- [ ] CORS policy explicitly configured with allowed origins list — never `Access-Control-Allow-Origin: *` on authenticated APIs
- [ ] Security headers set: `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy`
- [ ] API responses don't leak internal details: no stack traces, no framework version headers, no database error messages in 5xx responses
- [ ] Sensitive data has retention policies — old data is automatically purged or anonymized per compliance requirements

---

## 🟢 NICE-TO-HAVE

- [ ] API versioning strategy implemented and documented (`/v1/`, header-based, or query param — pick one and be consistent)
- [ ] Canary deployment or blue-green deployment configured for zero-downtime releases
- [ ] Feature flags integrated for safe rollouts and instant kill switches (LaunchDarkly, Flagsmith, Unleash, or config-based)
- [ ] Idempotency keys supported on all mutation endpoints (not just payments — any POST that creates resources)
- [ ] Webhook delivery includes retry with exponential backoff and HMAC signature verification for consumers
- [ ] Shadow traffic / traffic replay testing (GoReplay) run before major refactors to compare old vs new behavior
- [ ] API rate limit headers returned to clients: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
- [ ] Request/response examples in OpenAPI spec for every endpoint (improves developer experience for API consumers)
- [ ] Chaos engineering: tested behavior under dependency failure, network partition, high latency (Chaos Monkey, Litmus, Toxiproxy)
- [ ] Automated canary analysis: new deployments are auto-rolled-back if error rate or latency degrades vs baseline
- [ ] Data migration scripts are idempotent and can be re-run safely
- [ ] API deprecation strategy documented: how much notice, sunset headers, migration guides for consumers
