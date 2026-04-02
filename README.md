# production-checklist

> Ship confidently. A tiered, context-aware production readiness checklist for any coding agent.

When you're about to deploy — web app, mobile app, API, or smart contract — this skill gives you a structured, severity-tiered checklist tailored to your stack. No more forgetting security headers, missing store requirements, or shipping unprotected endpoints.

**200+ actionable items** across 4 deployment types. Every item is a binary pass/fail check with a fix hint. No fluff.

## What it does

1. **Detects** your deployment type from conversation context (web, mobile, API, smart contract)
2. **Routes** to the right domain-specific reference file
3. **Presents** a checklist tiered by severity: 🔴 Critical / 🟡 Important / 🟢 Nice-to-have
4. **Skips** irrelevant items based on your stack signals
5. **Produces** a pass/fail scorecard you can act on immediately
6. **Offers fixes** for any failing critical items — with actual code/config, not vague advice

## Deployment types covered

| Type | File | Critical | Important | Nice-to-have | Total |
|------|------|----------|-----------|-------------|-------|
| Web / Frontend | `web.md` | 25 | 42 | 15 | **82** |
| Mobile (iOS & Android) | `mobile.md` | 28 | 40 | 13 | **81** |
| Backend API | `api.md` | 27 | 47 | 12 | **86** |
| Smart Contract (EVM) | `smart-contract.md` | 27 | 40 | 12 | **79** |

### What's covered per type

**Web:** Security headers, CSP, CORS, HTTPS, auth token storage, Lighthouse, Core Web Vitals, SEO/OG tags, WCAG accessibility, error tracking, cross-browser testing, legal/GDPR compliance

**Mobile:** App Store & Play Store compliance, code signing, ProGuard/R8, certificate pinning, ATS, ASO optimization, crash reporting, ATT/GDPR, account deletion requirement, deep linking

**API:** OAuth/JWT auth, injection prevention, secrets management, rate limiting, structured logging, distributed tracing, load testing, circuit breakers, graceful shutdown, SAST/DAST scans

**Smart Contract:** Audit readiness, reentrancy protection, fuzz testing, invariant tests, multi-sig ownership, pausability, gas optimization, oracle security, flash loan vectors, MEV protection

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
"I'm pushing to prod tonight — run preflight"
"/production-checklist"
```

The skill auto-detects your deployment type from context. If ambiguous, it asks once.

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
SKILL.md (routing logic, < 100 lines)
  ├── detects deployment type from conversation signals
  ├── reads ONLY the matching reference file
  └── walks developer through tier-by-tier review

references/
  ├── web.md            ← self-contained, 82 items
  ├── mobile.md         ← self-contained, 81 items
  ├── api.md            ← self-contained, 86 items
  └── smart-contract.md ← self-contained, 79 items
```

Each reference file is independent. No cross-file dependencies. The agent loads only what's needed.

## Contributing

PRs welcome! Each `references/*.md` file is self-contained. To add or improve:

1. **Add items:** Follow the format `- [ ] Item description (fix hint in parens)`
2. **New deployment type:** Create `references/your-type.md` with 🔴/🟡/🟢 tiers, add signals to `SKILL.md`
3. **Update existing:** Fix outdated tool references, add new best practices, improve fix hints

Keep severity tiers consistent. 🔴 = causes real damage if missed. 🟡 = causes pain within a week. 🟢 = best practice.

## Roadmap

| Version | Feature |
|---------|---------|
| v1.1 | `docker.md` — Container & Kubernetes production readiness |
| v1.2 | `nextjs.md` — Next.js-specific checks (ISR, edge runtime, middleware, App Router) |
| v1.3 | `supabase.md` — Supabase production hardening (RLS policies, backups, connection pooling) |
| v2.0 | Agent-mode scan — auto-check `.env`, `package.json`, manifests, config files when file access available |
| v2.1 | `base-chain.md` — Base / L2-specific deployment checks |

## License

MIT
