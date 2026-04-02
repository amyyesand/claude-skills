# Strategy Profiling in TradeBlocks

Read this file when the health check (or any analysis) mentions that strategy profiles are missing, or when the user asks what profiling is or how to do it. Walk the user through it conversationally — don't paste the whole thing at once.

---

## What Is a Profile, and Why Does It Matter?

When TradeBlocks runs a health check, it flags some checks as "skipped — no strategy profiles found." A **profile** is simply a short description of a strategy that you store in TradeBlocks: what type of trade it is, what Greek exposure it has, what market conditions it's designed for.

Without profiles, TradeBlocks can analyze raw performance numbers (P&L, win rate, drawdown). With profiles, it can answer deeper questions:
- Is this strategy trading in the market conditions it was built for?
- Is it generating the Greek exposure it's supposed to?
- Where does the portfolio have gaps or overlaps in market coverage?

Think of it like adding a label to each folder — once labeled, TradeBlocks knows what's supposed to be inside.

---

## What You'll Need to Know About Each Strategy

You don't need to know everything — just the basics. Claude will ask questions and fill in the profile. Here's what matters:

**Structure type** — What kind of option trade is it? Examples: iron condor, calendar spread, double calendar, vertical spread, butterfly, straddle, strangle, reverse iron condor. If you're not sure, describe it and Claude can help identify it.

**Greek bias** — What's the primary exposure? Common ones:
- `theta_positive` — you're selling premium and collecting time decay
- `vega_negative` — you profit when implied volatility drops
- `delta_neutral` — you're not betting on market direction
- `delta_positive` / `delta_negative` — you have a directional bias

**Expected VIX regimes** — What volatility environment is this strategy designed for? TradeBlocks uses these buckets:
- `very_low` = VIX below 13
- `low` = VIX 13–16
- `below_avg` = VIX 16–20
- `above_avg` = VIX 20–25
- `high` = VIX 25–30
- `extreme` = VIX 30+

**Underlying** — What ticker does it trade? SPX, SPY, QQQ, GLD, etc.

---

## How Claude Will Help

Claude will walk through each strategy with you conversationally. For a portfolio with many strategies, it makes sense to start with the most important 2–3 rather than profiling everything at once.

A typical exchange looks like:

> Claude: "Let's profile your '3/7 DC 25/20D' strategy. From the name it looks like a double calendar — does that sound right?"
> User: "Yes, it's a double calendar on SPX."
> Claude: "Got it. Is it primarily collecting theta — time decay — or is it more of a volatility play?"
> User: "Theta, mainly."
> Claude: "And what VIX environment do you target? Low vol, normal, or does it trade in any conditions?"

Claude then calls `profile_strategy` to store the result.

---

## What Gets Unlocked After Profiling

Once at least one strategy is profiled, the health check will run these previously-skipped checks:

- **Regime coverage** — Are the strategies covering a useful range of market conditions, or are they all clustered in one regime?
- **Day coverage** — Is there redundant exposure on certain days of the week?
- **Concentration risk** — Is one strategy taking up too much of the portfolio's Greek exposure?
- **Correlation risk** — Are profiled strategies with similar structures also correlated in their P&L?
- **Scaling alignment** — Does the backtest position sizing match what you're actually trading live?

The `analyze_structure_fit` tool also becomes available — it checks whether a strategy is actually winning and losing in the regimes it was designed for, or whether the real-world results tell a different story.

---

## Getting Started

When the user is ready to profile, start by listing their strategies and asking which one to tackle first. If the health check already flagged correlated pairs, those are worth profiling early since the regime tools can then show whether the correlation is structural (same trade type) or incidental.
