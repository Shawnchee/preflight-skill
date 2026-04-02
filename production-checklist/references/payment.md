# Payment & Financial System Production Checklist

---

## 🔴 CRITICAL (ship-blockers)

### PCI DSS Compliance

- [ ] PCI DSS scope determined — identify all systems that store, process, or transmit cardholder data (minimize scope by using tokenization)
- [ ] Cardholder data NEVER touches your servers — use PCI-compliant hosted payment fields (Stripe Elements, Braintree Drop-in, Adyen Components)
- [ ] If handling card data directly: PCI DSS Level 1 assessment completed by a Qualified Security Assessor (QSA)
- [ ] PCI DSS v4.0 requirement 6.4.3 satisfied: complete inventory of all scripts on payment pages with documented authorization for each
- [ ] PCI DSS v4.0 requirement 11.6.1 satisfied: real-time detection and alerting on unauthorized changes to payment page elements (HTML, scripts, iframes)
- [ ] SAQ (Self-Assessment Questionnaire) completed and filed with your acquiring bank — type depends on integration method (SAQ-A for hosted, SAQ-A-EP for iframes)
- [ ] Cardholder data not stored after authorization — no full PAN, CVV, or magnetic stripe data retained in any system (database, logs, files, backups)
- [ ] All cardholder data transmission encrypted with TLS 1.2+ — no fallback to older protocols
- [ ] Tokenization implemented: store payment method tokens from your payment processor, never raw card numbers
- [ ] Payment page served exclusively over HTTPS — no mixed content, no HTTP fallback
- [ ] Quarterly vulnerability scans by an Approved Scanning Vendor (ASV) passing with zero critical/high findings
- [ ] Annual penetration test completed on payment infrastructure (internal and external network segments)

### Transaction Integrity

- [ ] Idempotency keys required on ALL payment mutation endpoints — duplicate requests must not cause duplicate charges
- [ ] Double-entry bookkeeping implemented: every financial operation creates balanced debit and credit entries (sum of all debits = sum of all credits)
- [ ] Ledger is append-only — corrections are recorded as new reversal entries, never as mutations of existing records
- [ ] All financial operations wrapped in database transactions with serializable or repeatable-read isolation level
- [ ] Amount and currency always stored and transmitted together — a bare numeric value without currency context is never valid
- [ ] Monetary amounts stored as integers in smallest unit (cents, paise) — never floating-point (no `float`/`double` for money)
- [ ] Transaction state machine is well-defined with explicit states: `pending` → `processing` → `succeeded`/`failed` — no ambiguous intermediate states
- [ ] Every transaction has a unique, immutable transaction ID generated server-side (UUID v7 or similar)
- [ ] Optimistic locking or row-level locking prevents race conditions on balance updates (no lost updates from concurrent requests)
- [ ] Partial failures handled: if payment succeeds but order creation fails, compensation logic triggers refund or retries order creation

### Fraud Prevention

- [ ] Velocity checks implemented: limit transactions per card, per IP, per user within time windows (e.g., max 5 transactions per card per hour)
- [ ] Address Verification System (AVS) enabled for card-not-present transactions — flag mismatches
- [ ] CVV/CVC verification required for all card-not-present transactions
- [ ] 3D Secure (3DS2) / Strong Customer Authentication (SCA) implemented for EU payments per PSD2 regulation
- [ ] Device fingerprinting collected for risk scoring — flag transactions from suspicious devices, VPNs, or known fraud proxies
- [ ] Real-time fraud scoring integrated: either payment processor's built-in (Stripe Radar, Adyen Risk Engine) or third-party (Sift, Kount)
- [ ] High-risk transaction review queue established — suspicious transactions held for manual review before capture
- [ ] Card testing / carding attack detection: rate limit authorization attempts, flag rapid low-value charges from same IP
- [ ] Chargeback monitoring dashboard in place — track chargeback rate and get alerts before exceeding network threshold (1% Visa, 1% Mastercard)

### Authorization & Access Control

- [ ] Payment endpoints require authenticated users — no anonymous payment initiation
- [ ] Admin payment operations (refunds, adjustments, manual charges) require elevated privileges with audit trail
- [ ] Separation of duties enforced: the person who initiates a refund cannot approve it (dual authorization for amounts above threshold)
- [ ] Payment API keys stored in secrets manager — never in code, config files, or environment files committed to Git
- [ ] Test/sandbox and production payment environments strictly separated — test keys never used in production, production keys never in dev
- [ ] Webhook signatures verified cryptographically — never trust unverified payment processor callbacks (Stripe: `stripe-signature`; PayPal: `PAYPAL-TRANSMISSION-SIG`)

