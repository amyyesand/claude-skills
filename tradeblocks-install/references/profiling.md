# Strategy Profiling in TradeBlocks

Read this file when the health check flags missing profiles, or when the user asks about
profiling. Walk the user through it conversationally — one strategy at a time.

---

## What Is a Profile, and Why Does It Matter?

A **profile** is a short description of a strategy stored in TradeBlocks: what type of trade
it is, what Greek exposure it has, what market conditions it's designed for.

Without profiles, TradeBlocks can analyze raw performance numbers (P&L, win rate, drawdown).
With profiles, it can answer deeper questions:
- Is this strategy trading in the market conditions it was built for?
- Is it generating the Greek exposure it's supposed to?
- Where does the portfolio have gaps or overlaps in market coverage?

Think of it like adding a label to each folder — once labeled, TradeBlocks knows what's
supposed to be inside.

---

## Step 1: Identify the Strategies to Profile

Before asking the user anything, check whether the block has a populated Strategy column.

Call `get_statistics` or examine the block data to see if strategy names are present.

**If the Strategy column has values (portfolio CSV):**
The block contains multiple named strategies. List the unique strategy names you find and
say something like:
> "I can see this block contains [N] strategies: [list them]. I'll walk you through
> creating a profile for each one — let's start with [first strategy name]."

**If the Strategy column is empty (single strategy CSV):**
There's only one strategy in the block, but it has no name. Ask:
> "This looks like a single strategy backtest. What would you like to call it?
> Something short that describes what it does — for example, 'Iron Condor SPX' or
> 'Double Calendar 5/7 DTE'."
Use whatever name they give as the `strategyName` when calling `profile_strategy`.

---

## Step 2: Profile Each Strategy

Work through strategies one at a time. For each one, ask these questions conversationally
— not as a list, but as a natural back-and-forth:

**Structure type** — What kind of option trade is it?
Examples: iron condor, calendar spread, double calendar, vertical spread, butterfly,
straddle, strangle, reverse iron condor. If they're not sure, have them describe it
and help identify it from the legs.

**Greek bias** — What's the primary exposure?
- `theta_positive` — selling premium, collecting time decay
- `vega_negative` — profits when implied volatility drops
- `delta_neutral` — not betting on market direction
- `delta_positive` / `delta_negative` — has a directional bias

**Underlying** — What ticker does it trade? SPX, SPY, QQQ, GLD, etc.

**Expected VIX regimes** — What volatility environment is it designed for?
- `very_low` = VIX below 13
- `low` = VIX 13–16
- `below_avg` = VIX 16–20
- `above_avg` = VIX 20–25
- `high` = VIX 25–30
- `extreme` = VIX 30+

Once you have these four basics, call `profile_strategy` to store it. You don't need the
advanced fields (legs, entry filters, exit rules, position sizing) for the tutorial —
those can be added later.

A typical exchange looks like:
> Claude: "Let's profile your 'DC_923_baseline' strategy. From the legs in the CSV it
>   looks like a double calendar — does that sound right?"
> User: "Yes, it's a double calendar on SPX."
> Claude: "Got it. Is it primarily collecting theta — time decay — or more of a volatility play?"
> User: "Theta, mainly."
> Claude: "And what VIX range do you target? Low vol, normal, high, or does it trade in
>   any conditions?"
> User: "Low to normal — I avoid high VIX."
> Claude: [calls profile_strategy with structureType="double_calendar", greeksBias="theta_positive",
>   underlying="SPX", expectedRegimes=["very_low","low","below_avg"]]
> Claude: "Profile saved! Ready to do the next one?"

---

## Step 3: What Gets Unlocked After Profiling

Once at least one strategy is profiled, the health check will run these previously-skipped checks:

- **Regime coverage** — Are strategies covering a useful range of market conditions, or
  clustered in one regime?
- **Day coverage** — Is there redundant exposure on certain days of the week?
- **Concentration risk** — Is one strategy taking up too much of the portfolio's Greek exposure?
- **Correlation risk** — Are profiled strategies with similar structures also correlated in P&L?
- **Scaling alignment** — Does the backtest position sizing match what's being traded live?

The `analyze_structure_fit` tool also becomes available — it checks whether a strategy is
actually winning and losing in the regimes it was designed for.

---

## Tips

**For portfolios with many strategies:** Start with the 2–3 most important ones rather than
profiling everything at once. The most-traded or highest-P&L strategies are good candidates.

**If they don't know the structure type:** Look at the Legs column in the CSV — the leg
descriptions usually make it clear. For example, two puts and two calls at different
expirations = double calendar or iron condor.

**If a strategy appears in multiple blocks:** Use `get_strategy_profile` to retrieve the
existing profile and copy it to the new block, updating only position sizing if needed.
Don't re-ask the user for all the same information.
