# Edge Cases & Boundary Conditions

> Catalog of edge cases covered by the test suite. Referenced from `requirements.md` and `tasks.md`.

## EC-A — API Error Responses

### EC-A1 — Authentication Errors

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-A1.1 | Invalid `client_id` + valid `client_secret` | 422 with `data.client_id` field error | `authService.test.ts`, `live-api.test.ts` |
| EC-A1.2 | Valid `client_id` + invalid `client_secret` | 422 with `data.client_secret` field error | `authService.test.ts`, `live-api.test.ts` |
| EC-A1.3 | Both credentials empty strings | 422 or 401 | `live-api.test.ts` |
| EC-A1.4 | Server returns 500 on `/token` | Axios error propagated | `authService.test.ts` |

### EC-A2 — Order Errors

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-A2.1 | `package_id` does not exist | 422 with reason error | `orderService.test.ts`, `live-api.test.ts` |
| EC-A2.2 | `quantity = 0` | 422 validation error | `orderService.test.ts`, `live-api.test.ts` |
| EC-A2.3 | `quantity < 0` (negative) | 422 validation error | `live-api.test.ts` |
| EC-A2.4 | Empty `package_id` | 422 validation error | `live-api.test.ts` |
| EC-A2.5 | Expired or invalid Bearer token | 401 auth error | `orderService.test.ts`, `live-api.test.ts` |
| EC-A2.6 | Empty Bearer token | 401 or 422 | `live-api.test.ts` |
| EC-A2.7 | Server returns 500/502/503 | Error propagated | `airalo-flow.test.ts` |
| EC-A2.8 | Server returns 429 (rate limit) | Error propagated, retry eligible | `airalo-flow.test.ts` |

### EC-A3 — eSIM Errors

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-A3.1 | Invalid/non-existent iccid | 404 or 422 | `esimService.test.ts`, `live-api.test.ts` |
| EC-A3.2 | Missing Bearer token | 401 | `esimService.test.ts`, `live-api.test.ts` |
| EC-A3.3 | One eSIM fetch fails, others succeed (partial failure) | First error propagated via Promise.all | `airalo-flow.test.ts` |
| EC-A3.4 | Numeric-only iccid value | URL construction works correctly | `esimService.test.ts` |

## EC-N — Network-Level Errors

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-N1 | ECONNREFUSED from createOrder | Network error propagated | `airalo-flow.test.ts` |
| EC-N2 | ETIMEDOUT from createOrder | Network error propagated | `airalo-flow.test.ts` |
| EC-N3 | ECONNRESET from createOrder | Network error propagated | `airalo-flow.test.ts` |
| EC-N4 | ENOTFOUND from createOrder | Network error propagated | `airalo-flow.test.ts` |
| EC-N5 | ETIMEDOUT from fetchEsim (post-order) | Network error propagated | `airalo-flow.test.ts` |

## EC-S — Schema Edge Cases

### EC-S1 — Backward Compatibility

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-S1.1 | Extra unknown fields in token response | Stripped by Zod, parse succeeds | `api-contract.test.ts` |
| EC-S1.2 | Extra unknown fields in order response | Stripped, parse succeeds | `api-contract.test.ts` |
| EC-S1.3 | Extra unknown fields in eSIM response | Stripped, parse succeeds | `api-contract.test.ts` |
| EC-S1.4 | Minimum viable token payload (only required fields) | Parse succeeds | `api-contract.test.ts` |
| EC-S1.5 | Minimum viable order payload | Parse succeeds | `api-contract.test.ts` |
| EC-S1.6 | Minimum viable eSIM payload | Parse succeeds | `api-contract.test.ts` |

### EC-S2 — Breaking Changes

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-S2.1 | Remove `access_token` from token response | ZodError — missing required field | `api-contract.test.ts` |
| EC-S2.2 | Remove `package_id` from order response | ZodError | `api-contract.test.ts` |
| EC-S2.3 | Remove `iccid` from eSIM response | ZodError | `api-contract.test.ts` |
| EC-S2.4 | Change `expires_in` from number to string | ZodError — type changed | `api-contract.test.ts` |
| EC-S2.5 | Change `quantity` from number to string | ZodError — type changed | `api-contract.test.ts` |
| EC-S2.6 | Change `is_roaming` from boolean to string | ZodError — type changed | `api-contract.test.ts` |
| EC-S2.7 | Empty sims array (min: 1 violated) | ZodError | `orderService.test.ts`, `api-contract.test.ts` |

### EC-S3 — Cross-Fixture Consistency

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-S3.1 | Every eSIM fixture iccid belongs to the order fixture sims | Match found | `api-contract.test.ts` |
| EC-S3.2 | Order fixture sim count = 6 (quantity) | Exact match | `api-contract.test.ts` |
| EC-S3.3 | eSIM fixture simable.package_id = order fixture package_id | Consistent | `api-contract.test.ts` |
| EC-S3.4 | All order iccids are unique | Set size = array length | `api-contract.test.ts` |

