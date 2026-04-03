# production-checklist v2.1.0

> Ship confidently. An automated production readiness scanner for any codebase.

Run one command. Get a full report. Know exactly what's missing before you deploy.

This skill scans your codebase — detects your stack, checks for security issues, missing configs, and best practice violations — and produces a detailed pass/fail report. No manual input needed. It reads your code and tells you what's wrong, with file paths and line numbers.

**780 checklist items** across 6 deployment types. Based on Google SRE, AWS Well-Architected, PCI DSS v4.0, OWASP, and WCAG 2.2.

---

## How It Works (30-second version)

```
You type:   /production-checklist

The skill:  1. Scans your project files to detect the stack
            2. Loads the right checklist (web? API? mobile? payment?)
            3. Greps your code for issues (hardcoded secrets, missing headers, etc.)
            4. Produces a report:

╔══════════════════════════════════════╗
║   PRODUCTION READINESS SCAN REPORT  ║
╚══════════════════════════════════════╝

🔴 CRITICAL
1. ❌ FAIL — Hardcoded API key in source code
   src/config.ts:14 → const API_KEY = "sk_live_..."
   Fix: Move to .env, add .env* to .gitignore

2. ✅ PASS — HTTPS enforced
   nginx.conf:3 → redirect 301 https://...

3. ⚠️ MANUAL — SSL certificate auto-renewal
   Cannot verify from code — check your host

SCORECARD: 12/15 critical passed | VERDICT: ⚠️ NOT READY

You say:    "Fix all critical issues"
The skill:  Applies actual code fixes.
```

**That's it.** One prompt. One report. Optional auto-fix.

---

## Install

### Using skills.sh (recommended)

The easiest way — works with 45+ coding agents including Claude Code, Cursor, Windsurf, Codex, and more.

```bash
# Install to your project (team can use it too)
npx skills add Shawnchee/preflight-skill

# Install globally (available in all your projects)
npx skills add Shawnchee/preflight-skill -g

# Install to a specific agent only
npx skills add Shawnchee/preflight-skill -a claude-code
npx skills add Shawnchee/preflight-skill -a cursor
npx skills add Shawnchee/preflight-skill -a windsurf

# Preview what's available before installing
npx skills add Shawnchee/preflight-skill --list

# CI/CD-friendly (no prompts)
npx skills add Shawnchee/preflight-skill -g -a claude-code -y
```

### Manual install (any agent)

```bash
git clone https://github.com/Shawnchee/preflight-skill.git
```

Then copy the `production-checklist/` folder to your agent's skills directory:

| Agent | Project-level | Global |
|-------|--------------|--------|
| **Claude Code** | `.claude/skills/` | `~/.claude/skills/` |
| **Cursor** | `.cursor/skills/` | `~/.cursor/skills/` |
| **Windsurf** | `.windsurf/skills/` | `~/.windsurf/skills/` |
| **Codex / OpenCode** | `.codex/skills/` | `~/.codex/skills/` |
| **Gemini CLI** | `.gemini/skills/` | `~/.gemini/skills/` |

