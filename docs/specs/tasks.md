# Task Specification

> Maps every requirement to the test files and test cases that verify it.

## Traceability Matrix

### R1 — OAuth2 Authentication

| Req ID | Test File | Test Case | Layer |
|--------|-----------|-----------|-------|
| R1.1 | `apps/api/test/unit/authService.test.ts` | sends POST to /token, sends client_id/secret, grant_type | Unit |
| R1.1 | `apps/api/test/integration/live-api.test.ts` | obtains an OAuth2 access token | Integration |
| R1.1 | `apps/api/test/spec/live-contract.test.ts` | parses live token response without errors | Live Contract |
| R1.2 | `apps/api/test/unit/authService.test.ts` | returns the exact access_token from fixture | Unit |
| R1.2 | `apps/api/test/spec/api-contract.test.ts` | token contract — field presence and types | Contract |
| R1.3 | `apps/api/test/spec/api-contract.test.ts` | token contract — meta.message | Contract |
| R1.4 | `apps/api/test/unit/authService.test.ts` | propagates 401/422/500 | Unit |
| R1.4 | `apps/api/test/integration/live-api.test.ts` | rejects invalid/empty credentials | Integration |
| R1.4 | `apps/api/test/spec/live-contract.test.ts` | rejects invalid credentials (parseable error) | Live Contract |
| R1.5 | `apps/api/test/unit/authService.test.ts` | throws when access_token is empty string | Unit |
| R1.5 | `apps/api/test/spec/api-contract.test.ts` | token contract — access_token non-empty | Contract |
| R1.6 | `apps/api/test/unit/authService.test.ts` | throws when expires_in is negative | Unit |
| R1.6 | `apps/api/test/spec/api-contract.test.ts` | token contract — expires_in positive | Contract |

### R2 — Order Submission

| Req ID | Test File | Test Case | Layer |
|--------|-----------|-----------|-------|
| R2.1 | `apps/api/test/unit/orderService.test.ts` | sends POST to /orders, sends package_id/quantity/token | Unit |
| R2.1 | `apps/api/test/integration/live-api.test.ts` | creates an order with the default package | Integration |
| R2.2 | `apps/api/test/unit/orderService.test.ts` | returns orderId, attaches raw order data | Unit |
| R2.2 | `apps/api/test/spec/api-contract.test.ts` | order contract — id, code, package_id, quantity, sims[] | Contract |
| R2.3 | `apps/api/test/spec/api-contract.test.ts` | order contract — sims contain iccid, id, lpa, qrcode, matching_id | Contract |
| R2.4 | `apps/api/test/unit/orderService.test.ts` | returns sims array with correct iccids | Unit |
| R2.4 | `apps/api/test/unit/airalo-flow.test.ts` | handles single/large orders (1, 6, 50 SIMs) | Unit |
| R2.4 | `apps/api/test/spec/api-contract.test.ts` | order contract — sims count = quantity | Contract |
| R2.5 | `apps/api/test/spec/api-contract.test.ts` | order contract — all iccids unique | Contract |
| R2.6 | `apps/api/test/unit/orderService.test.ts` | propagates 422 for invalid package_id, zero quantity | Unit |
| R2.6 | `apps/api/test/unit/airalo-flow.test.ts` | propagates 422/429/500/502/503 from createOrder | Unit |
| R2.6 | `apps/api/test/integration/live-api.test.ts` | rejects invalid package_id / zero / negative / empty | Integration |
| R2.7 | `apps/api/test/unit/orderService.test.ts` | propagates 401 for invalid token | Unit |
| R2.7 | `apps/api/test/integration/live-api.test.ts` | rejects expired/invalid/empty token | Integration |
| R2.8 | `apps/api/test/unit/orderService.test.ts` | returns message from meta | Unit |
| R2.8 | `apps/api/test/spec/api-contract.test.ts` | order contract — meta.message matches /success/i | Contract |

### R3 — eSIM Retrieval

| Req ID | Test File | Test Case | Layer |
|--------|-----------|-----------|-------|
| R3.1 | `apps/api/test/unit/esimService.test.ts` | sends GET to /sims/{iccid}, returns iccid | Unit |
| R3.1 | `apps/api/test/integration/live-api.test.ts` | retrieves eSIM details by iccid | Integration |
| R3.2 | `apps/api/test/unit/airalo-flow.test.ts` | returns eSIM details in same order (Promise.all) | Unit |
| R3.2 | `apps/api/test/spec/live-contract.test.ts` | all 6 eSIM responses parse without Zod errors | Live Contract |
| R3.3 | `apps/api/test/unit/esimService.test.ts` | attaches raw eSIM data (all fields) | Unit |
| R3.3 | `apps/api/test/spec/api-contract.test.ts` | esim contract — iccid, lpa, qrcode, qrcode_url, matching_id, is_roaming | Contract |
| R3.4 | `apps/api/test/unit/esimService.test.ts` | attaches simable reference to parent order | Unit |
| R3.4 | `apps/api/test/spec/api-contract.test.ts` | esim contract — simable.package_id matches order | Contract |
| R3.5 | `apps/api/test/unit/esimService.test.ts` | propagates 404/401/422/500 | Unit |
| R3.5 | `apps/api/test/unit/airalo-flow.test.ts` | propagates 404/401/500/503 from fetchEsim | Unit |
| R3.5 | `apps/api/test/integration/live-api.test.ts` | rejects invalid iccid / no auth | Integration |
| R3.6 | `apps/api/test/spec/api-contract.test.ts` | esim contract — id positive number, created_at non-empty | Contract |

