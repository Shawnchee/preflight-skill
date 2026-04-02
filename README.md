# production-checklist

> Ship confidently. A tiered, context-aware production readiness checklist for any coding agent.

When you're about to deploy — web app, mobile app, API, smart contract, payment system, or infrastructure — this skill gives you a structured, severity-tiered checklist tailored to your stack. No more forgetting security headers, missing store requirements, or shipping unprotected endpoints.

**750+ actionable items** across 6 deployment types. Every item is a binary pass/fail check with a fix hint. No fluff. FAANG-grade thoroughness covering PCI DSS, SRE best practices, OWASP, GDPR/CCPA, SOX, and the Google SRE Production Readiness Review framework.

## What it does

1. **Detects** your deployment type from conversation context (web, mobile, API, smart contract, payment, infrastructure)
2. **Routes** to the right domain-specific reference file
3. **Presents** a checklist tiered by severity: 🔴 Critical / 🟡 Important / 🟢 Nice-to-have
4. **Skips** irrelevant items based on your stack signals
5. **Produces** a pass/fail scorecard you can act on immediately
6. **Offers fixes** for any failing critical items — with actual code/config, not vague advice

## Deployment types covered

| Type | File | Critical | Important | Nice-to-have | Total |
|------|------|----------|-----------|-------------|-------|
| Web / Frontend | `web.md` | 40 | 62 | 24 | **126** |
| Mobile (iOS & Android) | `mobile.md` | 35 | 63 | 21 | **119** |
| Backend API | `api.md` | 40 | 73 | 20 | **133** |
| Smart Contract (EVM) | `smart-contract.md` | 27 | 52 | 17 | **96** |
| Payment & Financial | `payment.md` | 32 | 50 | 18 | **100** |
| Infrastructure & SRE | `infrastructure.md` | 38 | 65 | 20 | **123** |

### What's covered per type

**Web:** Security headers, CSP, CORS, HTTPS, OWASP protections, auth token storage, PCI DSS 4.0 client-side security, Lighthouse, Core Web Vitals, SEO/OG tags, WCAG 2.2 accessibility, i18n/RTL support, error tracking, cross-browser testing, payment frontend integration, legal/GDPR/CCPA compliance

**Mobile:** App Store & Play Store compliance, EU DMA, code signing, ProGuard/R8, certificate pinning, ATS, binary obfuscation, ASO optimization, crash reporting, ATT/GDPR, in-app purchase validation, subscription lifecycle, account deletion, deep linking, VoiceOver/TalkBack accessibility

**API:** OAuth/JWT auth, MFA, injection prevention (SQL, NoSQL, SSRF, XXE, command injection), secrets management, rate limiting, structured logging, distributed tracing, SLOs/error budgets, load testing, circuit breakers, bulkheads, graceful shutdown, SAST/DAST scans, data management, GDPR/CCPA compliance, microservices patterns, contract testing, service mesh

**Smart Contract:** Audit readiness, reentrancy protection, fuzz testing, invariant tests, multi-sig ownership, pausability, gas optimization, oracle security, flash loan vectors, MEV protection, DeFi-specific checks, cross-chain bridge security, governance attack prevention, formal verification

**Payment:** PCI DSS v4.0 compliance, tokenization, double-entry bookkeeping, idempotency, fraud detection (AVS, 3DS2, velocity checks), reconciliation pipeline, multi-currency (ISO 4217), tax engine integration, subscription dunning, KYC/AML, SOX audit trail, chargeback management

**Infrastructure:** Infrastructure as Code (Terraform/Pulumi), container security, Kubernetes hardening, IAM least privilege, secrets management, SLOs with error budgets, disaster recovery (RTO/RPO), capacity planning, chaos engineering, incident management, on-call rotation, postmortem process, SOC 2 controls, GitOps, service mesh, FinOps

## Install

### Claude Code (Plugin Marketplace)

```bash
/plugin marketplace add byshawnchee/production-checklist-skill
/plugin install production-checklist-plugin@production-checklist-skill
```

### Manual install (Claude Code)

```bash
git clone https://github.com/byshawnchee/production-checklist-skill
# Global (all projects):
cp -r production-checklist-skill/production-checklist ~/.claude/skills/
# Project-level:
cp -r production-checklist-skill/production-checklist .claude/skills/
```