> **Note:** Skill directory paths may vary by agent version. Claude Code paths are verified; other agent paths follow the emerging [SKILL.md standard](https://agentskills.io) and may need adjustment.

Example:
```bash
# Project-level (just this repo)
cp -r preflight-skill/production-checklist .claude/skills/

# Global (all your projects)
cp -r preflight-skill/production-checklist ~/.claude/skills/
```

### Uninstall

```bash
# Via skills.sh
npx skills remove production-checklist

# Manual
rm -rf .claude/skills/production-checklist
```

### Version

Current version: **2.1.0** ([changelog](CHANGELOG.md))

### Updating

```bash
# Via skills.sh
npx skills update production-checklist

# Manual
cd preflight-skill && git pull
cp -r production-checklist/ ~/.claude/skills/production-checklist/
```

---

## Usage

Just tell your coding agent you want a production check. Any of these work:

```
/production-checklist
"Run the production checklist"
"Is my app production-ready?"
"Preflight check before deploy"
"Check if this is ready to ship"
```

The skill auto-detects your stack from the files in your project. You don't need to specify what you're building.

### What It Detects Automatically

| If your project has... | It runs the... |
|----------------------|---------------|
| `package.json` with React/Next/Vue/Svelte | Web frontend checklist |
| `Podfile`, `build.gradle`, `pubspec.yaml` | Mobile app checklist |
| `server.js`, `manage.py`, `go.mod`, route files | Backend API checklist |
| `*.sol`, `foundry.toml`, `hardhat.config.*` | Smart contract checklist |
| Stripe/PayPal/Adyen in dependencies | Payment system checklist |
| `*.tf`, `Dockerfile`, `k8s/`, CI configs | Infrastructure checklist |

**Full-stack projects?** It detects multiple types and runs all relevant checklists. A Next.js app with Express backend and Stripe gets all three.

---

## What's Covered

| Type | Items | What it checks |
|------|-------|---------------|
| **Web / Frontend** | 144 | Security headers, CSP, CORS, HTTPS, XSS, OWASP, auth tokens, Core Web Vitals, SEO, WCAG 2.2 accessibility, i18n, payment frontend (PCI DSS 4.0), GDPR/CCPA |
| **Mobile (iOS & Android)** | 123 | App Store / Play Store compliance, code signing, certificate pinning, binary security, in-app purchases, subscription lifecycle, crash reporting, accessibility |
| **Backend API** | 148 | Auth (OAuth/JWT/MFA), injection prevention (SQL, NoSQL, SSRF, XXE), rate limiting, SLOs, distributed tracing, circuit breakers, data compliance, microservices |
| **Smart Contract (EVM)** | 108 | Audit readiness, reentrancy, fuzz testing, multi-sig, gas optimization, oracle security, flash loans, MEV, DeFi checks, cross-chain |
| **Payment & Financial** | 112 | PCI DSS v4.0, tokenization, double-entry bookkeeping, fraud detection, reconciliation, multi-currency, tax, subscriptions, KYC/AML, SOX audit |
| **Infrastructure & SRE** | 145 | IaC, container security, Kubernetes, IAM, secrets management, disaster recovery (RTO/RPO), capacity planning, chaos engineering, incident management |

**Total: 780 items** — every one is a concrete, verifiable check. No fluff.

---

## Token Usage & Cost

The skill is designed to be token-efficient. Here's what it actually costs:

| Scenario | Tokens used | Approximate cost |
|----------|------------|-----------------|
| Single type (e.g., just a web app) | ~12,000 | ~$0.04 (Sonnet) / ~$0.18 (Opus) |
| Two types (e.g., web + API) | ~22,000 | ~$0.07 (Sonnet) / ~$0.33 (Opus) |
| Three types (e.g., web + API + payment) | ~30,000 | ~$0.10 (Sonnet) / ~$0.45 (Opus) |

**For context:** A typical coding conversation where you ask "fix this bug" uses 20,000-50,000 tokens. This skill uses about the same as reading one large source file.

### Why it's efficient

1. **Only loads what's needed** — if you have a web app, it only loads the web checklist (not all 6)
2. **Targeted scanning** — greps for specific patterns, doesn't read every file in your repo
3. **Single report** — one prompt in, one report out. No back-and-forth

### What the tokens go to

| Component | Tokens | Why |
|-----------|--------|-----|
| Skill instructions | ~1,400 | Tells the agent what to do |
| Checklist reference | ~3,000-5,000 | The actual items to check (per type) |
| Code scanning | ~4,000-7,000 | Reading your configs, grepping for patterns |
| Report output | ~3,000-5,000 | The detailed pass/fail report |

---

## How the Scan Works (technical details)

```
production-checklist/
├── SKILL.md              ← Instructions for the agent (what to scan, how to report)
└── references/
    ├── web.md            ← 144 items: web/frontend checks
    ├── mobile.md         ← 123 items: iOS & Android checks
    ├── api.md            ← 148 items: backend API checks
    ├── smart-contract.md ← 108 items: Solidity/EVM checks
    ├── payment.md        ← 112 items: payment & financial checks
    └── infrastructure.md ← 145 items: infra & SRE checks
```

1. **SKILL.md** is read by the agent — it contains the scanning instructions
2. The agent **globs your project** to detect what stack you're using
3. It **loads only the matching reference file(s)** — not all 6
4. It **scans your code** using grep/read/glob for each checklist item
5. It **produces one report** with every item marked ✅ PASS / ❌ FAIL / ⚠️ MANUAL
6. It **offers to fix** any failing items with actual code changes

Each reference file is independent — no cross-file dependencies.

---

## Standards & Sources

This checklist was compiled from industry-leading production readiness standards:

| Standard | What it covers | Version |
|----------|---------------|---------|
| [Google SRE Book](https://sre.google/sre-book/launch-checklist/) | Production readiness review, launch checklist | — |
| [AWS Well-Architected](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) | 6-pillar cloud architecture review | 2024 |
| [PCI DSS](https://www.pcisecuritystandards.org/standards/) | Payment card security | v4.0 |
| [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) | Application security verification | v5.0 |
| [OWASP Top 10](https://owasp.org/www-project-top-ten/) | Most critical web security risks | 2025 |
| [WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/) | Web accessibility guidelines | 2.2 |
| [Stripe PCI Guide](https://stripe.com/guides/pci-compliance) | Developer PCI compliance | — |

Last reviewed: April 2026.

---

## FAQ

### Who is this for?
Any developer or team shipping to production. The checks range from basics (is HTTPS on?) to FAANG-grade (are your SLOs defined? is your error budget tracked?). It's useful whether you're a solo dev or a platform team at a large company.

### Does it modify my code?
Not unless you ask it to. The scan is read-only. After the report, it offers to fix failing items — but only if you say yes.

### What if my project type isn't listed?
The 6 types cover the vast majority of production deployments. If you're deploying something else (ML model, data pipeline, desktop app), the skill will still check general items from the closest matching type. We're adding more types in future versions.

### Does it work with monorepos?
Yes. It scans from the project root and detects multiple types. A monorepo with a React frontend, Express API, and Terraform infra will get all three checklists.

### Can I customize the checklist?
Yes — each `references/*.md` file is plain markdown. Add, remove, or modify items to match your org's standards. The format is just `- [ ] Item description (fix hint)`.

---

## Contributing

PRs welcome! Each `references/*.md` file is self-contained.

1. **Add items:** Follow the format `- [ ] Item description (fix hint in parens)`
2. **New deployment type:** Create `references/your-type.md` with 🔴/🟡/🟢 tiers, add signals to `SKILL.md`
3. **Update existing:** Fix outdated references, add new best practices, improve fix hints

Severity tiers: 🔴 = causes real damage if missed. 🟡 = causes pain within a week. 🟢 = best practice.

---

## Roadmap

| Version | Feature |
|---------|---------|
| v2.2 | `nextjs.md` — Next.js-specific checks (ISR, edge runtime, middleware, App Router) |
| v2.3 | `supabase.md` — Supabase production hardening (RLS policies, backups, connection pooling) |
| v2.4 | `data-pipeline.md` — ML model and data pipeline deployment checks |
| v3.0 | Auto-remediation — agent detects and fixes common issues automatically |

---

## License

MIT
