# Migrating off a deprecated `@polar-sh/<framework>` adapter

How to move a project from a deprecated adapter package to the SDK-direct
recipes in [SKILL.md](../SKILL.md). This file only maps the old adapter API to
the recipes — the recipes themselves are the single source of truth for how the
endpoints work.

**Deprecated adapters** (pinned at their final `0.x`, no further releases):
`@polar-sh/hono`, `@polar-sh/express`, `@polar-sh/fastify`,
`@polar-sh/sveltekit`, `@polar-sh/astro`, `@polar-sh/remix`,
`@polar-sh/elysia`, `@polar-sh/supabase`, `@polar-sh/deno` (JSR).

**Supported adapters** (do NOT apply this file): `@polar-sh/nextjs`,
`@polar-sh/better-auth`, `@polar-sh/tanstack-start`, `@polar-sh/nuxt` —
upgrade those to `1.x` instead; see
[polarsource/polar](https://github.com/polarsource/polar).

## Procedure

1. Find adapter usage: imports from `@polar-sh/<framework>` (typically
   `Checkout`, `CustomerPortal`, `Webhooks`, sometimes `Entitlements`).
2. Replace each route with the corresponding recipe from SKILL.md, carrying the
   old config over using the mappings below.
3. `npm uninstall @polar-sh/<framework>` — keep `@polar-sh/sdk` (the recipes
   run against the `0.x` SDK the project already has; upgrading the SDK to
   `1.x` is a separate, later step).
4. Verify: typecheck/build, send a test webhook from the Polar dashboard
   (sandbox), run one test checkout.

## `Checkout({ ... })` → Recipe 1

The recipe's handler reads the same query params the adapter did
(`products`, `customerId`, `customerExternalId`, `customerEmail`, `metadata`,
…), so checkout links keep working. Map the config object:

| Adapter config | Recipe equivalent |
|---|---|
| `accessToken`, `server` | `new Polar({ accessToken, server })` client init |
| `successUrl` | `SUCCESS_URL` const. The adapter appended `?checkoutId={CHECKOUT_ID}` unless `includeCheckoutId: false` — reproduce by putting `{CHECKOUT_ID}` in the URL (or not) |
| `theme` | `THEME` const (appended to `result.url` as a query param) |
| `returnUrl` | Pass `returnUrl` to `polar.checkouts.create()` (see field table) |

The adapter resolved relative `successUrl`/`returnUrl` against the request URL;
if the project used relative URLs, resolve with `new URL(value, request.url)`.

## `CustomerPortal({ ... })` → Recipe 2

| Adapter config | Recipe equivalent |
|---|---|
| `accessToken`, `server` | client init |
| `getCustomerId(event)` | the recipe's auth-helper function; pass its result as `customerId` |
| `getExternalCustomerId(event)` | same, passed as `externalCustomerId` (the default in the recipe) |
| `returnUrl` | `RETURN_URL` const |

The adapter passed the framework's request/event object to the resolver — port
whatever it read (session, cookies, `locals`) into the auth helper.

## `Webhooks({ ... })` → Recipe 3

`webhookSecret` → `POLAR_WEBHOOK_SECRET`. Handlers:

- `onPayload(payload)` → code placed right after `validateEvent`, before the
  `switch`.
- Granular `onXxx` handlers → `case` blocks in the recipe's `switch`. The name
  rule is mechanical — strip `on`, then the event type is the PascalCase parts
  as `resource.action` in snake_case:
  - `onOrderPaid` → `"order.paid"`
  - `onCheckoutCreated` → `"checkout.created"`
  - `onBenefitGrantRevoked` → `"benefit_grant.revoked"`
  - `onCustomerStateChanged` → `"customer.state_changed"`
  - `onCustomerSeatAssigned` → `"customer_seat.assigned"`
- `entitlements` (rare) → the adapters' `Entitlements` helper reacted to
  `benefit_grant.created`/`benefit_grant.revoked`; port that logic into those
  two `case` blocks. Frozen original for reference:
  [`adapter-utils/entitlement.ts`](https://github.com/polarsource/polar-adapters/blob/main/packages/adapter-utils/src/entitlement/entitlement.ts).

**Express/Fastify gotcha:** the old adapters verified signatures against
`JSON.stringify(req.body)` and required the JSON body parser on the webhook
route. The recipes verify the **raw body** instead (more robust) — so when
migrating, switch that route's body parsing as shown in Recipe 3's framework
variations (`express.raw(...)` / `@fastify/raw-body`). Leaving `express.json()`
on the route will make verification fail.

## Response-shape notes

Adapter handlers returned `403 {"received": false}` on bad signatures,
`200 {"received": true}` on success, `400` for missing `products`, and `302`
redirects — the recipes match, so no client-side changes are needed.