---

## 🟡 IMPORTANT (should fix before launch)

### Reconciliation & Accounting

- [ ] Automated reconciliation pipeline built: compare internal ledger against payment processor settlement reports daily
- [ ] Discrepancies flagged and routed to investigation queue — reconciliation failures never silently ignored
- [ ] Bank statement reconciliation automated where possible — match deposits to expected settlement amounts
- [ ] Reconciliation service runs as an independent system with its own data store — not a scheduled job attached to the main ledger
- [ ] End-of-day batch totals verified: sum of transactions processed matches sum reported by payment processor
- [ ] Failed payment retries tracked — know which payments are in retry and when they will be abandoned
- [ ] Settlement timing documented per payment method — credit card (T+2), ACH (T+3-5), wire (T+0-1) — cash flow projections accurate
- [ ] Financial reports generated: daily transaction summary, monthly revenue, refund rates, chargeback rates, fees paid to processors

### Refunds & Disputes

- [ ] Full and partial refund flows implemented and tested end-to-end (API to ledger to payment processor to customer notification)
- [ ] Refund idempotency: duplicate refund requests for the same transaction are detected and rejected
- [ ] Refund limits enforced: cannot refund more than the original transaction amount, cannot refund an already-refunded transaction
- [ ] Refund processing time communicated to users — set expectations (3-5 business days for card refunds, 5-10 for ACH)
- [ ] Chargeback response workflow defined: evidence collection template, submission deadlines, escalation path
- [ ] Chargeback reason codes tracked and analyzed — identify patterns (fraud, product not received, subscription not canceled)
- [ ] Dispute evidence auto-collection: automatically gather receipt, shipping proof, usage logs, communication history for chargeback defense
- [ ] Credit memos / store credit as alternative refund method — reduce cash outflow while maintaining customer satisfaction

### Multi-Currency & International Payments

- [ ] Currency stored as ISO 4217 code alongside amount in every record — `{ amount: 1999, currency: "USD" }` never just `1999`
- [ ] Currency conversion uses provider rates at transaction time — conversion rate recorded immutably with the transaction
- [ ] Rounding rules follow ISO 4217 currency exponent (USD: 2 decimal places, JPY: 0 decimal places, KWD: 3 decimal places)
- [ ] Display formatting uses `Intl.NumberFormat` or equivalent locale-aware formatter — never hardcode `$` or comma separators
- [ ] FX markup/spread documented and disclosed to users where applicable (regulatory requirement in many jurisdictions)
- [ ] Multi-currency settlement configured with payment processor — settle in local currency to avoid double-conversion fees
- [ ] Currency mismatch prevention: verify that charge currency matches the currency displayed to the user at checkout
- [ ] Cross-border payment compliance: sanctions screening (OFAC, EU), restricted countries list maintained and enforced

### Tax & Regulatory

- [ ] Tax calculation engine integrated (Stripe Tax, TaxJar, Avalara, Vertex) — tax computed at checkout based on customer location
- [ ] Tax amounts recorded as separate line items in ledger — never mixed with product revenue
- [ ] Tax ID / VAT number collection and validation for B2B transactions (EU VAT reverse charge, GST for India/Australia)
- [ ] Invoice generation automated: valid tax invoice with all required fields (seller info, buyer info, tax breakdown, invoice number)
- [ ] Digital services tax compliance for cross-border digital sales (EU MOSS/OSS, UK digital services, Australian GST)
- [ ] Receipt emitted for every successful transaction — sent via email and available in user dashboard
- [ ] KYC (Know Your Customer) verification integrated for payment amounts above regulatory thresholds
- [ ] AML (Anti-Money Laundering) screening: transactions screened against sanctions lists and suspicious activity reported per FinCEN/local regulations
- [ ] Money transmission license requirements reviewed — determine if your business model requires a license in operating jurisdictions

### Subscriptions & Recurring Billing (if applicable)

