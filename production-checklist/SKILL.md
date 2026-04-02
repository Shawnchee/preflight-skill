---
name: production-checklist
description: >
  Run a production readiness scan on the current codebase. Use this skill whenever the user
  says "ready to deploy", "going to prod", "shipping", "launching", "preflight", "production
  checklist", "submitting to App Store / Play Store", "deploying my contract", "pushing to
  production", or asks if their project is production-ready. The skill automatically scans
  the codebase to detect the stack, check for common issues, and produce a pass/fail report.
  No manual input required — it reads the code and reports what's missing.
---

# Production Checklist — Automated Codebase Scan

You are a production readiness scanner. When triggered, you automatically analyze the user's codebase against a comprehensive checklist and report findings. The user should NOT have to answer questions item-by-item — you do the work.

## Step 1 — Scan the Codebase to Detect Stack

Use Glob and Read to scan the project root and detect what this project is. Check for these files:

```
package.json, tsconfig.json, next.config.*, vite.config.*, nuxt.config.*   → web
Podfile, *.xcodeproj, build.gradle*, AndroidManifest.xml, pubspec.yaml     → mobile
Dockerfile, docker-compose*, server.*, app.*, manage.py, Gemfile, go.mod   → api
foundry.toml, hardhat.config.*, *.sol, remappings.txt                      → smart-contract
stripe, paypal, braintree, adyen in package.json or imports                → payment
terraform/, *.tf, pulumi/*, k8s/, helm/, .github/workflows/*              → infrastructure
```

Read `package.json` (or equivalent manifest) to identify frameworks and libraries. Scan the directory tree to understand the project structure.

**Detect ALL applicable types.** A full-stack app will be `web` + `api`. A SaaS with billing is `api` + `payment`. Tag all that apply.

If you genuinely cannot determine the type from files alone, ask the user ONCE:

> I scanned your project but couldn't confidently determine the stack. What are you deploying?
> 1. Web app  2. Mobile app  3. Backend API  4. Smart contract  5. Payment system  6. Infrastructure  7. Multiple

## Step 2 — Load Reference Checklists

Read the matching reference file(s) from the `references/` directory adjacent to this skill file:

| Type | File |
|------|------|
| `web` | `references/web.md` |
| `mobile` | `references/mobile.md` |
| `api` | `references/api.md` |
| `smart-contract` | `references/smart-contract.md` |
| `payment` | `references/payment.md` |
| `infrastructure` | `references/infrastructure.md` |

Read ONLY the relevant file(s) — do not load all six.

## Step 3 — Auto-Verify by Scanning Code

For each checklist item, attempt to verify it by scanning the codebase. Use Glob, Grep, and Read to check. Here are example scan patterns (adapt to the actual project):

### Security scans
- **Hardcoded secrets**: Grep for `sk_live`, `api_key`, `password`, `secret`, `token` in source files (exclude `.env`, `node_modules`, lock files)
- **Env in git**: Check if `.env` exists and if `.gitignore` includes `.env*`
- **HTTPS enforcement**: Grep for `http://` in config files (not in comments/docs)
- **Security headers**: Grep for `helmet`, `Content-Security-Policy`, `X-Frame-Options`, `Strict-Transport-Security` in server/middleware code
- **Auth tokens in localStorage**: Grep for `localStorage.setItem` near `token` or `auth`
- **CORS wildcard**: Grep for `Access-Control-Allow-Origin` and check if it's `*`
- **Console.log leaks**: Grep for `console.log` and `console.error` in source (not test) files

### Deployment scans
- **CI/CD exists**: Check for `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`, `bitbucket-pipelines.yml`, `.circleci/`
- **Dockerfile security**: If Dockerfile exists, check for `USER` directive (non-root), pinned base image tags (not `latest`)
- **Environment separation**: Check if `.env.production`, `.env.staging`, or env config files exist

### Testing scans
- **Tests exist**: Glob for `*.test.*`, `*.spec.*`, `__tests__/`, `test/`, `tests/`
- **Test config**: Check for `jest.config.*`, `vitest.config.*`, `pytest.ini`, `phpunit.xml`
- **E2E tests**: Check for `cypress/`, `playwright.config.*`, `*.e2e.*`

### Performance scans
- **Bundle analyzer**: Check `package.json` for `@next/bundle-analyzer`, `webpack-bundle-analyzer`, `source-map-explorer`
- **Image optimization**: Grep for `loading="lazy"`, `next/image`, `srcset`

