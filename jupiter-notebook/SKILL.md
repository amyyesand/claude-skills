---
name: jupyter-notebook
description: >
  Use this skill whenever Amy asks to build a new Jupyter notebook, start a new
  backtest analysis, add a cell to an existing notebook, or work through
  Option Omega CSV data in Python. Triggers include: "start a notebook",
  "new notebook", "add a cell", "write a cell", "next cell", "help me analyze
  this backtest", or any session that begins with a SESSION CONTEXT block.
  Also use when Amy pastes printed output from a notebook cell and asks for
  interpretation or the next step. This skill governs all notebook structure,
  cell conventions, and the working rhythm for iterative analysis sessions.
---

# Jupyter Notebook Skill

Amy analyzes options backtests from Option Omega in Jupyter notebooks inside
IntelliJ/PyCharm. This skill governs how to open a session, build notebook
cells, and work iteratively with her through analysis.

---

## Opening a New Notebook Session

Before writing any code, do these steps in order:

1. **Ask for the strategy description.**
   > "Give me a brief description of the strategy — what it is, how it's
   > structured, and what you're trying to learn from this backtest."
   Wait for the answer before proceeding.

2. **Confirm file paths.** Amy will provide these upfront. If she hasn't,
   ask: "What's the file path (or paths) for this backtest?"

3. **Generate the scaffold** — title cell, load cell, session summary cell,
   cleanup cell — in that order, one cell at a time.

4. **Pause after the load cell.** Ask Amy to run it and paste the output.
   Interpret the output before writing any analysis cells.

---

## Notebook Scaffold (in order)

### 1. Title Cell (markdown)
```markdown
# Strategy Name — Brief Description
```
Use a single `#`. Keep it to one line.

### 2. Setup Section Header + Load Cell
Always emit the header markdown cell immediately before the load cell:

```markdown
# 1. Setup
Imports, file paths, standard derived columns, and plot defaults.
```

Standard load cell template — always include all of the following:

```python
# region Load
from pathlib import Path
import pandas as pd
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
# Optional — uncomment and add to this cell if needed for the session:
# import matplotlib.ticker as mticker  # axis tick formatting (dollar amounts, %)
# import matplotlib.dates as mdates    # date axis formatting on time series charts
# import json                          # reading short_weeks.json or other reference files
# import duckdb                        # when using the market database

# ── Plot defaults ─────────────────────────────────────────────────────────────
matplotlib.rcParams.update({
    "figure.facecolor": "#f9f9f9",
    "axes.facecolor":   "#ffffff",
    "axes.grid":        True,
    "grid.alpha":       0.4,
    "font.size":        11,
})

# ← change these
BASE = Path("/Users/amythinks/PythonProjects/StrategyNotebooks/backtests/")
FILE = BASE / "your_file.csv"

# Load
df = pd.read_csv(FILE, encoding="utf-8-sig")

# Drop columns that are never needed
df.drop(columns=["Underlying", "Strategy", "No. of Contracts"],
        errors="ignore", inplace=True)

# Parse dates
df["Date Opened"] = pd.to_datetime(df["Date Opened"])
df["Date Closed"] = pd.to_datetime(df["Date Closed"])

# Derived columns
df["Year"]        = df["Date Opened"].dt.year
df["Month"]       = df["Date Opened"].dt.month
df["Day of Week"] = df["Date Opened"].dt.day_name()
df["VIX Change"]  = df["Closing VIX"] - df["Opening VIX"]

# Cumulative P/L and drawdown
df["Cum P/L"]    = df["P/L"].cumsum()
df["Drawdown"]   = df["Cum P/L"] - df["Cum P/L"].cummax()

# Money column parser — use when a column contains $, commas, or (negatives)
def _parse_money(series):
    return (
        series.astype(str)
              .str.replace(r"[\$,]", "", regex=True)
              .str.replace(r"\((.+)\)", r"-\1", regex=True)
              .str.strip()
              .replace("", pd.NA)
              .astype(float)
    )

# Optional columns check
OPTIONAL_COLS = ["Opening S/L Ratio", "Closing S/L Ratio"]
missing = [c for c in OPTIONAL_COLS if c not in df.columns]
if missing:
    print(f"⚠️  Optional columns not found: {missing}")
    print("    → Re-export with these columns if needed, or say 'not needed'.")

print("Load complete.")
print(f"Rows: {len(df)}")
print(f"Columns: {list(df.columns)}")
# endregion
```

**If multiple files:** load each into its own DataFrame (e.g., `df_calls`,
`df_puts`) and note the naming clearly with `# ← change this` comments.

**Legs column:** present in most exports, do not parse on load. Parse in a
dedicated cell only when Amy needs it.

### 3. Session Summary Section Header + Cell
Always emit the header markdown cell immediately before the session summary:

```markdown
# 2. Session Summary
Run this after loading. Paste the output into Claude to establish context for the session.
```

```python
# region Session Summary
THIS_SESSION = "describe what you want to explore today"  # ← edit before running

_pl      = df["P/L"]
_winners = _pl[_pl > 0]
_losers  = _pl[_pl < 0]
_pf      = _winners.sum() / abs(_losers.sum()) if len(_losers) else float("inf")

print(f"""
=== SESSION CONTEXT ===
Date        : {pd.Timestamp.now().strftime('%Y-%m-%d')}
Source      : {FILE.name}
Rows        : {len(df)}
Date range  : {df['Date Opened'].min().date()} → {df['Date Opened'].max().date()}
Columns     : {list(df.columns)}

── Key Stats ──
Win rate    : {len(_winners)/len(_pl)*100:.1f}%
Total P/L   : ${_pl.sum():,.2f}
Avg P/L     : ${_pl.mean():,.2f}
Avg win     : ${_winners.mean():,.2f}
Avg loss    : ${_losers.mean():,.2f}
Prof factor : {_pf:.2f}
Max drawdown: ${df['Drawdown'].min():,.2f}

── This Session ──
{THIS_SESSION}
=======================
""")
# endregion
```

