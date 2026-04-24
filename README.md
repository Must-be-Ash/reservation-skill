# reservation-skill

Book restaurant reservations on [Resy](https://resy.com) via [agentres.dev](https://agentres.dev) — payments settle automatically in USDC on Base from the user's [awal](https://www.npmjs.com/package/awal) agent wallet.

## Install

```bash
npx skills add Must-be-Ash/reservation-skill
```

## Prerequisites

1. **awal agent wallet** — install wallet skills first:
   ```bash
   npx skills add coinbase/agentic-wallet-skills
   ```
   This ships `authenticate-wallet` and `fund` skills.

2. **Authenticate** — confirm wallet is ready:
   ```bash
   npx awal@latest status
   ```

3. **Fund** — ensure ≥ $1 USDC balance:
   ```bash
   npx awal@latest balance
   ```

## Usage

Just say what you want:

- "Book a table for 2 at a sushi place in NYC tonight"
- "Find Italian restaurants in SF"
- "What's available at Sushi Nakazawa tomorrow?"
- "Cancel my reservation at Carbone"
- "Show my upcoming reservations"
- "/reservation"

## What it supports

| Feature | Details |
|---------|---------|
| **Search** | Find restaurants by cuisine, name, or type in any supported city. |
| **Availability** | Check open time slots for any date and party size. |
| **Book** | Reserve a table with explicit confirmation. Costs 0.01 USDC. |
| **List** | View upcoming or past reservations. |
| **Cancel** | Cancel a reservation with explicit confirmation. |
| **Memory** | Your name, email, and phone saved on first use — never asked again. |

## How it works

1. You tell the agent what you want (search, book, cancel, etc.)
2. Agent checks your agentres account (creates one on first use)
3. Agent links your Resy.com account if needed (one-time, uses email verification code)
4. Agent searches, checks availability, and presents options
5. You confirm the booking — 0.01 USDC settles on Base via awal
6. Agent saves your profile for future sessions

## Cost

- **Booking:** 0.01 USDC per reservation (non-refundable)
- **Everything else:** Free (search, availability, list, cancel)

## Skill contents

```
plugins/reservation/skills/reservation/
├── SKILL.md                        # Workflow and instructions
└── references/
    └── agentres-api.md             # Endpoint payloads and API details
```

## License

MIT
