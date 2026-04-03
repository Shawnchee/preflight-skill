# Changelog

All notable changes to this project will be documented in this file.

## [2.1.0] - 2026-04-03

### Fixed
- Consolidated 4 duplicate checklist items (3 in api.md, 1 in mobile.md)
- Fixed marketplace.json repository URL and author field
- Fixed plugin.json author field
- Scoped down settings.local.json to minimum required permissions
- Tightened API type detection signals to reduce false positives

### Added
- Edge case handling: no-type-detected fallback message
- Monorepo scoping support (scan subdirectories)
- Directory exclusion patterns (node_modules, vendor, .git, dist, build)
- Secret redaction in scan reports
- "Ready to Ship" report template for all-pass scenarios
- Report truncation for large scans (50+ items)
- Step 5 guardrails (show changes before applying, no auto-commit)
- .gitignore file
- CHANGELOG.md
- Tags and license fields in marketplace.json

### Changed
- Version bumped to 2.1.0
- Total checklist items: 784 → 780 (duplicates removed)
- API checklist: 151 → 148 items
- Mobile checklist: 124 → 123 items

## [2.0.0] - 2026-04-02

### Changed
- Complete redesign: auto-scan codebase instead of manual Q&A
- Slimmed SKILL.md for token efficiency (-40%)
- Audit pass: fixed counts, duplicates, outdated refs, formatting
- Rewrote README

## [1.0.0] - 2026-03-31

### Added
- Initial release with 750+ items across 6 deployment types
- Web, Mobile, API, Smart Contract, Payment, Infrastructure checklists
- Based on Google SRE, AWS Well-Architected, PCI DSS v4.0, OWASP, WCAG 2.2