### 4. Analysis Section Header
Always emit this markdown cell before the first analysis cell. Subsequent
analysis cells go directly beneath it — no additional top-level headers needed
unless starting a clearly distinct major section.

```markdown
# 3. Analysis
Each subsection below is one analysis. Run cells individually and paste output into Claude.
```

### 5. Cleanup Section Header + Cell
Always emit the header markdown cell immediately before the cleanup cell:

```markdown
# 4. Cleanup
Close any open database connections. Run this at the end of every session.
```

Always include the cleanup cell, even if there are no database connections.

```python
# region Cleanup
# No database connections in this notebook.
print("Done.")
# endregion
```

---

## Analysis Cells — Working Rhythm

After the scaffold is built and Amy has pasted the load cell output:

- **Interpret the output first** — note row count, date range, any warnings,
  anything unexpected. Do this in plain English before writing the next cell.
- **Write one cell at a time.** Never write two analysis cells in a row without
  Amy running the first and pasting output.
- **Never assume column names or data shape** that haven't appeared in pasted
  output. If unsure, write an inspection cell first and wait.
- **If output is unexpected**, say so before proceeding. Ask one clarifying
  question if needed.

---

## Cell Conventions — No Exceptions

These rules apply to every cell, including one-liners, inspection cells,
and cleanup cells.

| Rule | Detail |
|------|--------|
| `# region` / `# endregion` | Every code cell, no exceptions. Forgetting this on short cells is a known failure mode — check every cell before delivering it. |
| Single `#` on all markdown cells | Never use `##` or deeper — it breaks collapsibility in PyCharm/IntelliJ |
| `print()` only | Never use `display()` or rely on Jupyter auto-output |
| Named aggregation | `Count=('P/L', 'count')` not dict-style `.agg({'P/L': 'count'})` |
| Imports in load cell only | Never re-import in analysis cells. When a new import is needed mid-session (e.g. `mticker`, `mdates`, `json`, `duckdb`), flag it explicitly: "Add this to your load cell imports" and uncomment the relevant line from the optional block. |
| Changeable things at top | File paths, thresholds, table names — at the very top with `# ← change this` |
| One cell at a time | Clearly labeled, ready to copy-paste |

---

## Column Reference

### Always present in OO exports
`Date Opened, Date Closed, Time Opened, Time Closed, P/L, P/L%,
Gap, Movement, Reason For Close, Opening Price, Premium,
Max Profit, Max Loss, Opening VIX, Closing VIX, Legs`

### Always derived on load
`Year, Month, Day of Week, VIX Change, Cum P/L, Drawdown`

### Always dropped on load
`Underlying, Strategy, No. of Contracts`

### Conditionally present (check on load, warn if missing)
`Opening S/L Ratio, Closing S/L Ratio`

### Parse on demand only
`Legs` — free text, strategy-specific. Only parse when Amy asks for
strike or leg price analysis. Write a dedicated cell at that point.

---

## Market Database

A DuckDB database is available for enriching backtest analysis with SPX and
VIX price history. Use it when Amy needs context beyond what's in the OO
export — regime analysis, intraday price behavior, VIX term structure, etc.

**Path:** `/Users/amythinks/PythonProjects/SPXBacktesting/signals/market.duckdb`

**Important:** All date columns are `VARCHAR`, not `DATE` type. Always cast
when joining or filtering: `CAST(date AS DATE)`.

| Table | Rows | Contents |
|-------|------|----------|
| `spx_daily` | 8,366 | SPX daily OHLCV + trade count |
| `spx_intraday` | 377,396 | SPX intraday OHLC — columns: `date`, `time_str` |
| `vix_daily` | 13,721 | VIX daily OHLC by symbol |
| `vix_intraday` | 1,010,959 | VIX intraday OHLC by symbol — columns: `date`, `time_str` |
| `straddle_data` | 375,376 | ATM straddle bid/ask/price for calls & puts + `minutes_to_expiry` |
| `vx_futures` | 28,174 | VX futures daily settle with volume, OI, and `expiry_date` |

**Standard connection snippet** — add to load cell when database is needed:

```python
import duckdb  # ← add to imports in load cell

DB_PATH = "/Users/amythinks/PythonProjects/SPXBacktesting/signals/market.duckdb"
con = duckdb.connect(DB_PATH, read_only=True)
```

**Cleanup cell** — always close the connection when database is used:

```python
# region Cleanup
try:
    con.close()
    print("Database connection closed.")
except:
    pass
# endregion
```

Only add the database connection when Amy indicates she needs it. Do not
include it in every notebook by default.

---

## File Paths

```
Base:       /Users/amythinks/PythonProjects/StrategyNotebooks/
Notebooks:  .../notebooks/[strategy]/
Backtests:  .../backtests/[strategy]/
Scripts:    .../scripts/
```

---

## What NOT to Do

- Don't generate the full notebook at once — scaffold only, then one analysis
  cell at a time
- Don't write a cell that depends on a column not yet confirmed in pasted output
- Don't use `##` or deeper in markdown cells
- Don't omit `# region` / `# endregion` — this is the most common mistake,
  check every cell
- Don't use `display()` or dict-style `.agg()`
- Don't re-import libraries outside the load cell
- Don't ask multiple questions at once — one question at a time
