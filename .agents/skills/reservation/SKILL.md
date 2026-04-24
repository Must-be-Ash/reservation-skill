---
name: reservation
description: >-
  Book restaurant reservations on Resy via agentres.dev using x402 USDC payments
  on Base from the awal agent wallet. Use when the user asks to book a table,
  make a reservation, find a restaurant, check availability, list upcoming
  reservations, or cancel a booking. Covers phrases like "book dinner", "find a
  restaurant", "table for 2", "cancel my reservation", or "/reservation".
---

Read fully before acting. Every rule is load-bearing.

## Prerequisites

Before any reservation operation, verify wallet readiness:

```bash
npx awal@latest status    # must show authenticated
npx awal@latest balance   # need ≥ $1 USDC
```

- Not authenticated → run the `authenticate-wallet` skill
- Insufficient funds → run the `fund` skill
- No awal installed → direct user to https://docs.cdp.coinbase.com/agentic-wallet/welcome and `npx skills add coinbase/agentic-wallet-skills`

## Auth: x402 via awal only

This skill uses **Path B only** — all requests go through awal's x402 payment flow. Never use API keys or `x-agent-key` headers.

Bookings cost **0.01 USDC** (non-refundable). All other endpoints cost **$0** (wallet signs for identity only).

## Calling agentres.dev via x402

```bash
# GET requests
npx awal@latest x402 pay 'https://agentres.dev/api/<endpoint>?<params>' -X GET --json

# POST requests
npx awal@latest x402 pay 'https://agentres.dev/api/<endpoint>' -X POST -d '<json-body>' --json
```

Rules:
- Single-quote URLs and JSON bodies to prevent shell expansion
- Every POST must include `content-type: application/json`
- Do NOT pass `--max-amount`
- Output starts with a status line, then JSON — parse from first `{`

## Workflow

Execute step-by-step. Don't preview the flow or batch questions — ask only for what the current step needs, then continue.

### Step 1 — Check account

```bash
npx awal@latest x402 pay 'https://agentres.dev/api/me' -X GET --json
```

Branch on the response:
- **200 + `resy_linked: true`** → jump to the user's request (search, book, list, cancel)
- **200 + `resy_linked: false`** → jump to **Link Resy**
- **401** → no agentres account for this wallet. Jump to **Create account**

### Step 2 — Create account (first-time only)

Only if `/api/me` returned 401.

Check Claude auto-memory for saved user email (`reservation_user.md`). If not found, ask: *"What email should I register your agentres account under?"*

```bash
npx awal@latest x402 pay 'https://agentres.dev/api/account' -X POST -d '{"email":"user@example.com"}' --json
```

Then continue to **Link Resy**.

### Step 3 — Link Resy (one-time setup)

Only if `resy_linked: false`.

Ask: *"What email is your **Resy.com** account under?"* (their existing Resy login, not a new email)

```bash
npx awal@latest x402 pay 'https://agentres.dev/api/link-resy' -X POST -d '{"em_address":"user@resy.com"}' --json
```

Tell the user to check their email for a 6-digit code from Resy. Then:

```bash
npx awal@latest x402 pay 'https://agentres.dev/api/link-resy' -X POST -d '{"em_address":"user@resy.com","code":"123456"}' --json
```

Field is **`em_address`** (not `email`). Code is a string.

### Search

```bash
npx awal@latest x402 pay 'https://agentres.dev/api/search?query=sushi&city=nyc' -X GET --json
```

`query` required. `city` optional (default `nyc`). Present results with name, neighborhood, cuisine, and rating.

### Availability

```bash
npx awal@latest x402 pay 'https://agentres.dev/api/availability?venue_id=59237&party_size=2&day=2026-04-25' -X GET --json
```

All three params required. `day` is `YYYY-MM-DD`. Present available slots as a table, ask which to book, remember the `config_id`.

**"Anything open in X at 8pm"** queries: search the area, call availability for every venue **in parallel**, filter ±30min, present as a table.

### Book (explicit confirmation required)

**Always fetch fresh availability immediately before booking** — `config_id` expires.

Show a summary and wait for explicit "yes":

> **Please confirm:** Book **[Venue]** on **[Date]** at **[Time]** for **[N]** people?
> This will charge **0.01 USDC** (non-refundable). Reply "yes" to confirm.

Only "yes" / "confirm" / "book it" / "go ahead" count. Anything ambiguous → clarify, do not book.

```bash
npx awal@latest x402 pay 'https://agentres.dev/api/book' -X POST \
  -d '{"venue_id":"59237","config_id":"<from availability>","party_size":2,"day":"2026-04-25"}' --json
```

- `venue_id`: string
- `config_id`: copy verbatim from availability response
- `party_size`: JSON number, NOT a string
- `day`: `YYYY-MM-DD` string

### List reservations

```bash
npx awal@latest x402 pay 'https://agentres.dev/api/reservations?type=upcoming&limit=10' -X GET --json
```

`type`: `upcoming` (default) | `past`. `limit`: 1–50. `offset`: ≥ 0.

### Cancel

1. Fetch upcoming reservations to find `resy_token`
2. Match the user's description; clarify if ambiguous
3. Show a summary and wait for explicit "yes"
4. ```bash
   npx awal@latest x402 pay 'https://agentres.dev/api/cancel' -X POST -d '{"resy_token":"<TOKEN>"}' --json
   ```

Never guess or reuse `resy_token` across conversations.

## Memory

### User profile

Check Claude auto-memory for `reservation_user.md` on first use. If not found, save after account creation:

```markdown
---
name: reservation-user-profile
description: User profile for Resy reservation skill (agentres.dev)
type: user
---
Name: [name]
Email: [agentres account email]
Resy email: [resy.com login email]
Phone: [phone number]
```

On subsequent uses, load silently — don't ask for info already saved.

## Rules

1. **x402 via awal only.** Never use API keys or `x-agent-key` headers.
2. **Always start with `/api/me`** to check account status.
3. **Always use `/api/search`** to get `venue_id` — never guess.
4. **Always fetch fresh `/api/availability`** immediately before `/api/book` — `config_id` expires.
5. **Dates are `YYYY-MM-DD`.** `party_size` is a JSON number.
6. **Booking and cancelling require a summary + explicit "yes".**
7. **Booking fee (0.01 USDC) is non-refundable** — disclose at booking confirmation, not at cancel.
8. **Errors return `{ error: { code, message, retryable, next_step } }`** — follow `next_step`.
9. **One step, one ask.** Don't announce future steps or request info you won't use this turn.

## Reference

For full API schemas, see `references/agentres-api.md`.