### Cursor

```bash
git clone https://github.com/byshawnchee/production-checklist-skill
cp -r production-checklist-skill/production-checklist .cursor/skills/
```

### Windsurf

```bash
git clone https://github.com/byshawnchee/production-checklist-skill
cp -r production-checklist-skill/production-checklist .windsurf/skills/
```

### Gemini CLI

```bash
skillkit install byshawnchee/production-checklist-skill
# or manual:
cp -r production-checklist-skill/production-checklist ~/.gemini/skills/
```

### Codex CLI / OpenCode

```bash
git clone https://github.com/byshawnchee/production-checklist-skill
cp -r production-checklist-skill/production-checklist ~/.codex/skills/
# OpenCode: follows SKILL.md standard — compatible
```

## Usage

Just tell your agent you're about to deploy:

```
"I'm about to deploy my Next.js app to Vercel — run the production checklist"
"We're shipping to App Store tomorrow — are we ready?"
"Check if my Fastify API is production-ready"
"Pre-audit checklist for my ERC-20 contract before mainnet"
"Our Stripe integration is going live — run payment checklist"
"Review our Kubernetes infrastructure before launch"
"I'm pushing to prod tonight — run preflight"
"/production-checklist"
```

The skill auto-detects your deployment type from context. If ambiguous, it asks once. For full-stack projects, it detects multiple types and runs them sequentially.

## Example output

```
Production Readiness Score
══════════════════════════
✅ Critical:     8/10 passed
⚠️  Important:   12/15 passed
💡 Nice-to-have: 3/8 passed

VERDICT: ⚠️ NOT READY — 2 critical items unresolved

Blockers:
1. CORS is set to * in production — restrict to known origins
2. Rate limiting not configured on login endpoint

Want me to generate fixes for these?
```

## How it works

```
SKILL.md (routing logic)
  ├── detects deployment type from conversation signals
  ├── reads ONLY the matching reference file(s)
  └── walks developer through tier-by-tier review

references/
  ├── web.md            ← Web / Frontend (126 items)
  ├── mobile.md         ← iOS & Android (119 items)
  ├── api.md            ← Backend API (133 items)
  ├── smart-contract.md ← Solidity / EVM (96 items)
  ├── payment.md        ← Payment & Financial (100 items)
  └── infrastructure.md ← Infrastructure & SRE (123 items)
```

Each reference file is independent. No cross-file dependencies. The agent loads only what's needed.

## Sources & standards referenced

This checklist was compiled from industry-leading sources:

- [Google SRE Book — Production Readiness Review](https://sre.google/sre-book/evolving-sre-engagement-model/)
- [Google SRE Book — Launch Checklist](https://sre.google/sre-book/launch-checklist/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [PCI DSS v4.0 Standards](https://www.pcisecuritystandards.org/standards/)
- [OWASP Application Security Verification Standard (ASVS) v5.0](https://owasp.org/www-project-application-security-verification-standard/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [WCAG 2.2 Accessibility Guidelines](https://www.w3.org/WAI/standards-guidelines/wcag/)
- [Stripe PCI Compliance Guide](https://stripe.com/guides/pci-compliance)
- [SRE Checklist (GitHub)](https://github.com/bregman-arie/sre-checklist)

## Contributing

PRs welcome! Each `references/*.md` file is self-contained. To add or improve:

1. **Add items:** Follow the format `- [ ] Item description (fix hint in parens)`
2. **New deployment type:** Create `references/your-type.md` with 🔴/🟡/🟢 tiers, add signals to `SKILL.md`
3. **Update existing:** Fix outdated tool references, add new best practices, improve fix hints

Keep severity tiers consistent. 🔴 = causes real damage if missed. 🟡 = causes pain within a week. 🟢 = best practice.

## Roadmap

| Version | Feature |
|---------|---------|
| v2.0 | Agent-mode scan — auto-check `.env`, `package.json`, manifests, config files when file access available |
| v2.1 | `nextjs.md` — Next.js-specific checks (ISR, edge runtime, middleware, App Router) |
| v2.2 | `supabase.md` — Supabase production hardening (RLS policies, backups, connection pooling) |
| v2.3 | `base-chain.md` — Base / L2-specific deployment checks |
| v3.0 | Auto-remediation — agent detects and fixes common issues automatically |

## License

MIT
