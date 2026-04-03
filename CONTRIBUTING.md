# Contributing

Thanks for helping improve the production checklist! This project is open to contributions of all sizes.

## Quick Start

1. Fork the repo
2. Make your changes
3. Submit a PR

## What You Can Contribute

### Add checklist items

Each `production-checklist/references/*.md` file is self-contained. To add an item:

```markdown
- [ ] Your checklist item description (fix hint in parentheses)
```

Place it under the correct severity tier:
- **🔴 CRITICAL** — causes real damage if missed (data loss, security breach, downtime)
- **🟡 IMPORTANT** — causes pain within a week (performance issues, missing monitoring, compliance gaps)
- **🟢 NICE-TO-HAVE** — best practice, polish (developer experience, optimization, future-proofing)

**Good items are:**
- Verifiable from code (can be checked with grep/glob/read)
- Specific (cite a standard, tool, or concrete threshold)
- Actionable (include a fix hint in parentheses)

**Example:**
```markdown
- [ ] Rate limiting enabled on login endpoint — max 5 attempts per minute per IP (use express-rate-limit or equivalent)
```

### Add a new deployment type

1. Create `production-checklist/references/your-type.md`
2. Follow the existing format: `🔴 CRITICAL` → `🟡 IMPORTANT` → `🟢 NICE-TO-HAVE` sections
3. Add detection signals to `production-checklist/SKILL.md` Step 1 table
4. Update item counts in `README.md`

### Fix or improve existing items

- Update outdated version references
- Improve fix hints to be more actionable
- Fix incorrect categorization (wrong severity tier)
- Remove items that are no longer relevant

## Guidelines

- **One PR per concern** — don't mix new items with formatting fixes
- **Keep items concise** — one line per item, fix hint in parentheses
- **No duplicates** — check other reference files for overlap before adding
- **Cite standards** — reference OWASP, PCI DSS, WCAG, etc. where applicable
- **Test your wording** — would an AI agent know how to verify this from code?

## File Structure

```
production-checklist/
├── SKILL.md              ← Agent instructions (be careful editing this)
└── references/
    ├── web.md            ← 144 items
    ├── mobile.md         ← 123 items
    ├── api.md            ← 148 items
    ├── smart-contract.md ← 108 items
    ├── payment.md        ← 112 items
    └── infrastructure.md ← 145 items
```

## Questions?

Open an issue on [GitHub](https://github.com/Shawnchee/preflight-skill/issues).
