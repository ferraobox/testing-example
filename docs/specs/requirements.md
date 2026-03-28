# Requirements Specification

> Source of truth for all acceptance criteria. Every test traces back to a requirement ID here.

## R1 — OAuth2 Authentication (POST /token)

| ID | Requirement | Priority |
|----|------------|----------|
| R1.1 | Obtain a valid Bearer token using `client_credentials` grant | Must |
| R1.2 | Token response must contain `access_token`, `token_type`, `expires_in` | Must |
| R1.3 | Token response must include `meta.message` | Must |
| R1.4 | Invalid credentials must return a parseable error response | Must |
| R1.5 | Token must be non-empty and usable for subsequent API calls | Must |
| R1.6 | `expires_in` must be a positive integer | Must |

## R2 — Order Submission (POST /orders)

| ID | Requirement | Priority |
|----|------------|----------|
| R2.1 | Submit an order with `package_id` and `quantity` using Bearer token | Must |
| R2.2 | Order response must include `id`, `code`, `package_id`, `quantity`, `sims[]` | Must |
| R2.3 | Each SIM entry must include `iccid`, `id`, `lpa`, `qrcode`, `matching_id` | Must |
| R2.4 | Number of SIMs returned must equal requested `quantity` | Must |
| R2.5 | All SIM `iccid` values in an order must be unique | Must |
| R2.6 | Invalid `package_id` must return a parseable error response | Must |
| R2.7 | Missing token must return an auth error | Must |
| R2.8 | `meta.message` must match `/success/i` on success | Must |

## R3 — eSIM Retrieval (GET /sims/{iccid})

| ID | Requirement | Priority |
|----|------------|----------|
| R3.1 | Retrieve eSIM details by `iccid` using Bearer token | Must |
| R3.2 | All 6 eSIM responses from an order must parse without errors | Must |
| R3.3 | eSIM response must include `iccid`, `lpa`, `qrcode`, `qrcode_url`, `matching_id`, `is_roaming` | Must |
| R3.4 | `simable` field must reference the parent order's `package_id` | Should |
| R3.5 | Invalid `iccid` must return a parseable not-found/error response | Must |
| R3.6 | eSIM `id` must be a positive number, `created_at` non-empty | Must |

## R4 — Web UI: Country eSIM Purchase Flow

| ID | Requirement | Priority |
|----|------------|----------|
| R4.1 | Homepage loads with search input visible | Must |
| R4.2 | Search for a country returns matching results in dropdown | Must |
| R4.3 | Selecting a country navigates to its eSIM packages page | Must |
| R4.4 | Country page displays heading, operator, breadcrumb, package tabs | Must |
| R4.5 | Clicking a package tab filters packages by type (Standard/Unlimited) | Must |
| R4.6 | Selecting a package shows cart dialog with matching price | Must |
| R4.7 | "Package details" panel shows policies (activation, validity, fair usage) | Must |
| R4.8 | "Broader coverage" section is visible on country page | Should |
| R4.9 | Search is case-insensitive | Must |
| R4.10 | Non-existent country search returns empty results | Must |

## R5 — Accessibility

| ID | Requirement | Priority |
|----|------------|----------|
| R5.1 | Homepage must pass axe-core WCAG 2.1 AA scan (no critical violations) | Must |
| R5.2 | Country eSIM page must pass axe-core scan (no critical violations) | Must |
| R5.3 | Search input must be keyboard-navigable | Should |
| R5.4 | Images should have alt text (< 10% missing) | Should |
| R5.5 | Color contrast violations must be ≤ 5 | Should |

## R6 — Performance (Core Web Vitals)

| ID | Requirement | Priority |
|----|------------|----------|
| R6.1 | First Contentful Paint (FCP) ≤ 1800 ms | Should |
| R6.2 | Time to First Byte (TTFB) ≤ 800 ms | Should |
| R6.3 | Largest Contentful Paint (LCP) ≤ 2500 ms | Should |
| R6.4 | Cumulative Layout Shift (CLS) ≤ 0.1 | Should |
| R6.5 | Interaction to Next Paint (INP) ≤ 200 ms | Should |

## R7 — Schema Contracts & Backward Compatibility

| ID | Requirement | Priority |
|----|------------|----------|
| R7.1 | All API responses must validate against Zod schemas | Must |
| R7.2 | Schemas must strip unknown fields (Zod default) — future-proof | Must |
| R7.3 | Minimum viable payloads must still parse (no over-constraining) | Must |
| R7.4 | Removal of required fields must be detected as breaking | Must |
| R7.5 | Type changes on required fields must be detected as breaking | Must |
| R7.6 | Cross-fixture consistency (eSIM iccid ∈ order sims, etc.) | Must |

## R8 — State Machine Lifecycle

| ID | Requirement | Priority |
|----|------------|----------|
| R8.1 | OAuth token machine: NO_TOKEN → REQUESTING → VALID_TOKEN → EXPIRED cycle | Must |
| R8.2 | Order machine: IDLE → CREATING → CREATED → COMPLETE lifecycle | Must |
| R8.3 | eSIM machine: UNRESOLVED → FETCHING → RESOLVED lifecycle | Must |
| R8.4 | Full flow machine: INIT → AUTHENTICATED → ORDER_SUBMITTED → ESIMS_FETCHING → COMPLETE | Must |
| R8.5 | All machines must handle error states and recovery (RETRY, RESET) | Must |
| R8.6 | Invalid transitions must be no-ops (state unchanged) | Must |
