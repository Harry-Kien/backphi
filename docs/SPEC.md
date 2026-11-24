# BACKPHI - Product & Engineering Spec

## 1. What we are building
BACKPHI (backphi.com) is a trading-fee rebate platform for Forex and crypto traders.
BACKPHI is NOT a broker or an exchange. It is an introducing broker (IB) / affiliate
of brokers and exchanges. A broker pays BACKPHI a commission on the trading activity
of referred clients. BACKPHI returns a configured share of that commission to the
trader.

Money flow: broker -> BACKPHI commission -> rebate to the trader (+ optional share to
the partner who referred the trader) -> BACKPHI keeps the margin.

Reference for scope only: backcom.io. Study its public pages to understand the
feature set. Do not copy its text, design, layout, images, logo, numbers or
testimonials. All BACKPHI code, copy, design and brand assets must be original.

## 2. Roles
- Visitor: reads the public site, uses tools, signs up.
- Trader (user): links broker accounts, tracks rebates, withdraws.
- Partner (sub-IB, KOL, community owner): refers traders, earns a commission share.
- Admin roles: super_admin, finance, support, content. Least privilege.

## 3. Core flows (each must work end to end)
F1 Onboarding. Trader signs up -> picks a broker -> opens a broker account through
   BACKPHI's referral link or code (or follows the broker-specific "change partner"
   guide) -> submits the broker account ID (UID) -> the UID is verified as being
   under BACKPHI's IB -> status becomes active.
F2 Commission ingestion. A per-broker adapter imports commission and volume records
   (API sync or CSV upload) -> records are matched to verified UIDs -> unmatched
   records go to a review queue. Imports are idempotent.
F3 Rebate calculation. For each commission record, apply the rate in force on the
   trade date for that broker / account type / user tier / promotion -> create
   rebate entries. Two payout modes per broker: DIRECT (the broker pays the trader's
   trading account; BACKPHI only displays it) and WALLET (BACKPHI credits its own
   wallet ledger).
F4 Withdrawal. Trader requests a withdrawal from the wallet -> validations (minimum,
   limits, verified destination, 2FA) -> finance approves -> payout is recorded with
   an external reference -> ledger updated. A failed payout reverses cleanly.
F5 Referral and partner. Referral link and code, attribution at signup, partner share
   computed from the same commission records, prevention of self-referral and
   duplicate accounts.
F6 Reconciliation. Per broker and period: commission received = rebates + partner
   shares + BACKPHI margin. Mismatches are reported, never hidden.

## 4. Surfaces
4.1 Public site (SEO, i18n): home, how it works, broker/exchange directory with one
    page per broker (rate, payout schedule, payout mode, step-by-step guide), rebate
    calculator, UID checker ("is my account under BACKPHI?"), VIP tiers, promotions,
    referral program, partner program, trader tools (position size calculator,
    market hours, broker rebate comparison), blog, FAQ, contact, legal pages (terms,
    privacy, risk disclosure).
4.2 Trader app: dashboard (rebates by broker and by day, fees, volume; pending,
    credited, withdrawn), linked accounts and their status, rebate history with
    filters and export, wallet and withdrawals, referral center, notifications,
    support tickets, profile, security (2FA, sessions), language.
4.3 Partner portal: application and verification, referred traders (privacy-safe),
    volume and commission share by day/week/month, links and codes, payouts, exports.
4.4 Admin: KPIs (users, verified accounts, commission in, rebate liability, pending
    withdrawals), broker and rate management (effective-dated), UID verification
    queue, commission imports and unmatched queue, rebate runs, withdrawal approvals,
    reconciliation reports, users and partners, promotions and VIP tiers, CMS for
    blog / FAQ / broker pages, support desk, settings, roles, audit log viewer.

## 5. Broker integration layer
One adapter interface per broker: verifyUid(), fetchCommissions(from, to),
describeCapabilities(). Each adapter declares whether it works by API or CSV import.
Build first: (a) a generic CSV-import adapter, (b) one API adapter behind a feature
flag, (c) a mock adapter with seed data for tests and demos.
Brokers, rates, referral links and schedules are database configuration, never
hard-coded. Broker API credentials are stored encrypted and are never logged or sent
to the client.
Do not invent broker API endpoints. Read the broker's official partner/affiliate API
documentation first. If it is unavailable, use the CSV adapter and record the gap.

## 6. Money rules (non-negotiable)
- Immutable double-entry ledger. Balances are derived from ledger entries, or from
  materialized balances that are reconciled against them. No UPDATE or DELETE on
  ledger entries.
- Corrections are reversal or adjustment entries with a reason and an actor.
- Decimal arithmetic only. Amounts are stored as integers in minor units or as
  NUMERIC with explicit currency and precision. No floating point in money paths.
- Every money-moving operation is atomic, carries an idempotency key, and is safe
  under concurrent requests and retries.
- Admins cannot edit a balance directly. Sensitive actions (approve withdrawal,
  change rate, manual adjustment, role change) require re-authentication or 2FA and
  are audit-logged.
- Rates are effective-dated. Past rebates are never recalculated silently.

