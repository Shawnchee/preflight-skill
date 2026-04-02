# Web / Frontend Production Checklist

---

## 🔴 CRITICAL (ship-blockers)

### Security

- [ ] All environment variables are in `.env` and NOT committed to Git (check `.gitignore` includes `.env*`)
- [ ] HTTPS is enforced on all routes — no mixed content warnings (set up automatic HTTP → HTTPS redirect)
- [ ] Content Security Policy (CSP) headers configured (start with `Content-Security-Policy: default-src 'self'` and expand)
- [ ] Security headers set: `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- [ ] No hardcoded API keys, secrets, or tokens in client-side code (search codebase for `sk_`, `api_key`, `secret`, `password`)
- [ ] Authentication tokens stored in `httpOnly` secure cookies — NOT localStorage or sessionStorage (XSS can steal localStorage)
- [ ] CORS restricted to known origins — never `Access-Control-Allow-Origin: *` in production (set explicit domain allowlist)
- [ ] Rate limiting enabled on all public-facing form submissions and API routes (use middleware like `express-rate-limit` or Cloudflare rules)
- [ ] SSL/TLS certificate is valid and auto-renews (check expiry date; use Let's Encrypt or your host's managed certs)
- [ ] No sensitive data exposed in URL query parameters (tokens, emails, passwords must go in request body or headers)
- [ ] Subresource Integrity (SRI) hashes set on third-party `<script>` and `<link>` tags (add `integrity` attribute)
- [ ] `Referrer-Policy` header set to `strict-origin-when-cross-origin` or stricter

### Functionality

- [ ] All critical user flows tested end-to-end on the actual production build (signup, login, checkout, payment, password reset)
- [ ] Custom 404 page exists, is on-brand, and includes navigation back to the app
- [ ] Custom 500 / error page exists — users never see a raw stack trace or framework error page
- [ ] No `console.log`, `console.error`, or `debugger` statements leaking sensitive data in production build (use build-time stripping)
- [ ] All external API calls have error handling, loading states, and user-friendly fallback UI
- [ ] Forms have both client-side and server-side validation (never trust client-only validation)
- [ ] Empty states, zero-data states, and loading skeletons implemented for all dynamic content
- [ ] User-facing error messages are helpful and non-technical (no "undefined is not a function")

### Deployment

- [ ] CI/CD pipeline runs and passes all tests on the production branch before deploy
- [ ] Rollback procedure is documented and has been tested at least once (know how to revert within 5 minutes)
- [ ] Production environment variables are set and differ from staging/dev (especially API URLs, database strings, feature flags)
- [ ] DNS is configured and propagated — domain points to the correct production server/CDN (verify with `dig` or DNS checker)
- [ ] Source maps are uploaded to error tracking service but NOT served publicly (prevents reverse-engineering)

---

## 🟡 IMPORTANT (should fix before launch)

### Performance

- [ ] Lighthouse score ≥ 90 on Performance, Accessibility, and Best Practices (run on production URL, not localhost)
- [ ] Core Web Vitals pass: LCP < 2.5s, INP < 200ms, CLS < 0.1 (test with PageSpeed Insights on real URL)
- [ ] Images use next-gen formats (WebP or AVIF) with fallbacks and are lazy-loaded (`loading="lazy"`)
- [ ] Images are properly sized — no serving 4000px images in 400px containers (use `srcset` and `sizes`)
- [ ] JS and CSS are minified, tree-shaken, and code-split (verify no unused code ships to the client)
- [ ] Static assets served via CDN with cache headers (`Cache-Control: public, max-age=31536000, immutable` for hashed assets)
- [ ] Fonts are subset to used characters and loaded with `font-display: swap` (or `optional` to avoid layout shift)
- [ ] No render-blocking resources in `<head>` — defer non-critical JS, inline critical CSS
- [ ] Bundle size analyzed and within budget — no single route JS chunk > 200KB gzipped (use `source-map-explorer` or `@next/bundle-analyzer`)
- [ ] Server-side or edge caching configured for repeated data queries (Redis, Vercel KV, Cloudflare KV, or framework cache)
- [ ] Third-party scripts audited for size and load impact — defer or lazy-load analytics, chat widgets, etc.
- [ ] `<link rel="preload">` used for critical above-the-fold assets (hero image, primary font)

### SEO & Metadata

- [ ] Unique `<title>` and `<meta name="description">` on every page (title 30–60 chars, description 120–155 chars)
- [ ] Open Graph tags set: `og:title`, `og:description`, `og:image` (1200×630px), `og:url`, `og:type`
- [ ] Twitter Card tags set: `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- [ ] `sitemap.xml` generated and submitted to Google Search Console (auto-generate on build for dynamic sites)
- [ ] `robots.txt` configured — not accidentally blocking production (check `Disallow` rules)
- [ ] Canonical URLs set with `<link rel="canonical">` on every page to prevent duplicate content indexing
- [ ] Structured data (JSON-LD) on key pages — at minimum: Organization, WebSite, BreadcrumbList (validate with Google Rich Results Test)
- [ ] All pages are server-rendered or pre-rendered for SEO-critical content (SPAs need SSR/SSG for crawlability)
- [ ] `<html lang="...">` attribute set correctly for primary language