### R4 — Web UI: Country Purchase Flow

| Req ID | Test File | Test Case | Layer |
|--------|-----------|-----------|-------|
| R4.1 | `apps/web/test/e2e/airalo-flow.spec.ts` | open homepage, verify loaded, location tabs | E2E |
| R4.1 | `apps/web/test/unit/language.test.ts` | getLanguageConfig resolution | Unit |
| R4.2 | `apps/web/test/e2e/airalo-flow.spec.ts` | search for country, select from dropdown | E2E |
| R4.3 | `apps/web/test/e2e/airalo-flow.spec.ts` | navigates to country eSIM page | E2E |
| R4.3 | `apps/web/test/unit/country.test.ts` | getCountryConfig resolution | Unit |
| R4.4 | `apps/web/test/e2e/airalo-flow.spec.ts` | heading, operator, breadcrumb, package tabs | E2E |
| R4.4 | `apps/web/test/unit/country.test.ts` | resolveCountryConfig merges data | Unit |
| R4.5 | `apps/web/test/e2e/airalo-flow.spec.ts` | click tab, verify packages load, duration groups | E2E |
| R4.5 | `apps/web/test/unit/country.test.ts` | resolveFirstCardSpec — tab/Unlimited localization | Unit |
| R4.6 | `apps/web/test/e2e/airalo-flow.spec.ts` | select package, cart dialog, price match | E2E |
| R4.6 | `apps/web/test/unit/format.test.ts` | normalisePrice extraction/comparison | Unit |

### R5 — Accessibility

| Req ID | Test File | Test Case | Layer |
|--------|-----------|-----------|-------|
| R5.1 | `apps/web/test/a11y/airalo-a11y.spec.ts` | homepage — no critical violations, landmarks, ARIA | A11y |
| R5.2 | `apps/web/test/a11y/airalo-a11y.spec.ts` | country page — no critical violations, cards, focusable | A11y |
| R5.3 | `apps/web/test/a11y/airalo-a11y.spec.ts` | search input keyboard navigable | A11y |
| R5.4 | `apps/web/test/a11y/airalo-a11y.spec.ts` | all images have alt text | A11y |
| R5.5 | `apps/web/test/a11y/airalo-a11y.spec.ts` | color contrast meets WCAG AA | A11y |

### R6 — Performance

| Req ID | Test File | Test Case | Layer |
|--------|-----------|-----------|-------|
| R6.1 | `apps/web/test/perf/airalo-perf.spec.ts` | FCP ≤ 1800 ms | Perf |
| R6.2 | `apps/web/test/perf/airalo-perf.spec.ts` | TTFB ≤ 800 ms | Perf |
| R6.3 | `apps/web/test/perf/airalo-perf.spec.ts` | LCP ≤ 2500 ms | Perf |
| R6.4 | `apps/web/test/perf/airalo-perf.spec.ts` | CLS ≤ 0.1 | Perf |
| R6.5 | `apps/web/test/perf/airalo-perf.spec.ts` | INP ≤ 200 ms | Perf |

### R7 — Schema Contracts

| Req ID | Test File | Test Case | Layer |
|--------|-----------|-----------|-------|
| R7.1 | `apps/api/test/spec/api-contract.test.ts` | all endpoint schemas parse fixtures | Contract |
| R7.1 | `apps/api/test/spec/live-contract.test.ts` | all live responses validate against schemas | Live Contract |
| R7.2 | `apps/api/test/spec/api-contract.test.ts` | unknown fields stripped, backward compat | Contract |
| R7.3 | `apps/api/test/spec/api-contract.test.ts` | minimum viable payloads still parse | Contract |
| R7.4 | `apps/api/test/spec/api-contract.test.ts` | removing required fields detected as breaking | Contract |
| R7.5 | `apps/api/test/spec/api-contract.test.ts` | type changes on required fields detected | Contract |
| R7.6 | `apps/api/test/spec/api-contract.test.ts` | cross-fixture consistency (eSIM ∈ order sims) | Contract |

### R8 — State Machines

| Req ID | Test File | Test Case | Layer |
|--------|-----------|-----------|-------|
| R8.1 | `apps/api/test/state/oauth-lifecycle.test.ts` | full OAuth lifecycle + 5×5 matrix | State |
| R8.2 | `apps/api/test/state/order-lifecycle.test.ts` | full order lifecycle + 6×6 matrix | State |
| R8.3 | `apps/api/test/state/esim-lifecycle.test.ts` | full eSIM lifecycle + 5×5 matrix | State |
| R8.4 | `apps/api/test/state/full-flow-lifecycle.test.ts` | full flow lifecycle + token expiry | State |
| R8.5 | All 4 state test files | error states, retry, reset, recovery | State |
| R8.6 | All 4 state test files | invalid transitions are no-ops | State |

## Test Count Summary

| Layer | Files | Tests |
|-------|-------|-------|
| API Unit | 4 | 72 |
| API Contract (fixture) | 1 | 62 |
| API Contract (live) | 1 | 7 |
| API State Machine | 4 | 157 |
| API Integration (live) | 1 | 14 |
| Web Unit | 4 | 85 |
| Web E2E (Playwright) | 1 | 17 steps |
| Web Accessibility (Playwright) | 1 | 10 steps |
| Web Performance (Playwright) | 1 | 6 steps |
| **Total** | **18** | **430+** |
