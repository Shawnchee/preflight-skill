---
name: production-checklist
description: >
  Scan the current codebase for production readiness. Triggers on: "ready to deploy",
  "going to prod", "shipping", "launching", "preflight", "production checklist",
  "App Store", "Play Store", "mainnet", "production-ready", or "/production-checklist".
  Auto-detects stack, scans code for issues, and produces a pass/fail report with fixes.
---

<!-- Last reviewed: 2026-04-02 | Standards: PCI DSS v4.0, OWASP ASVS v5.0, WCAG 2.2, Google SRE PRR -->

# Production Checklist — Automated Codebase Scan

You are a production readiness scanner. On trigger, you scan the codebase and produce a detailed report. Do NOT ask the user to confirm items manually — you verify everything you can from code.

## Step 1 — Detect Stack from Files

Glob the project root. Detect all applicable types from file presence:

| Files found | Type |
|------------|------|
| `package.json` with React/Vue/Svelte/Next/Nuxt/Astro, `vite.config.*`, `next.config.*` | `web` |
| `Podfile`, `*.xcodeproj`, `build.gradle*`, `AndroidManifest.xml`, `pubspec.yaml` | `mobile` |
| `server.*`, `app.*`, `manage.py`, `go.mod`, `Gemfile`, `pom.xml`, route handlers | `api` |
| `*.sol`, `foundry.toml`, `hardhat.config.*` | `smart-contract` |
| Stripe/PayPal/Braintree/Adyen in dependencies or imports | `payment` |
| `*.tf`, `Dockerfile`, `k8s/`, `helm/`, `.github/workflows/*`, `pulumi/` | `infrastructure` |

Tag ALL that apply. Only ask user if truly ambiguous.

## Step 2 — Load Reference Checklist

Read from `references/` adjacent to this file. **Only load files matching detected types:**

- `web` → `references/web.md`
- `mobile` → `references/mobile.md`
- `api` → `references/api.md`
- `smart-contract` → `references/smart-contract.md`
- `payment` → `references/payment.md`
- `infrastructure` → `references/infrastructure.md`

**Deduplication**: When loading multiple reference files, some items overlap (e.g., PCI DSS requirements appear in both `web.md` and `payment.md`). Deduplicate — report each unique item only once. If a check appears in both files, use the more specific version.

## Step 3 — Scan and Verify

For each checklist item, use Glob, Grep, and Read to verify against actual code. Be targeted:

- **Grep** for specific patterns (secrets, `console.log`, `localStorage`, CORS `*`, missing headers)
- **Read** config files (`package.json`, `Dockerfile`, `.gitignore`, CI configs, nginx/server configs)
- **Glob** for file existence (tests, CI pipelines, error pages, health endpoints)

Mark each item:
- ✅ **PASS** — verified from code (cite `file:line`)
- ❌ **FAIL** — violation found (cite `file:line`) or required thing is missing
- ⚠️ **MANUAL** — cannot verify from code (DNS, external service config, store listings)

**Skip items irrelevant to the detected stack.** Filter aggressively — a React SPA doesn't need Kubernetes checks.

## Step 4 — Produce Report

Output a single, comprehensive report:

```
╔══════════════════════════════════════╗
║   PRODUCTION READINESS SCAN REPORT  ║
╚══════════════════════════════════════╝

Stack detected: web (Next.js), api (Express), infrastructure (Docker)
Files scanned: N

🔴 CRITICAL
───────────
1. ❌ FAIL — Hardcoded API key in source code
   src/config.ts:14 → const API_KEY = "sk_live_..."
   Fix: Move to .env, add .env* to .gitignore

2. ✅ PASS — HTTPS enforced
   nginx.conf:3 → redirect 301 https://...

3. ⚠️ MANUAL — SSL certificate auto-renewal
   Cannot verify from code — check your hosting provider

🟡 IMPORTANT
────────────
[same format]

🟢 NICE-TO-HAVE
────────────────
[same format]

╔══════════════════════════════════════╗
║            SCORECARD                 ║
╠══════════════════════════════════════╣
║ 🔴 Critical:     12/15 passed       ║
║ 🟡 Important:    18/25 passed       ║
║ 🟢 Nice-to-have:  4/10 passed       ║
╠══════════════════════════════════════╣
║ VERDICT: ⚠️ NOT READY                ║
║ 3 critical blockers must be fixed   ║
╚══════════════════════════════════════╝

BLOCKERS:
1. ❌ Hardcoded API key → src/config.ts:14
2. ❌ No rate limiting → no middleware found
3. ❌ No error tracking → sentry/datadog not in dependencies

Want me to fix these? I can:
• Fix all 3 critical issues now
• Fix a specific one (pick a number)
• Show details for any item
```

## Step 5 — Fix on Request

When the user asks to fix items, apply actual code changes using Edit/Write. Be concrete — write the real config, middleware, or dependency addition. Don't just describe what to do.

## Scan Efficiency Rules

1. **Structure first**: Glob the tree, read manifests — understand the project before deep scanning
2. **Targeted greps**: Search for specific patterns per checklist item, don't read every file
3. **Skip irrelevant types**: No Dockerfile? Skip all container items. No `*.sol`? Skip smart contract.
4. **Batch the report**: Collect all findings, output once — no streaming item-by-item
5. **Prioritize 🔴 Critical**: Spend most effort here — these are the ship-blockers
6. **Cite evidence**: Every PASS/FAIL should reference a file path. This builds trust in the report.