### EC-S4 — Negative Schema Cross-Validation

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-S4.1 | Token payload → Order schema | ZodError (wrong schema) | `api-contract.test.ts` |
| EC-S4.2 | Order payload → Token schema | ZodError | `api-contract.test.ts` |
| EC-S4.3 | eSIM payload → Order schema | ZodError | `api-contract.test.ts` |
| EC-S4.4 | Complete garbage object → any schema | ZodError | `api-contract.test.ts` |
| EC-S4.5 | Error payload → success schema | ZodError | `api-contract.test.ts` |

## EC-SM — State Machine Edge Cases

### EC-SM1 — Token Machine

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-SM1.1 | Missing credentials → AUTHENTICATE | NO_TOKEN → FAILED (guard) | `oauth-lifecycle.test.ts` |
| EC-SM1.2 | FAILED → RETRY (under max) | FAILED → REQUESTING (retryCount incremented) | `oauth-lifecycle.test.ts` |
| EC-SM1.3 | FAILED → RETRY (at max retries) | FAILED → FAILED (stays, retryCount unchanged) | `oauth-lifecycle.test.ts` |
| EC-SM1.4 | VALID_TOKEN → any non-TTL_EXPIRED event | No-op | `oauth-lifecycle.test.ts` (5×5 matrix) |

### EC-SM2 — Order Machine

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-SM2.1 | Empty token → SUBMIT_ORDER | IDLE → AUTH_FAILED (guard) | `order-lifecycle.test.ts` |
| EC-SM2.2 | Invalid input → SUBMIT_ORDER | IDLE → VALIDATION_FAILED (guard) | `order-lifecycle.test.ts` |
| EC-SM2.3 | Terminal state + RESET | → IDLE (context cleared) | `order-lifecycle.test.ts` |
| EC-SM2.4 | CREATING → non-response event | No-op | `order-lifecycle.test.ts` (6×6 matrix) |

### EC-SM3 — eSIM Machine

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-SM3.1 | Empty iccid → FETCH | UNRESOLVED → ERROR (guard) | `esim-lifecycle.test.ts` |
| EC-SM3.2 | RESOLVED → any event | No-op (terminal) | `esim-lifecycle.test.ts` |
| EC-SM3.3 | NOT_FOUND → RETRY | NOT_FOUND → FETCHING | `esim-lifecycle.test.ts` |
| EC-SM3.4 | ERROR → RETRY | ERROR → FETCHING | `esim-lifecycle.test.ts` |

### EC-SM4 — Flow Machine

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-SM4.1 | Token expires mid-flow | AUTHENTICATED → RE_AUTH_REQUIRED | `full-flow-lifecycle.test.ts` |
| EC-SM4.2 | Re-auth succeeds | RE_AUTH_REQUIRED → AUTHENTICATED | `full-flow-lifecycle.test.ts` |
| EC-SM4.3 | Re-auth fails | RE_AUTH_REQUIRED → FAILED | `full-flow-lifecycle.test.ts` |
| EC-SM4.4 | COMPLETE is terminal | COMPLETE → COMPLETE (no-op) | `full-flow-lifecycle.test.ts` |
| EC-SM4.5 | FAILED is terminal | FAILED → FAILED (no-op) | `full-flow-lifecycle.test.ts` |

## EC-W — Web UI Edge Cases

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-W1 | Search with non-existent country | Empty dropdown or no results | `airalo-flow.spec.ts` |
| EC-W2 | Case-insensitive search | Results appear regardless of casing | `airalo-flow.spec.ts` |
| EC-W3 | Cookie banner auto-dismiss | Banner closed programmatically before interactions | `airaloHomePage.ts` (POM) |
| EC-W4 | Price format normalization across currencies | Currency symbol stripping, decimal handling | `format.test.ts` |
| EC-W5 | Whitespace in price strings | Leading/trailing/inner spaces handled | `format.test.ts` |
| EC-W6 | RegExp serialization in JSON fixtures | `/pattern/flags` string → native RegExp | `regex.test.ts` |

## EC-R — Retry & Rate Limiting

| ID | Scenario | Expected | Covered By |
|----|----------|----------|------------|
| EC-R1 | HTTP 429 from live API | `isTransient()` returns true → retry with backoff | `retry.ts` + integration tests |
| EC-R2 | HTTP 5xx from live API | `isTransient()` returns true → retry | `retry.ts` + integration tests |
| EC-R3 | Network error (ECONNABORTED, etc.) | `isTransient()` returns true → retry | `retry.ts` + integration tests |
| EC-R4 | HTTP 4xx (non-429) from live API | `isTransient()` returns false → fail immediately | `retry.ts` logic |
| EC-R5 | Max retries exceeded | Throws last error after 3 attempts | `retry.ts` logic |
