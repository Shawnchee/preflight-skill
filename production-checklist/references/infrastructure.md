# Infrastructure & SRE Production Checklist

---

## 🔴 CRITICAL (ship-blockers)

### Infrastructure as Code (IaC)

- [ ] All infrastructure defined in code — Terraform, Pulumi, CloudFormation, or CDK (no manual console-created resources in production)
- [ ] IaC state stored remotely with locking (Terraform: S3 + DynamoDB lock; Pulumi: Pulumi Cloud or S3 backend)
- [ ] Infrastructure CI/CD pipeline: `plan` on PR, `apply` on merge to main — no `terraform apply` from developer laptops
- [ ] Drift detection automated — alert when actual infrastructure deviates from IaC state (scheduled `terraform plan` or CloudFormation drift detection)
- [ ] IaC modules versioned and pinned — no `latest` tags or unpinned module sources in production
- [ ] Secrets and credentials NEVER in IaC code or state files — use secrets manager references or encrypted variables
- [ ] Destructive changes require approval — resource deletion or replacement gated by policy (Terraform `prevent_destroy`, OPA policies, or manual approval step)
- [ ] Disaster recovery: infrastructure can be fully recreated from IaC in a new region/account within documented RTO

### Compute & Container Orchestration

- [ ] Container images use minimal base images (distroless, Alpine, or scratch) — no full OS distributions in production
- [ ] Container images scanned for vulnerabilities in CI pipeline (Trivy, Snyk, Grype) — zero critical/high CVEs before deploy
- [ ] Container images are immutable and tagged with Git SHA — no `latest` tag in production deployments
- [ ] Resource requests and limits set on all containers: CPU and memory requests defined, memory limits set (CPU limits are optional — omitting them avoids throttling, but set them if you need predictable multi-tenant isolation)
- [ ] Pod disruption budgets (PDB) configured for all production workloads — ensure minimum available replicas during node maintenance
- [ ] Horizontal Pod Autoscaler (HPA) configured and tested: scales out under load, scales in during low traffic (verify both directions)
- [ ] Liveness and readiness probes configured on all containers — readiness includes dependency health checks
- [ ] No containers run as root — `runAsNonRoot: true` in security context, `readOnlyRootFilesystem: true` where possible
- [ ] Pod anti-affinity rules ensure replicas spread across nodes/zones — no single node failure takes down all replicas
- [ ] Graceful shutdown: containers handle SIGTERM, drain connections within `terminationGracePeriodSeconds`
- [ ] Init containers handle dependency ordering — don't rely on service readiness for startup sequence
- [ ] Container registry access controlled — only CI/CD pipeline can push images, production cluster pulls with read-only credentials

### Networking & Load Balancing