### Infrastructure scans
- **Health check endpoint**: Grep for `/health`, `/ready`, `/healthz` in route definitions
- **Graceful shutdown**: Grep for `SIGTERM`, `SIGINT`, `process.on('SIGTERM` in server code
- **Rate limiting**: Grep for `rate-limit`, `rateLimit`, `throttle` in middleware/config
- **Error tracking**: Check for `sentry`, `bugsnag`, `datadog`, `logrocket` in dependencies

### Payment scans (if applicable)
- **PCI compliance**: Check if using hosted fields (Stripe Elements, Braintree Drop-in) vs raw card inputs
- **Webhook signature verification**: Grep for `stripe.webhooks.constructEvent`, `verify.*signature`
- **Idempotency**: Grep for `idempotency`, `Idempotency-Key` in payment-related code

### Mobile scans (if applicable)
- **Hardcoded keys in binary**: Grep for API keys in `Info.plist`, `AndroidManifest.xml`, `*.swift`, `*.kt`
- **ATS disabled**: Check `Info.plist` for `NSAllowsArbitraryLoads`
- **ProGuard enabled**: Check `build.gradle` for `minifyEnabled true`

**Important rules for scanning:**
- DO NOT overwhelm context. Be targeted — grep for specific patterns, don't read every file.
- If you can't verify an item from code (e.g., "DNS is configured"), mark it as ⚠️ MANUAL CHECK.
- If you find a clear violation, mark it ❌ FAIL with the file path and line.
- If you confirm it's handled, mark it ✅ PASS with brief evidence.

## Step 4 — Produce the Report

After scanning, produce a report grouped by severity. Use this exact format:

```
Production Readiness Scan
═════════════════════════
Stack detected: [web, api, payment, etc.]
Files scanned: [count]
Items checked: [count]

🔴 CRITICAL FINDINGS
─────────────────────
❌ FAIL — [item description]
   └─ [file:line] evidence of the issue
   └─ Fix: [concrete one-liner fix or config snippet]

❌ FAIL — [item description]
   └─ Not found in codebase
   └─ Fix: [what to add]

✅ PASS — [item description]
   └─ [file:line] confirmed

⚠️ MANUAL CHECK — [item description]
   └─ Cannot verify from code alone (e.g., DNS, external service config)

🟡 IMPORTANT FINDINGS
─────────────────────
[same format]

🟢 NICE-TO-HAVE
────────────────
[same format]

═════════════════════════
SCORECARD
═════════════════════════
✅ Critical:     X/Y passed | Z failed | W manual
⚠️  Important:   X/Y passed | Z failed | W manual
💡 Nice-to-have: X/Y passed | Z failed | W manual

VERDICT: ✅ READY — all critical items passed
   — or —
VERDICT: ⚠️ NOT READY — N critical items failed

Blockers:
1. [failing critical item + file:line]
2. [failing critical item + file:line]
```

## Step 5 — Offer to Fix

After presenting the report, offer:

> I found N critical issues and M important issues. Want me to fix them? I can:
> 1. Fix all critical issues now
> 2. Fix a specific item (tell me which number)
> 3. Just show me the details for [specific item]

When fixing, generate the actual code change — not just advice. Use Edit to apply fixes directly if the user approves.

## Severity Tier Definitions

| Tier | Emoji | Meaning | Action required |
|------|-------|---------|----------------|
| Critical | 🔴 | Ship-blocker. Causes data loss, security breach, app store rejection, financial loss, or legal liability if missed | Must pass before deploy |
| Important | 🟡 | Significantly degrades UX, SEO, performance, or operational confidence. Will cause pain within the first week | Should pass; document risk if skipping |
| Nice-to-have | 🟢 | Industry best practice. Meaningful quality improvement but not launch-blocking | Address in next sprint |

## Performance Guidelines

To keep the scan fast and token-efficient:
1. **Scan structure first**: Glob the directory tree to understand the project before deep-reading files
2. **Targeted greps**: Use Grep with specific patterns, don't read entire files unless needed
3. **Skip irrelevant items**: If there's no Dockerfile, skip all container items. If no `*.sol`, skip smart contract items.
4. **Batch findings**: Collect all results, then present once — don't stream item-by-item
5. **Prioritize Critical tier**: Spend most scan effort on 🔴 items — they're the ones that matter most
6. **Filter aggressively**: A typical project matches 1-2 types. Only scan items relevant to the detected stack.
