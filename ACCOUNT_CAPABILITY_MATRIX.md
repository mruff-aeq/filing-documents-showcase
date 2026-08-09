# Account / credential capability matrix (dev)

What each credential type can and cannot do when driving the business API in dev. Built from the
e2e suite work (2026-08). "Account-Id: 3040" is used throughout as the billing org.

## The three credentials

| Short | What it is | Key roles |
|---|---|---|
| **SVC** | `entity-service-account` JWT (client_credentials) | `system`, `edit`, `skip_unaffiliated_business_check`, `names_viewer` |
| **IDIR** | a staff IDIR session token (loginSource IDIR, staff org) | `staff`, `edit`, `make_payment`, `manage_business` |
| **API_GW** | org gateway API key, key-only, no bearer JWT (loginSource API_GW) | none (key auth only) |

## Capability matrix

| Capability | SVC | IDIR | API_GW | Notes |
|---|:--:|:--:|:--:|---|
| Standard filing on an affiliated business (COD/COA/alteration/SR/changeOfRegistration/staff filings) | ✓ | ✓ | ✓ | API_GW only on its own account's businesses |
| Bootstrap a numbered new business (IA, numbered amalgamation) | ✓ | ✓ | ✓ | |
| File a payload carrying a **Completing Party** (amalgamation, continuationIn) | ✓ | ✓ | ✗ | API_GW → "Permission Denied: cannot edit the completing party" (bug 03). SVC `system` role bypasses it |
| Fee-bearing filing **settles** (invoice → PAID) | ✓* | ✓* | ✓* | *only if the business is **affiliated** to the Account-Id org; otherwise falls to an unpaid DIRECT_PAY and hangs PENDING. This is about affiliation, not the credential |
| **Affiliate** a business to an org | ✗ | ✓ | ✗ | SVC → 401 INVALID_USER_CREDENTIALS. IDIR `staff` can (`POST /auth/orgs/{id}/affiliations`) |
| **Approve a Name Request** (makes an NR consumable) | ✗ | ✗ | ✗ | needs `names_approver`; **none** of these have it. Blocks coop IA + firm registration |
| **Approve a staff review** (continuationIn AWAITING_REVIEW → APPROVED) | ✗ | ✓ | ✗ | `POST /admin/reviews/{id}` needs `staff`; only IDIR has it. This is the one thing that made continuationIn completable |
| One-step non-draft `POST /businesses` | ✓ | ✓ | ✗ | API_GW one-step is broken (bug 05); SVC uses the two-step draft→PUT |

## Gates no credential clears (environment/data, not auth)

These block the filing regardless of which credential you use — the fix is config/data/code, not a
better token:

| Blocked thing | Why | Ticket |
|---|---|---|
| `courtOrder` on a coop (CP) | pay-api has no `fee_schedule` row for (CP, COURT) → 402 | 07 |
| `putBackOn` on SP/GP/CP | LaunchDarkly flag `supported-put-back-on-entities` excludes them → 403 (even though `allowedActions` advertises it) | 08 |
| Coop `incorporationApplication` / firm `registration` | need a consumable NR → need `names_approver`, which no available credential has | — |

## Practical guidance

- **Default driver: SVC.** It files everything on affiliated businesses, bypasses the completing-party
  wall, and needs no browser session. Use it for the whole suite.
- **Reach for IDIR only for the two things SVC can't do:** (1) **affiliate** a business (needed before a
  fee-bearing filing on a business the org doesn't own, e.g. an aged coop's annual report), and
  (2) **approve a staff review** (continuationIn). A staff IDIR token is short-lived (expires in hours).
- **Avoid API_GW keys.** Key-only accounts hit the completing-party wall and the broken one-step POST;
  the suite dropped them entirely.
- **NR-gated creations stay blocked** for everyone here until a token carries `names_approver`.
