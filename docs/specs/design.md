# Design Specification

> Architecture, data models, state machines, and design decisions for the Airalo QA test suite.

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                   Monorepo Root                      │
│  eslint.config.mjs · tsconfig.base.json · CI/CD     │
├────────────┬────────────────┬────────────────────────┤
│ packages/  │   apps/api/    │     apps/web/          │
│  shared/   │                │                        │
│            │  Services      │  Page Objects           │
│  Schemas   │  State Machines│  Playwright Tests       │
│  Types     │  Jest Tests    │  axe-core a11y          │
│  Constants │  Fixtures      │  Web Vitals perf        │
└────────────┴────────────────┴────────────────────────┘
```

### Design Principles

1. **Schema-First Contracts** — Zod schemas in `packages/shared` are the canonical definition of every API shape. Both fixture tests and live-API tests parse through the same schemas.
2. **Factory Pattern for Services** — `createAuthService(http)`, `createOrderService(http)`, `createEsimService(http)` accept an Axios instance and return an interface. This enables full mocking in unit tests while sharing real implementations in integration tests.
3. **State Machine Purity** — All state machines are pure functions `(state, event, ctx) → { state, ctx }` with zero I/O. They model the domain lifecycle and are testable without mocks.
4. **Page Object Model** — `AiraloHomePage` and `CountryPackagesPage` encapsulate all Playwright locators and actions. Tests read like business scenarios.
5. **Data-Driven Fixtures** — Countries and languages are JSON fixtures hydrated at import time. RegExp patterns are serialized as strings in JSON and deserialized by `toRegExp()`.

## 2. Data Models

### 2.1 Token Response

```
{ data: { access_token: string, token_type: string, expires_in: number }, meta: { message: string } }
```

### 2.2 Order Response

```
{ data: { id, code, currency, package_id, quantity, type, esim_type, validity, price, created_at,
          sims: [{ id, iccid, lpa, qrcode, qrcode_url, matching_id, apn_type, is_roaming, created_at }] },
  meta: { message: string } }
```

### 2.3 eSIM Response

```
{ data: { id, iccid, lpa, qrcode, qrcode_url, matching_id, apn_type, is_roaming, created_at,
          simable?: { id, code, package_id, quantity, type, esim_type, validity, price, currency } },
  meta: { message: string } }
```

### 2.4 Error Shapes (Live API)

The live Airalo API uses **inconsistent error envelopes** across endpoints:

| Endpoint | HTTP Status | Shape |
|----------|-------------|-------|
| POST /token (invalid creds) | 422 | `{ data: { [field]: string }, meta: { message } }` |
| POST /orders (bad package) | 422/400 | `{ code: number, reason: string }` |
| GET /sims/{bad iccid} | 404 | `{ data: [], meta: { message } }` |

Each shape has its own Zod schema (`airaloFieldErrorSchema`, `airaloReasonErrorSchema`, `airaloNotFoundErrorSchema`).

## 3. State Machines

### 3.1 OAuth Token Machine

```
NO_TOKEN ──AUTHENTICATE──▶ REQUESTING
REQUESTING ──SUCCESS──▶ VALID_TOKEN
REQUESTING ──FAILURE──▶ FAILED
VALID_TOKEN ──TTL_EXPIRED──▶ EXPIRED
EXPIRED ──AUTHENTICATE──▶ REQUESTING
FAILED ──RETRY──▶ REQUESTING (if retryCount < MAX)
FAILED ──RETRY──▶ FAILED (max retries exceeded)
```

Guards: missing credentials → FAILED immediately. Max retries = 3.

### 3.2 Order Machine

```
IDLE ──SUBMIT_ORDER──▶ CREATING
CREATING ──ORDER_SUCCESS──▶ CREATED
CREATING ──VALIDATION_ERROR──▶ VALIDATION_FAILED
CREATING ──AUTH_ERROR──▶ AUTH_FAILED
CREATED ──ALL_ESIMS_FETCHED──▶ COMPLETE
{VALIDATION_FAILED, AUTH_FAILED, COMPLETE} ──RESET──▶ IDLE
```

Guards: empty token → AUTH_FAILED, invalid input → VALIDATION_FAILED.

### 3.3 eSIM Machine

```
UNRESOLVED ──FETCH──▶ FETCHING
FETCHING ──SUCCESS──▶ RESOLVED
FETCHING ──NOT_FOUND──▶ NOT_FOUND
FETCHING ──ERROR──▶ ERROR
{NOT_FOUND, ERROR} ──RETRY──▶ FETCHING
```

Guard: empty iccid → ERROR.

### 3.4 Full Flow Machine

```
INIT ──AUTHENTICATE_SUCCESS──▶ AUTHENTICATED
INIT ──AUTH_FAILED──▶ FAILED
AUTHENTICATED ──SUBMIT_ORDER_SUCCESS──▶ ORDER_SUBMITTED
AUTHENTICATED ──TOKEN_EXPIRED──▶ RE_AUTH_REQUIRED
RE_AUTH_REQUIRED ──RE_AUTHENTICATED──▶ AUTHENTICATED
ORDER_SUBMITTED ──FETCH_ESIMS──▶ ESIMS_FETCHING
ESIMS_FETCHING ──ALL_RESOLVED──▶ COMPLETE
ESIMS_FETCHING ──ANY_FAILED──▶ FAILED
```

## 4. Service Layer Design

```
AiraloService (Composite)
├── AuthService    → POST /token
├── OrderService   → POST /orders
└── EsimService    → GET /sims/{iccid}
    ↑
    └── Token caching (in-process)
```

`AiraloService` implements `AiraloFlowService` (the orchestration interface), enabling it to be passed directly to `runAiraloOrderFlow()`.

`runAiraloOrderFlow()` orchestrates: submit order → parallel eSIM fetch for all SIMs.

## 5. Web Page Object Design

```
AiraloHomePage
├── searchInput (role=textbox)
├── searchDropdown (role=listbox)
├── cookieAcceptButton (role=button)
├── goto() → dismissCookieBanner()
├── searchCountry(term) → selectCountryFromDropdown(name)
└── waitForCountryPage(slug)

CountryPackagesPage
├── pageHeading, locationTitle, operatorName
├── packageButtons, durationTitles
├── tab(name) → standardTab, unlimitedTab
├── cartDialog → cartTotalPrice, buyNowButton
├── planDetailsPanel → planDetailItems
├── verifyPackagesLoaded() → clickTab() → selectPackage()
└── openPackageDetails()
```

## 6. Retry & Transient Error Strategy

`retryOnTransient(fn, opts)` — exponential backoff (2s → 4s → 8s, 3 attempts) for:
- HTTP 5xx server errors
- HTTP 429 rate-limit responses
- Network errors: ECONNABORTED, ETIMEDOUT, ECONNRESET

Used in all live-API tests and integration tests. This is critical because the Airalo sandbox rate-limits concurrent token requests across test groups.

## 7. Data-Driven Country/Language Configuration

Countries and languages are defined as JSON fixtures (`countries.json`, `languages.json`) and hydrated at import time:

- **Serialized RegExp** patterns in JSON (e.g. `"/Japan/i"`) are converted to native `RegExp` via `toRegExp()`
- `resolveCountryConfig(country, lang)` merges raw country data with localized language strings
- Environment variables (`AIRALO_COUNTRY`, `AIRALO_LANGUAGE`) control which configuration is active
- This design enables **parameterized CI matrix** runs across different country/language combos
