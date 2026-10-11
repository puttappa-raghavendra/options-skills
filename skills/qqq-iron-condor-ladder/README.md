# QQQ Iron Condor Ladder

**Ticker:** QQQ (Invesco QQQ Trust, Nasdaq-100 ETF) options
**Strategy:** weekly layered iron condors, ~30 days to expiration
**Broker:** Interactive Brokers, through the Claude IBKR connector (no scripts)

## The strategy in one paragraph

Each week the skill opens one new iron condor on QQQ about 30 days out (28 to 35 DTE
accepted), so over a month you hold a ladder of three or four condors with staggered
expirations. The short strikes sit near 15 delta. It looks at the last 5 sessions of
QQQ: on the side the market is trending toward, it moves the short strike further out
(12 delta) and widens that wing, giving the trend more room. Every position is managed
with the same mechanical exits, and every order is checked by an independent reviewer
before you approve it.

## Entry rules

- **When:** one new layer per calendar week, on Friday. Earlier in the week (Mon to Thu)
  only on a "dip day": the previous session fell 2 to 4%, QQQ is holding above the prior
  low, and VIX is at or below 35 and not up more than 10% from its prior close.
- **No entry when:** QQQ fell 4% or more in a day, VIX is below 12 or above 40, FOMC, CPI
  or a mega-cap earnings report is within the next two sessions, or the account is past
  its drawdown limit.
- **Strikes:** short call and short put at about 15 delta; trend-side short at about
  12 delta with the wing widened by `trend_wing_extra`. If the wide wing doesn't fit the
  layer's risk budget, it falls back to the base wing.
- **Sizing:** contracts = layer risk budget / max loss per condor (wing width minus
  credit, plus fees), rounded down. Total open risk across all layers is capped.

## Exit rules (checked every trading day)

| Rule | Action |
|---|---|
| Profit reaches 50% of the credit | Close |
| Loss reaches `stop_loss_mult` x the credit | Close |
| QQQ trades through a short strike | Close |
| 21 days to expiration | Close |
| Short leg reaches 0.25 delta | Watch |
| Short leg reaches 0.30 delta | Defend once: roll the untested side closer |

## Risk profiles

Set `risk_profile` in [`qqq-ic-config.json`](qqq-ic-config.json). The default is `moderate`.

| Profile | Risk per layer | Max total risk | Base wing | Trend-side extra | Stop | Drawdown pause | Half size above VIX |
|---|---|---|---|---|---|---|---|
| conservative | 5% | 15% | $5 | +$5 | 1.5x credit | 10% | 25 |
| **moderate** | 10% | 25% | $10 | +$10 | 2x credit | 15% | 30 |
| aggressive | 20% | 40% | $10 | +$10 | 2x credit | 20% | 35 |

To change a single rule for every profile, put it under `"overrides"`, for example
`"profit_target_pct": 0.4`.

## How to use it

1. Install the skill (copy this folder into your Claude skills directory) and add the
   Interactive Brokers connector in Claude (Settings > Connectors). The connector is
   required: every run first checks that it is added and signed in, and stops with a
   clear message if it isn't, rather than trading on stale or guessed numbers.
2. Ask Claude things like:
   - "Plan this Friday's QQQ condor layer."
   - "Check my open QQQ condors."
   - "Switch my condor risk profile to conservative."
3. For each new layer Claude writes a trade ticket (strikes, credit, sizing math), a
   separate reviewer subagent re-checks it against live IBKR data, and then you approve
   the order in IBKR, or enter it by hand from the ticket.

### Running it on a schedule

The skill is built to run as a daily routine in America/New_York time, so daylight
saving is handled for you:

- **9:50 ET, Mon to Fri** (20 minutes after the open): manage open layers, then check
  whether today is an entry day.
- **15:15 ET, Mon to Fri:** exit check only, reporting just when something needs action.

Scheduled runs never place orders on their own. They post reviewed tickets for you to
approve.

## Files

| File | In repo | Purpose |
|---|---|---|
| `SKILL.md` | yes | The full rules Claude follows |
| `qqq-ic-config.json` | yes | Risk profile and default values |
| `qqq-ic-ledger.json` | no (git-ignored) | Your open layers, entry credits and equity peak |
| `tickets/` | no (git-ignored) | One audit file per trade ticket |

## Disclaimer

Educational tool, not financial advice. Iron condors have defined but real maximum
losses, and a ladder can lose on several layers at once in a sharp move. Paper-trade it
first.
