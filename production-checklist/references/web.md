# Web / Frontend Production Checklist

---

## 🔴 CRITICAL (ship-blockers)

### Security

- [ ] All environment variables are in `.env` and NOT committed to Git (check `.gitignore` includes `.env*`)
- [ ] HTTPS is enforced on all routes — no mixed content warnings (set up automatic HTTP → HTTPS redirect)
- [ ] Content Security Policy (CSP) headers configured (start with `Content-Security-Policy: default-src 'self'` and expand per resource)
- [ ] Security headers set: `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- [ ] No hardcoded API keys, secrets, or tokens in client-side code (search codebase for `sk_`, `api_key`, `secret`, `password`)
- [ ] Authentication tokens stored in `httpOnly` secure cookies — NOT localStorage or sessionStorage (XSS can steal localStorage)
- [ ] CORS restricted to known origins — never `Access-Control-Allow-Origin: *` in production (set explicit domain allowlist)
- [ ] Rate limiting enabled on all public-facing form submissions and API routes (use middleware like `express-rate-limit` or Cloudflare rules)
- [ ] SSL/TLS certificate is valid and auto-renews (check expiry date; use Let's Encrypt or your host's managed certs)
- [ ] No sensitive data exposed in URL query parameters (tokens, emails, passwords must go in request body or headers)
- [ ] Subresource Integrity (SRI) hashes set on third-party `<script>` and `<link>` tags (add `integrity` attribute)
- [ ] `Referrer-Policy` header set to `strict-origin-when-cross-origin` or stricter
- [ ] `Permissions-Policy` header configured to disable unused browser features (`camera`, `microphone`, `geolocation`, `payment` — deny what you don't need)
- [ ] All user-generated content sanitized before rendering — use DOMPurify or framework-native sanitization to prevent stored XSS
- [ ] No `eval()`, `innerHTML`, or `document.write()` with user-controlled data (use `textContent` or framework bindings)
- [ ] CSP `script-src` does NOT include `'unsafe-inline'` or `'unsafe-eval'` in production (use nonces or hashes instead)
- [ ] All third-party scripts inventoried and authorized — PCI DSS 4.0 requirement 6.4.3 for payment pages (maintain a script inventory with integrity checks)
- [ ] Payment page scripts monitored for unauthorized changes — PCI DSS 4.0 requirement 11.6.1 (use CSP reporting or a client-side security tool)
- [ ] Session fixation prevented — regenerate session ID on login and privilege escalation
- [ ] Clickjacking protection verified — `X-Frame-Options: DENY` and CSP `frame-ancestors 'none'` both set

### Functionality

- [ ] All critical user flows tested end-to-end on the actual production build (signup, login, checkout, payment, password reset)
- [ ] Custom 404 page exists, is on-brand, and includes navigation back to the app
- [ ] Custom 500 / error page exists — users never see a raw stack trace or framework error page
- [ ] No `console.log`, `console.error`, or `debugger` statements leaking sensitive data in production build (use build-time stripping)
- [ ] All external API calls have error handling, loading states, and user-friendly fallback UI
- [ ] Forms have both client-side and server-side validation (never trust client-only validation)
- [ ] Empty states, zero-data states, and loading skeletons implemented for all dynamic content
- [ ] User-facing error messages are helpful and non-technical (no "undefined is not a function")
- [ ] Logout functionality works correctly — clears all session data, tokens, and cached sensitive content
- [ ] Session timeout implemented for idle users — redirect to login with a clear message after inactivity threshold
- [ ] File upload inputs validate file type, size, and content on both client and server (never trust client-only checks)
- [ ] All redirects validated — no open redirect vulnerabilities (whitelist allowed redirect destinations)

### Deployment

- [ ] CI/CD pipeline runs and passes all tests on the production branch before deploy
- [ ] Rollback procedure is documented and has been tested at least once (know how to revert within 5 minutes)
- [ ] Production environment variables are set and differ from staging/dev (especially API URLs, database strings, feature flags)
- [ ] DNS is configured and propagated — domain points to the correct production server/CDN (verify with `dig` or DNS checker)
- [ ] Source maps are uploaded to error tracking service but NOT served publicly (prevents reverse-engineering)
- [ ] Build artifacts are immutable and versioned — same build artifact deploys to staging and production (no "build in prod")
- [ ] Zero-downtime deployment configured — users never see a maintenance page during routine deploys (blue-green, rolling, or atomic deploys)
- [ ] Health check endpoint exists and is monitored — load balancer only routes to healthy instances

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
- [ ] Long tasks broken up — no single JS task blocks the main thread for > 50ms (use `requestIdleCallback`, Web Workers, or chunking)
- [ ] Service Worker or edge caching provides stale-while-revalidate for non-critical data
- [ ] Database queries behind the frontend are paginated — no endpoints return unbounded result sets
- [ ] Memory leaks checked — no growing heap from event listeners, intervals, or subscriptions left open on unmount

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
- [ ] Pagination pages use `rel="next"` / `rel="prev"` and canonical URLs correctly
- [ ] 301 redirects configured for all old URLs if migrating from a previous site (preserve SEO equity)

### Accessibility

- [ ] All images have meaningful `alt` text (decorative images use `alt=""`)
- [ ] Color contrast meets WCAG 2.2 AA: 4.5:1 for normal text, 3:1 for large text (check with browser DevTools or Stark)
- [ ] All interactive elements are keyboard-navigable — test full flows using only Tab, Enter, Escape, Arrow keys
- [ ] Focus indicators are visible on all interactive elements (never set `outline: none` without a replacement)
- [ ] ARIA labels on icon-only buttons, form inputs, and non-semantic interactive elements
- [ ] Heading hierarchy is logical — one `<h1>` per page, no skipped levels (`<h1>` → `<h3>` without `<h2>`)
- [ ] Form fields have associated `<label>` elements (not just placeholder text)
- [ ] Tested with at least one screen reader (VoiceOver on Mac, NVDA on Windows) on critical flows
- [ ] Skip navigation link provided for keyboard users (`Skip to main content`)
- [ ] Animations respect `prefers-reduced-motion` media query
- [ ] Touch targets ≥ 44×44px on mobile — WCAG 2.2 Target Size criterion (2.5.8)
- [ ] Error messages programmatically associated with form fields using `aria-describedby` or `aria-errormessage`
- [ ] Dynamic content updates announced to screen readers via `aria-live` regions (toasts, notifications, form errors)
- [ ] Drag-and-drop interactions have keyboard-accessible alternatives
- [ ] Color is not the only visual indicator for status, errors, or required fields (add icons, text, or patterns)

### Internationalization (i18n) & Localization

- [ ] All user-facing strings externalized into resource files — no hardcoded text in components (use `react-intl`, `next-intl`, `i18next`, etc.)
- [ ] Date, time, number, and currency formatting uses `Intl` API or locale-aware library (never hardcode `MM/DD/YYYY`)
- [ ] Right-to-left (RTL) layout support implemented if serving Arabic, Hebrew, or other RTL locales (use CSS `direction: rtl` and logical properties)
- [ ] Text expansion accounted for — German and French can be 30-40% longer than English (no fixed-width containers that clip translated text)
- [ ] Locale detected from user preference (Accept-Language header, browser setting, or user profile) — not from IP geolocation alone
- [ ] Unicode properly supported — emojis, CJK characters, and diacritics render correctly in all text fields and databases

### Monitoring & Observability

- [ ] Client-side error tracking configured and verified — errors flow to dashboard (Sentry, Datadog RUM, LogRocket)
- [ ] Analytics installed and tracking key events: page views, signups, conversions (GA4, PostHog, Plausible, Mixpanel)
- [ ] Uptime monitoring set up — alerts on downtime (Better Uptime, Freshping, UptimeRobot, Checkly)
- [ ] Real User Monitoring (RUM) enabled to track actual user Core Web Vitals (Vercel Analytics, Datadog RUM, SpeedCurve)
- [ ] Alerts configured for error rate spikes (e.g., > 5% error rate triggers PagerDuty/Slack notification)
- [ ] CSP violation reporting enabled — `report-uri` or `report-to` directive sends violations to a monitoring endpoint
- [ ] Client-side performance budgets enforced in CI — fail the build if bundle size or LCP regresses beyond threshold
- [ ] User session replay available for debugging production issues (LogRocket, FullStory, PostHog — ensure PII is masked)

### Cross-Browser & Device Testing

- [ ] Tested on latest Chrome, Firefox, Safari, and Edge (cover > 95% of users)
- [ ] Tested on mobile browsers: iOS Safari and Android Chrome at minimum
- [ ] Responsive design verified at common breakpoints: 375px, 768px, 1024px, 1440px
- [ ] Touch interactions work correctly on mobile (tap targets ≥ 44×44px, no hover-only interactions)
- [ ] Tested with browser zoom at 200% — content remains usable and readable (WCAG 1.4.4)
- [ ] Tested with browser text scaling at 200% — no text clipping or overflow (WCAG 1.4.4)

### Legal & Compliance

- [ ] Privacy policy page exists and is accessible from every page (footer link)
- [ ] Cookie consent banner implemented if serving EU/UK users (GDPR) or other regulated regions — blocks non-essential cookies until consent
- [ ] Terms of service page exists if users create accounts or transact
- [ ] Data processing agreements in place with third-party services that handle user data
- [ ] CCPA "Do Not Sell or Share My Personal Information" link visible if serving California users
- [ ] Data subject access request (DSAR) flow exists — users can request export or deletion of their personal data
- [ ] Age verification or COPPA compliance implemented if app could attract users under 13
- [ ] Accessibility statement published if required by jurisdiction (ADA, EAA European Accessibility Act 2025)
- [ ] Cookie policy details all cookies, their purpose, duration, and third-party origins

### Payment Frontend (if applicable)

- [ ] Payment forms use PCI-compliant hosted fields or iframes (Stripe Elements, Braintree Drop-in, Adyen Web Components) — never collect raw card numbers in your own forms
- [ ] Payment page served over HTTPS with valid TLS 1.2+ — no mixed content allowed on checkout pages
- [ ] All scripts on payment pages inventoried and integrity-verified per PCI DSS 4.0 requirement 6.4.3
- [ ] Payment confirmation page shows transaction ID, amount, and clear success/failure status
- [ ] Double-submit prevention on payment buttons — disable button after click, use idempotency keys
- [ ] Failed payment UX is clear — show specific error messages (card declined, insufficient funds, expired card) not generic errors
- [ ] Saved payment methods display masked card numbers only (last 4 digits) — never show full card number
- [ ] Payment form autofill works correctly with browser autocomplete attributes (`cc-name`, `cc-number`, `cc-exp`, `cc-csc`)
- [ ] 3D Secure / Strong Customer Authentication (SCA) flow implemented for EU payments per PSD2 regulation

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
- [ ] Feature flags integrated for safe rollouts and instant kill switches (LaunchDarkly, Flagsmith, Unleash, or config-based)
- [ ] A/B testing infrastructure in place for conversion-critical pages (Optimizely, PostHog, GrowthBook)
- [ ] `prefers-color-scheme` media query supported for dark mode (or manual toggle provided)
- [ ] `prefers-contrast` media query respected for high-contrast mode users
- [ ] Stale content detection — cache-busting strategy ensures users never see outdated JS/CSS after deploy
- [ ] DNS prefetch (`<link rel="dns-prefetch">`) for all third-party domains referenced on the page
- [ ] Web Vitals tracking in CI — automated Lighthouse or Web Vitals regression testing on every PR
- [ ] Content Delivery Network configured with geographic distribution matching user base (multi-region PoPs)
- [ ] Edge functions / middleware used for geo-routing, A/B testing, or bot detection at the CDN layer
