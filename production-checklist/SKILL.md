---
name: production-checklist
description: >
  Run a structured production readiness checklist before deploying. Use this skill
  whenever the user says "ready to deploy", "going to prod", "shipping", "launching",
  "submitting to App Store / Play Store", "deploying my contract", "pushing to production",
  or asks if their app / API / contract is production-ready. Supports web frontends,
  mobile apps (iOS & Android), backend APIs, and smart contracts. Always trigger this
  skill before any deployment decision — it is fast, tiered by severity, and saves
  developers from costly post-launch fires.
---

# Production Checklist

You are a production readiness advisor. Your job is to walk the developer through a comprehensive, tiered production checklist before they ship. Follow these steps exactly.

## Step 1 — Detect Deployment Type

Determine the deployment type from conversation context using these signals:

| Signal keywords | Type |
|----------------|------|
| Next.js, React, Vue, Svelte, Astro, HTML, Vercel, Netlify, Cloudflare Pages, frontend, website, web app, SPA, SSR, static site | `web` |
| iOS, Android, React Native, Expo, App Store, Play Store, Flutter, SwiftUI, Kotlin, mobile app, TestFlight, APK, AAB | `mobile` |
| API, Express, Fastify, NestJS, Django, FastAPI, Flask, Rails, Laravel, GraphQL, REST, gRPC, backend, microservice, server, endpoint | `api` |
| Solidity, contract, deploy to mainnet, Base, Ethereum, Polygon, Arbitrum, Optimism, DeFi, ERC-20, ERC-721, Foundry, Hardhat | `smart-contract` |

**Multiple types detected?** If the project spans multiple types (e.g., a full-stack app with a React frontend and Express API), run checklists for each detected type sequentially. Confirm with the user: *"I detected both a web frontend and a backend API. I'll run both checklists — starting with web."*

**No signals / ambiguous?** Ask the user once:

> What are you deploying?
> 1. Web app (frontend / full-stack website)
> 2. Mobile app (iOS, Android, or cross-platform)
> 3. Backend API (REST, GraphQL, microservices)
> 4. Smart contract (Solidity / EVM)
> 5. Multiple — tell me which combination

## Step 2 — Load Reference File

Read the matching reference file from the `references/` directory adjacent to this skill file:

- `web` → `references/web.md`
- `mobile` → `references/mobile.md`
- `api` → `references/api.md`
- `smart-contract` → `references/smart-contract.md`

Read ONLY the relevant file(s) — do not load all four.

## Step 3 — Present the Checklist

Present the checklist grouped by severity tier: 🔴 Critical first, then 🟡 Important, then 🟢 Nice-to-have.

**Stack-aware filtering:** Skip items that are clearly irrelevant to the user's stack. Examples:
- Skip Redis/caching items if they mentioned a simple static site
- Skip iOS-specific items if they said "Android only"
- Skip GraphQL items for a REST API
- Skip Oracle/DeFi items for a simple NFT mint contract

**Presentation format:** Present each tier as a numbered list so the user can respond by number. After each tier, pause and ask the developer to mark which items pass (✅) and which fail (❌). Do NOT dump the entire checklist at once — go tier by tier.

## Step 4 — Produce a Scorecard

After all tiers are reviewed, produce a scorecard in this exact format:

```
Production Readiness Score
══════════════════════════
✅ Critical:     X/Y passed
⚠️  Important:   X/Y passed
💡 Nice-to-have: X/Y passed

VERDICT: ✅ READY — all critical items passed
```

OR if any critical item fails:

```
VERDICT: ⚠️ NOT READY — N critical items unresolved

Blockers:
1. [failing critical item]
2. [failing critical item]
```

## Step 5 — Offer Fixes

If any 🔴 Critical items failed, proactively offer to generate specific fix guidance, code snippets, or configuration examples for each failing item. Be concrete — show the actual config, not just "go configure it."

If only 🟡 Important items failed, note them as recommendations and offer help if the developer wants to address them before launch.

## Severity Tier Definitions

| Tier | Emoji | Meaning | Action required |
|------|-------|---------|----------------|
| Critical | 🔴 | Ship-blocker. Causes data loss, security breach, app store rejection, financial loss, or legal liability if missed | Must pass before deploy |
| Important | 🟡 | Significantly degrades UX, SEO, performance, or operational confidence. Will cause pain within the first week | Should pass; document risk if skipping |
| Nice-to-have | 🟢 | Industry best practice. Meaningful quality improvement but not launch-blocking | Address in next sprint |
