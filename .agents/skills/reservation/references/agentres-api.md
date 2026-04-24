# agentres.dev API Reference

**Base URL:** `https://agentres.dev`
**Full schemas:** `https://agentres.dev/openapi.json`
**Auth:** x402 via awal (Path B only)

## Endpoints

| Method | Path | Cost | Description |
|--------|------|------|-------------|
| GET | `/api/me` | $0 | Check account status and Resy link |
| POST | `/api/account` | $0 | Create agentres account (first-time only) |
| POST | `/api/link-resy` | $0 | Link Resy.com account (one-time setup) |
| GET | `/api/search` | $0 | Search restaurants |
| GET | `/api/availability` | $0 | Check available time slots |
| POST | `/api/book` | **$0.01** | Book a reservation |
| GET | `/api/reservations` | $0 | List reservations |
| POST | `/api/cancel` | $0 | Cancel a reservation |

## GET /api/me

Response:
```json
{ "user_id": "string", "email": "string", "resy_linked": true }
```

- 200 + `resy_linked: true` → ready to use
- 200 + `resy_linked: false` → need to link Resy
- 401 → no account, need to create one

## POST /api/account

Create a new agentres account for this wallet.

```json
{ "email": "user@example.com" }
```

## POST /api/link-resy

Step 1 — Request code:
```json
{ "em_address": "user@resy.com" }
```

Step 2 — Submit code:
```json
{ "em_address": "user@resy.com", "code": "123456" }
```

Field is `em_address` (NOT `email`). Code is a string.

## GET /api/search

```
/api/search?query=sushi&city=nyc
```

- `query` (required): search term
- `city` (optional): default `nyc`

Response:
```json
[{ "venue_id": "59237", "name": "Sushi Nakazawa", "neighborhood": "West Village", "cuisine": "Japanese", "rating": 4.8 }]
```

## GET /api/availability

```
/api/availability?venue_id=59237&party_size=2&day=2026-04-25
```

All three params required. `day` is `YYYY-MM-DD`.

Response:
```json
{
  "venue_name": "Sushi Nakazawa",
  "slots": [
    { "config_id": "abc123", "time": "7:00 PM", "type": "Dining Room" },
    { "config_id": "def456", "time": "8:30 PM", "type": "Bar" }
  ]
}
```

## POST /api/book

```json
{
  "venue_id": "59237",
  "config_id": "abc123",
  "party_size": 2,
  "day": "2026-04-25"
}
```

- `venue_id`: string
- `config_id`: string (copy verbatim from availability — it expires)
- `party_size`: JSON number (NOT string)
- `day`: `YYYY-MM-DD` string

**Costs 0.01 USDC. Non-refundable.**

Always fetch fresh availability immediately before booking.

## GET /api/reservations

```
/api/reservations?type=upcoming&limit=10
```

- `type`: `upcoming` (default) | `past`
- `limit`: 1–50
- `offset`: ≥ 0

Response includes `resy_token` needed for cancellation.

## POST /api/cancel

```json
{ "resy_token": "<TOKEN>" }
```

Get `resy_token` from `/api/reservations`. Never guess or reuse across conversations.

## Error responses

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "retryable": true,
    "next_step": "string"
  }
}
```

Always follow `next_step` from error responses.