- [ ] Subscription lifecycle fully implemented: creation, upgrade, downgrade, cancellation, pause, resume
- [ ] Proration logic correct for mid-cycle plan changes — tested for upgrade, downgrade, and cancellation scenarios
- [ ] Grace period configured for failed recurring payments — don't cancel immediately, retry with exponential backoff (dunning)
- [ ] Dunning email sequence configured: payment failed → retry scheduled → final warning → subscription suspended
- [ ] Involuntary churn tracked: failed payment rate, card update rate, recovery rate after dunning
- [ ] Card expiry detection: proactively notify users before their card expires (use Stripe Account Updater or similar for auto-update)
- [ ] Free trial abuse prevention: limit one trial per payment method or per device fingerprint
- [ ] Cancellation flow includes retention offers where appropriate — downgrade option, pause, or discount before confirming cancel
- [ ] Subscription status webhook handling is idempotent — handle out-of-order and duplicate webhook deliveries
- [ ] Revenue recognition aligned with ASC 606 / IFRS 15 — deferred revenue tracked for prepaid subscriptions

### Audit Trail & Compliance

- [ ] Every financial state change logged with: timestamp (UTC), actor (user/system), action, before-state, after-state, IP address
- [ ] Audit logs are append-only and stored in tamper-evident storage (separate from application database, WORM storage or signed logs)
- [ ] Audit log retention meets regulatory requirements: 7 years for IRS/SOX, 5 years for PCI DSS, jurisdiction-specific for GDPR
- [ ] Admin actions on payment data produce audit entries: who accessed what, when, from where
- [ ] SOX compliance: financial controls documented, access to financial systems reviewed quarterly, segregation of duties enforced (if publicly traded)
- [ ] PCI DSS audit trail requirements: all access to cardholder data logged, log integrity monitoring in place, logs reviewed daily
- [ ] Audit trail exportable for regulatory examination — structured format (CSV, JSON) with complete chain of custody

### Monitoring & Alerting

- [ ] Payment success rate monitored in real-time — alert if decline rate exceeds baseline by > 5% (indicates processor issue or fraud attack)
- [ ] Payment latency tracked: p50, p95, p99 for authorization, capture, and refund operations
- [ ] Payment processor health monitored — alert on elevated error rates or timeouts from processor API
- [ ] Chargeback rate monitored — alert at 0.5% (well before the 1% network threshold that triggers penalties)
- [ ] Revenue dashboards: GMV, net revenue, refund rate, average order value, payment method breakdown (updated real-time or hourly)
- [ ] Failed webhook delivery monitoring — ensure no payment events are lost (dead letter queue for failed webhook processing)
- [ ] Reconciliation discrepancy alerts — any mismatch between internal ledger and processor reports triggers investigation

---

## 🟢 NICE-TO-HAVE (polish)

- [ ] Multiple payment methods supported: cards, ACH/bank transfer, digital wallets (Apple Pay, Google Pay), Buy Now Pay Later (Klarna, Afterpay)
- [ ] Saved payment methods: users can save and manage multiple cards/bank accounts (using processor tokenization, never storing raw data)
- [ ] One-click checkout for returning customers (Stripe Link, Shop Pay, PayPal One Touch)
- [ ] Payment method fallback: if primary payment method fails, automatically try backup method (requires user opt-in)
- [ ] Split payments / marketplace payouts implemented: funds distributed to multiple recipients per transaction (Stripe Connect, PayPal for Marketplaces)
- [ ] Payout scheduling: sellers/creators paid on configurable schedule (daily, weekly, monthly) with minimum threshold
- [ ] Payment analytics: conversion funnel tracking from cart to payment success, drop-off analysis by payment method and device
- [ ] Smart payment routing: route transactions to the processor with highest approval rate for the card type/region
- [ ] Network tokenization enabled: store network tokens for improved approval rates and automatic card updates (Visa Token Service, Mastercard MDES)
- [ ] 3DS challenge optimization: use risk-based authentication to minimize friction for low-risk transactions (exemption engine)
- [ ] Automated accounting integration: sync transactions to QuickBooks, Xero, or NetSuite in real-time
- [ ] Multi-processor failover: if primary payment processor is down, route to backup processor within seconds
- [ ] Crypto payment option available if relevant to user base (USDC, ETH via provider like Coinbase Commerce, BitPay)
- [ ] Pre-authorization and delayed capture for order-type workflows (authorize at order, capture at shipment)
- [ ] Subscription pause/resume: users can temporarily pause subscriptions without canceling (retention tool)
- [ ] Usage-based billing: track metered usage and generate invoices based on consumption (for SaaS/API products)
- [ ] Dynamic currency conversion: show prices in customer's local currency with real-time conversion rates
- [ ] Payment receipt customization: branded receipts with itemized breakdown, tax details, and support contact