- [ ] Load balancer configured with health checks — unhealthy backends are automatically removed from rotation
- [ ] TLS termination at load balancer or ingress with managed certificates (AWS ACM, Let's Encrypt, Google-managed certs)
- [ ] Network segmentation enforced: databases and internal services in private subnets, not accessible from the internet
- [ ] Firewall rules / security groups follow least privilege — only required ports open, source IPs restricted
- [ ] DNS TTL set appropriately: low TTL (60-300s) for services that may failover, higher TTL for stable endpoints
- [ ] DDoS protection enabled at the edge (Cloudflare, AWS Shield, Google Cloud Armor)
- [ ] Internal service communication encrypted with mTLS or within a VPC/private network — no plaintext traffic between services
- [ ] Egress filtering configured — production workloads can only reach explicitly allowed external endpoints
- [ ] IPv6 support tested if enabled — no broken connectivity for IPv6-only clients

### Identity & Access Management (IAM)

- [ ] Root / super-admin account secured with MFA and hardware key — never used for day-to-day operations
- [ ] Least privilege IAM policies: no wildcard (`*`) permissions in production — each service has a scoped role
- [ ] Service accounts / instance roles used for machine-to-machine auth — no long-lived access keys
- [ ] IAM access reviewed quarterly: remove stale users, reduce over-provisioned permissions
- [ ] Break-glass procedure documented: emergency access path when normal auth is unavailable (documented, audited, alarmed)
- [ ] SSO enforced for all team access to cloud console and internal tools (Okta, Azure AD, Google Workspace)
- [ ] Cloud API audit logging enabled and monitored (AWS CloudTrail, GCP Audit Logs, Azure Activity Log) — alert on suspicious activity
- [ ] Temporary credentials used wherever possible — AWS STS AssumeRole, GCP Workload Identity, Azure Managed Identity

### Secrets Management

- [ ] Centralized secrets manager deployed: HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager, Azure Key Vault
- [ ] Secrets injected at runtime — never baked into container images, config maps, or Helm values
- [ ] Secret rotation automated on schedule — database credentials, API keys, TLS certificates rotate without downtime
- [ ] Secret access audited — every read/write to secrets manager is logged with actor identity and timestamp
- [ ] Encryption keys managed with KMS — customer-managed keys for sensitive workloads, key rotation enabled
- [ ] Secrets never appear in CI/CD logs — pipeline tools configured to mask/redact secret values in output

---

## 🟡 IMPORTANT (should fix before launch)

### Observability & Monitoring

- [ ] Three pillars configured: structured logs (ELK, Loki, CloudWatch), metrics (Prometheus, Datadog, CloudWatch), traces (OpenTelemetry, Jaeger)
- [ ] OpenTelemetry Collector deployed as DaemonSet or sidecar — vendor-agnostic telemetry pipeline
- [ ] SLOs defined for critical user journeys: availability and latency targets with error budget tracking
- [ ] Error budget burn rate alerts: fast-burn (> 10x) pages immediately, slow-burn (> 2x) creates ticket
- [ ] Four golden signals monitored for every service: latency, traffic, error rate, saturation
- [ ] Infrastructure metrics collected: CPU, memory, disk, network for all nodes — alert on saturation (> 80% sustained)
- [ ] Log aggregation configured: all container/pod logs shipped to central store with retention policy (30 days hot, 90 days warm, 1 year cold)
- [ ] PII scrubbed from all logs and traces before ingestion — automated redaction rules for emails, tokens, card numbers
- [ ] Dashboards created: service health overview, infrastructure utilization, deployment tracker, error rate trends
- [ ] Alerting hierarchy defined: SEV1 → pages on-call, SEV2 → Slack notification, SEV3 → ticket created — no alert fatigue
- [ ] Synthetic monitoring: automated tests simulate critical user journeys from multiple geographic regions every 5 minutes
- [ ] Cost monitoring dashboards: per-service and per-team cost allocation tracked with anomaly alerts for unexpected spend

### Deployment & Release

- [ ] CI/CD pipeline fully automated: commit → build → test → scan → deploy to staging → deploy to production
- [ ] Deployment strategy configured: rolling update, blue-green, or canary — never stop-the-world deploys
- [ ] Canary deployments for critical services: route 5% traffic to new version, monitor for 15 minutes, then expand
- [ ] Automated rollback: deployment auto-reverts if error rate or latency exceeds threshold during canary phase
- [ ] Deployment frequency tracked — aim for multiple deploys per day with confidence (DORA metric)
- [ ] Rollback tested and documented: team can revert to previous version within 5 minutes (one-click rollback)
- [ ] Database migrations decoupled from application deploys — run expand phase before deploy, contract phase after
- [ ] Feature flags used for risky changes — deploy dark code, enable gradually, kill instantly if needed
- [ ] Deployment notifications sent to team channel: who deployed what, when, with link to diff and rollback
- [ ] Artifact promotion: same build artifact progresses from staging → production (no rebuild for production)
- [ ] Deploy freeze process defined: how to pause deployments during incidents, holidays, or high-traffic events

### Disaster Recovery & Business Continuity

- [ ] RTO (Recovery Time Objective) and RPO (Recovery Point Objective) defined per service tier — documented and agreed with business
- [ ] Database backups automated and tested: backup runs on schedule, restore procedure tested quarterly
- [ ] Backup encryption enabled — backups encrypted at rest with separate key from production data
- [ ] Cross-region replication configured for critical data stores — failover region can serve traffic within RTO
- [ ] Disaster recovery runbook documented: step-by-step failover procedure with decision tree and responsible parties
- [ ] DR drill conducted at least annually — full failover to secondary region with timing measured against RTO/RPO
- [ ] Backup retention policy defined and automated: daily (30 days), weekly (12 weeks), monthly (12 months) — or per compliance requirements
- [ ] Point-in-time recovery (PITR) enabled for critical databases — can restore to any second within retention window
- [ ] Multi-AZ deployment for all stateful services — database, cache, message queue survive single-AZ failure
- [ ] Failback procedure documented and tested — how to restore operations to primary region after DR event
- [ ] Chaos testing conducted: randomly kill instances, inject network latency, fail a dependency — verify graceful degradation (Chaos Monkey, Gremlin, Litmus)

### Capacity Planning & Scaling

- [ ] Current resource utilization baselined: CPU, memory, disk, connections, IOPS for all production services
- [ ] Growth projections documented: expected traffic growth for next 6-12 months with scaling plan
- [ ] Auto-scaling tested under load: verify scale-out triggers, scale-in cooldown, and maximum instance limits
- [ ] Database capacity planned: storage growth rate, connection pool limits, read replica needs for projected traffic
- [ ] Rate limits calibrated to actual capacity — reject requests gracefully before system saturates
- [ ] Load test run with 2-3x expected peak traffic — identify bottlenecks before they hit production (k6, Locust, Gatling)
- [ ] Quotas and limits set on cloud resources — prevent runaway costs from auto-scaling bugs or resource leaks
- [ ] Queue depth monitoring with auto-scaling: worker pools scale based on queue backlog
- [ ] Capacity review scheduled quarterly — update projections, adjust resources, plan for upcoming events (seasonal traffic)

### Reliability & Resilience Patterns

- [ ] Circuit breakers configured for all external dependencies — prevent cascade failures when a dependency is down
- [ ] Retry policies with exponential backoff and jitter for all external calls — no retry storms during outages
- [ ] Bulkhead pattern: separate thread/connection pools for critical vs non-critical dependencies
- [ ] Timeout budgets: every external call has an explicit timeout, request-level timeout budget propagated across service chain
- [ ] Graceful degradation: if non-critical dependency fails, serve reduced functionality instead of 500 errors
- [ ] Rate limiting at service mesh / API gateway level — protect backend services from traffic spikes
- [ ] Data replication lag monitored for read replicas — alert if lag exceeds acceptable threshold for use case
- [ ] Leader election / distributed locks for singleton workloads (cron jobs, migration runners) — tested for split-brain scenarios
- [ ] Connection draining on shutdown: load balancer stops sending new requests, existing requests complete within grace period

### Incident Management & On-Call

- [ ] On-call rotation established with primary and secondary — rotation is fair, documented, and compensated
- [ ] Escalation policy defined: if primary doesn't acknowledge within 5 minutes, page secondary, then management
- [ ] Incident response process documented: detection → triage → mitigation → communication → resolution → postmortem
- [ ] Severity levels defined with examples: SEV1 (total outage), SEV2 (degraded for subset), SEV3 (minor impact), SEV4 (no user impact)
- [ ] Communication template ready: status page update, customer notification, internal Slack message — pre-written for common scenarios
- [ ] Status page configured and maintained (Statuspage.io, Instatus, Cachet) — auto-updated by monitoring where possible
- [ ] Incident communication channel auto-created on page (Slack channel, Zoom bridge) with context pinned
- [ ] Runbooks written for top 10 most likely incidents — step-by-step with commands, not just descriptions
- [ ] Blameless postmortem process: required for all SEV1/SEV2 within 48 hours, action items tracked to completion
- [ ] On-call handoff process: outgoing on-call documents active issues, pending deployments, known risks for incoming on-call
- [ ] Game days scheduled quarterly: simulate production failures and practice incident response (tabletop exercises at minimum)

### Security & Compliance Infrastructure

- [ ] Vulnerability scanning automated in CI/CD: SAST, DAST, container scanning, dependency scanning — zero critical before deploy
- [ ] Runtime security monitoring: detect anomalous process execution, file access, network connections in production containers (Falco, Aqua, Prisma Cloud)
- [ ] Network intrusion detection / prevention system (IDS/IPS) in place for production network segments
- [ ] WAF rules tuned and updated: OWASP Core Rule Set active, custom rules for application-specific patterns
- [ ] Immutable infrastructure: production instances are never patched in place — replace with new image containing patches
- [ ] OS and runtime patches applied within SLA: critical CVEs within 24 hours, high within 7 days, medium within 30 days
- [ ] SOC 2 Type II controls documented and evidenced if B2B SaaS (or equivalent compliance framework for your industry)
- [ ] Data classification policy applied to infrastructure: tag resources by data sensitivity level (public, internal, confidential, restricted)
- [ ] Encryption at rest enabled for all data stores: databases, object storage, EBS volumes, backups — using KMS-managed keys
- [ ] Encryption in transit enforced everywhere: TLS for all external traffic, mTLS or VPC for internal traffic
- [ ] Audit logging for infrastructure changes: who created/modified/deleted what resource, when, from which IP
- [ ] SBOM (Software Bill of Materials) generated for all production artifacts — enables rapid response to zero-day CVEs

### Documentation & Operations

- [ ] Architecture diagram up-to-date: services, data stores, message queues, external dependencies, network boundaries
- [ ] Service catalog maintained: every production service listed with owner, repo, runbook, SLO, dependencies, on-call team
- [ ] Architecture Decision Records (ADRs) maintained for all significant infrastructure decisions
- [ ] Dependency map visualized: understand which services depend on which, identify single points of failure
- [ ] Change management process defined: how changes are proposed, reviewed, approved, and rolled back
- [ ] Toil budget tracked: measure and reduce repetitive manual operational work (target < 50% of SRE time on toil per Google SRE book)
- [ ] Knowledge base maintained: troubleshooting guides, FAQ, common issues and resolutions

---

## 🟢 NICE-TO-HAVE (polish)

- [ ] GitOps adopted: ArgoCD or Flux for Kubernetes manifest deployment — cluster state driven from Git
- [ ] Policy as code: OPA/Gatekeeper, Kyverno, or Sentinel policies enforce security and compliance in CI/CD and cluster
- [ ] Service mesh deployed (Istio, Linkerd, Consul Connect) for mTLS, traffic management, and observability across services
- [ ] FinOps practices: reserved instances/savings plans for baseline, spot/preemptible for burst, per-team cost allocation
- [ ] Multi-cloud or multi-region active-active: traffic served from multiple regions/clouds with automatic failover
- [ ] Cluster autoscaler tuned: nodes scale based on pending pod resource requests, scale-down after cooldown period
- [ ] Ephemeral environments: spin up full stack per PR for testing, tear down on merge (Terraform workspaces, Argo CD ApplicationSets)
- [ ] Progressive delivery: Argo Rollouts or Flagger for automated canary analysis with metric-driven promotion
- [ ] Cost anomaly detection: alert when daily spend deviates > 20% from 7-day average
- [ ] Infrastructure testing: Terratest, Checkov, or `terraform validate` in CI — catch misconfigurations before apply
- [ ] Centralized certificate management: auto-provisioned and auto-renewed TLS certificates (cert-manager with Let's Encrypt)
- [ ] Log-based alerting: detect specific error patterns in logs and create alerts (Loki alerting rules, CloudWatch Insights)
- [ ] Observability-as-code: dashboards, alerts, and SLO definitions version-controlled alongside application code (Terraform, Crossplane)
- [ ] Internal developer platform: self-service infrastructure provisioning with guardrails (Backstage, Port, Cortex)
- [ ] Kubernetes namespace isolation: resource quotas, network policies, and RBAC per namespace/team
- [ ] Image signing and verification: Sigstore/Cosign for supply chain security — only signed images run in production
- [ ] Preemptible/spot instance strategy: non-critical workloads run on spot with graceful interruption handling
- [ ] Database schema change management: automated review of migration impact (query plan analysis, lock detection)
- [ ] Distributed tracing with business context: inject user ID, tenant ID, feature flags into trace spans for debugging
- [ ] Operational readiness review (ORR) gate: new services must pass production readiness checklist before first deploy
