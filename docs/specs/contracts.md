# API Contract Schemas

> Canonical Zod schemas defining the shape of every Airalo Partner API response. All schemas live in `packages/shared/src/schemas/airalo-api.ts`.

## Success Responses

### Token Response — `airaloTokenResponseSchema`

```typescript
z.object({
  data: z.object({
    access_token: z.string().min(1),
    token_type:   z.string(),
    expires_in:   z.number().int().positive(),
  }),
  meta: z.object({ message: z.string() }),
})
```

**Endpoint:** `POST /token`
**Grant type:** `client_credentials`

### Order Response — `airaloOrderResponseSchema`

```typescript
z.object({
  data: z.object({
    id:         z.number(),
    code:       z.string(),
    currency:   z.string(),
    package_id: z.string(),
    quantity:   z.number(),
    type:       z.string(),
    esim_type:  z.string().optional(),
    validity:   z.number().optional(),
    price:      z.number(),
    created_at: z.string(),
    sims: z.array(airaloSimSchema).min(1),
  }),
  meta: z.object({ message: z.string() }),
})
```

### SIM Entry — `airaloSimSchema`

```typescript
z.object({
  id:          z.number(),
  iccid:       z.string(),
  lpa:         z.string(),
  qrcode:      z.string().optional(),
  qrcode_url:  z.string().optional(),
  matching_id: z.string(),
  apn_type:    z.string().optional(),
  is_roaming:  z.boolean(),
  created_at:  z.string(),
})
```

### eSIM Response — `airaloEsimResponseSchema`

```typescript
z.object({
  data: z.object({
    id:          z.number(),
    iccid:       z.string(),
    lpa:         z.string(),
    qrcode:      z.string().optional(),
    qrcode_url:  z.string().optional(),
    matching_id: z.string(),
    apn_type:    z.string().optional(),
    is_roaming:  z.boolean(),
    created_at:  z.string(),
    simable:     airaloSimableSchema.optional(),
  }),
  meta: z.object({ message: z.string() }),
})
```

### Simable Reference — `airaloSimableSchema`

```typescript
z.object({
  id:         z.number(),
  code:       z.string().optional(),
  package_id: z.string(),
  quantity:   z.number(),
  type:       z.string(),
  esim_type:  z.string().optional(),
  validity:   z.number().optional(),
  price:      z.number(),
  currency:   z.string(),
})
```

## Error Responses

### Validation Error — `airaloValidationErrorSchema`

```typescript
z.object({
  data: z.record(z.string(), z.string()),
  meta: z.object({ message: z.string() }),
})
```

**When:** Missing/invalid fields in request body (e.g., bad `client_id`).

### Auth Error — `airaloAuthErrorSchema`

```typescript
z.object({
  data: z.object({ message: z.string() }),
  meta: z.object({ message: z.string() }),
})
```

**When:** Invalid or expired Bearer token.

### Field Error — `airaloFieldErrorSchema`

```typescript
z.object({
  data: z.record(z.string(), z.union([z.string(), z.array(z.string())])),
  meta: z.object({ message: z.string() }),
})
```

**When:** Request validation failures with multiple field-level errors.

### Reason Error — `airaloReasonErrorSchema`

```typescript
z.object({
  code:   z.number(),
  reason: z.string(),
})
```

**When:** Business logic rejection (e.g., invalid `package_id`).

### Not Found Error — `airaloNotFoundErrorSchema`

```typescript
z.object({
  data: z.array(z.unknown()).max(0),
  meta: z.object({ message: z.string() }),
})
```

**When:** Resource not found (e.g., invalid iccid → `GET /sims/{iccid}`).

## Design Decisions

1. **Zod default strip mode** — Unknown fields are silently dropped. This makes schemas forward-compatible: new API fields won't break existing consumers.

2. **`.optional()` on evolving fields** — Fields like `esim_type`, `validity`, `qrcode` are optional because they weren't always present in the API. This prevents false-positive schema failures if the API removes them.

3. **`sims.min(1)`** — An order with zero SIMs is invalid by definition. The schema enforces this at parse time rather than relying on application logic.

4. **`access_token.min(1)`** — An empty access token would pass authentication headers but fail authorization. The schema catches this early.

5. **`expires_in.int().positive()`** — Eliminates fractional/zero/negative TTL values at the schema boundary.

6. **Multiple error shapes** — The Airalo API returns different error structures depending on the endpoint and error type. Rather than a single union, each shape has its own schema. Tests validate that the appropriate error schema matches each failure mode.