## 7. Data model (minimum)
users, roles, permissions, user_profiles, sessions, brokers, broker_rate_plans,
broker_credentials, broker_accounts (user <-> UID, status), uid_verifications,
commission_imports, commission_records, rebate_entries, ledger_accounts,
ledger_entries, wallets, withdrawal_requests, payout_records, partners,
referral_codes, referral_attributions, partner_shares, vip_tiers, promotions,
notifications, support_tickets, support_messages, cms_pages, blog_posts, currencies,
system_settings, audit_logs, idempotency_keys.
Document every table, constraint and index in docs/DATABASE.md.

## 8. Stack
Inspect the repository first. Keep a working stack unless there is a documented
technical reason to change it. If the repository is empty, use: TypeScript end to
end; Next.js (App Router) for the public site and the portals; PostgreSQL; a typed
ORM with SQL migrations; Redis and a job queue for imports, rebate runs and
notifications; a worker process separate from the web process; REST API under
/api/v1 with OpenAPI documentation; S3-compatible storage; Docker Compose for local
development; CI with lint, typecheck and tests.
Record every choice in docs/DECISIONS.md.

## 9. Quality bars
Security: OWASP Top 10 mitigations, strong password hashing, secure sessions, CSRF /
XSS / SQL injection protection, rate limiting, RBAC checked on the server for every
route, upload validation, secrets only in environment, encrypted broker credentials,
backup and restore.
Testing: unit, integration, API and end-to-end tests. Mandatory cases: duplicate
import, duplicate withdrawal request, concurrent approval, wrong currency, rate
change mid-period, reversal, failed payout, unauthorized access for each role,
self-referral, database rollback.
Performance: paginated lists, indexed queries, cached public pages, background jobs
for heavy work. Set numeric targets in docs/PLAN.md and measure them.
UI: original, trustworthy fintech look; clear data hierarchy; mobile first for the
trader app; loading, empty and error states on every screen; accessible forms.
i18n: Vietnamese and English at launch; structure ready for more locales.
Currency: USD and USDT at launch; structure ready for more.

## 10. Honesty and compliance rules
- Never fabricate statistics, payout totals, user counts, reviews or testimonials.
  Use real data or leave the block out.
- Never state or imply guaranteed profit. Every page that mentions trading carries a
  risk disclosure. State clearly that BACKPHI is an independent intermediary.
- Use broker names and logos only as each broker's partner guidelines allow.
- Real payouts, KYC/AML and anything that depends on local law stay behind feature
  flags that are OFF until the owner confirms legal sign-off.
- If something needs a real account, API key, payment provider or legal decision,
  do not fake it. Build against the mock adapter and list it in
  docs/OWNER_ACTIONS.md.

## 11. Phases
 1 Repository audit, competitor research (section 13) and plan
 2 Architecture and data model
 3 Design system and app shell
 4 Auth, roles, i18n
 5 Brokers, rate plans, UID linking and verification
 6 Commission ingestion, rebate engine, ledger
 7 Wallet, withdrawals, payouts, reconciliation
 8 Referral and partner portal
 9 Admin portal
10 Public site, tools, CMS
11 Security audit, load test, full test pass
12 Deployment, monitoring, backups, runbook

## 12. Definition of done for a phase
1. Code committed; lint, typecheck and all tests green (paste the command output).
2. New behaviour is covered by tests; money paths cover the mandatory cases.
3. An agent that did not write the code has reviewed it; findings fixed or logged.
4. docs/ updated; docs/PROGRESS.md lists done, not done, known issues, next steps.
5. Acceptance report written to docs/reports/phase-N.md with evidence.
Never claim completion without evidence.

## 13. Competitive research and differentiation
Research (Phase 1). Use the REA skill (reverse-engineer-anything) and normal
browsing to study backcom.io as a reference: page map, onboarding steps, dashboard
structure, data shown per broker, calculators and tools, languages, support
channels. Write the result to docs/COMPETITOR.md as a feature inventory in your own
words, with a column "how BACKPHI does this better".

Limits. Only public pages, plus what an ordinary logged-in user sees in the owner's
own account. No copying of their source code, text, images or logos. No probing of
private APIs, no bypassing of authentication or rate limits, no access to other
users' data, no heavy automated crawling. Clean-room rule: only the research agent
looks at the reference site; build agents work from docs/COMPETITOR.md and this
spec, never from the reference site's code.

Where BACKPHI must be better (build these as first-class features):
- Full transparency for every rebate entry: commission received, rate applied,
  formula, payout mode and date, all visible to the trader.
- Every broker's rate, payout schedule, withdrawal method, fee and minimum is
  published on its public page. Nothing is hidden behind signup.
- UID verification shows live status and a stated time target, gives a clear reason
  on rejection and offers a one-click retry.
- Installable mobile web app (PWA) from day one, fast on a mid-range phone.
- The rebate calculator and broker comparison use the real configured rates.
- Notifications by email and Telegram for verification, credited rebates, payouts.
- Vietnamese-first copy and support flows; English as the second locale.
- Only verifiable numbers on the site (see section 10).