### Accessibility

- [ ] All images have meaningful `alt` text (decorative images use `alt=""`)
- [ ] Color contrast meets WCAG AA: 4.5:1 for normal text, 3:1 for large text (check with browser DevTools or Stark)
- [ ] All interactive elements are keyboard-navigable — test full flows using only Tab, Enter, Escape, Arrow keys
- [ ] Focus indicators are visible on all interactive elements (never set `outline: none` without a replacement)
- [ ] ARIA labels on icon-only buttons, form inputs, and non-semantic interactive elements
- [ ] Heading hierarchy is logical — one `<h1>` per page, no skipped levels (`<h1>` → `<h3>` without `<h2>`)
- [ ] Form fields have associated `<label>` elements (not just placeholder text)
- [ ] Tested with at least one screen reader (VoiceOver on Mac, NVDA on Windows) on critical flows
- [ ] Skip navigation link provided for keyboard users (`Skip to main content`)
- [ ] Animations respect `prefers-reduced-motion` media query

### Monitoring & Observability

- [ ] Client-side error tracking configured and verified — errors flow to dashboard (Sentry, Datadog RUM, LogRocket)
- [ ] Analytics installed and tracking key events: page views, signups, conversions (GA4, PostHog, Plausible, Mixpanel)
- [ ] Uptime monitoring set up — alerts on downtime (Better Uptime, Freshping, UptimeRobot, Checkly)
- [ ] Real User Monitoring (RUM) enabled to track actual user Core Web Vitals (Vercel Analytics, Datadog RUM, SpeedCurve)
- [ ] Alerts configured for error rate spikes (e.g., > 5% error rate triggers PagerDuty/Slack notification)

### Cross-Browser & Device Testing

- [ ] Tested on latest Chrome, Firefox, Safari, and Edge (cover > 95% of users)
- [ ] Tested on mobile browsers: iOS Safari and Android Chrome at minimum
- [ ] Responsive design verified at common breakpoints: 375px, 768px, 1024px, 1440px
- [ ] Touch interactions work correctly on mobile (tap targets ≥ 44×44px, no hover-only interactions)

### Legal & Compliance

- [ ] Privacy policy page exists and is accessible from every page (footer link)
- [ ] Cookie consent banner implemented if serving EU/UK users (GDPR) or other regulated regions
- [ ] Terms of service page exists if users create accounts or transact
- [ ] Data processing agreements in place with third-party services that handle user data

---

## 🟢 NICE-TO-HAVE (polish)

- [ ] Favicon set: `favicon.ico` + `apple-touch-icon.png` (180×180) + web manifest icons (use realfavicongenerator.net)
- [ ] PWA manifest configured with `name`, `short_name`, `icons`, `theme_color`, `start_url`
- [ ] Service worker for offline support or asset caching (if PWA)
- [ ] Print stylesheet provided for content-heavy pages (`@media print`)
- [ ] Broken links checked and fixed (use W3C link checker, Screaming Frog, or `linkinator`)
- [ ] `rel="preconnect"` for critical third-party origins (fonts.googleapis.com, CDN, analytics)
- [ ] HTTP/2 or HTTP/3 enabled on the server (most CDNs/hosts do this by default — verify)
- [ ] HSTS preload submitted at hstspreload.org (after confirming HSTS header works)
- [ ] Dependency audit run — no known vulnerabilities (`npm audit`, `pnpm audit`, or Snyk)
- [ ] Adblocker compatibility tested — critical features not broken by common adblockers
- [ ] 404 page includes search or suggested links to reduce bounce rate
- [ ] Proper `Cache-Control` for HTML pages (`no-cache` or short TTL to ensure fresh deploys are picked up)
- [ ] Social share preview tested with actual URLs (use opengraph.xyz or Twitter Card Validator)
- [ ] RSS feed available for blog/content sites
- [ ] Web app handles deep links / direct URL access correctly (no blank pages on refresh for SPAs)
