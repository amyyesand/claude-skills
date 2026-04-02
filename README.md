# Claude Skills for TradeBlocks

Two Claude skills that guide new users through setting up and learning TradeBlocks — an MCP server that lets Claude analyze options trading strategies through normal conversation.

---

## Start Here

If you're brand new to TradeBlocks, follow these four steps:

**1. Download the installation skill**
Click here to download: [tradeblocks-install.zip](tradeblocks-install/tradeblocks-install.zip)

**2. Install it in Claude Desktop**
- Open Claude Desktop
- Click **Customize** in the left sidebar
- Click **Skills**
- Click the **+** button and select **Upload a skill**
- Select the zip file you just downloaded

**3. Open a new chat and type:**
```
/tradeblocks-install
```

**4. Follow the guide**
Claude will walk you through everything — what TradeBlocks is, how to install it, how to get your data loaded, and how to launch the interactive tutorial when you're ready.

---

## Skills

### 1. `tradeblocks-install` — Installation Guide

Walks a brand-new user through everything they need to get started:

- What TradeBlocks is and what it does
- Installing the TradeBlocks MCP (terminal or one-click installer)
- Verifying the connection
- Getting trading data loaded
- Launching the tutorial skill

**Requirements:** No MCP needed — this skill works before anything is installed.

**Trigger:** Type `/tradeblocks-install` in Claude, or say "I want to install TradeBlocks."

[Download tradeblocks-install.zip](tradeblocks-install/tradeblocks-install.zip)

---

### 2. `tradeblocks-tutorial` — Interactive Tool Tour

Walks a connected user through TradeBlocks' core analysis tools one at a time, using their real data:

1. `list_blocks` — see what data you have
2. `get_statistics` — overall performance summary
3. `portfolio_health_check` — full diagnostic
4. `stress_test` — historical crash scenarios
5. `analyze_edge_decay` — is the strategy still working?
6. `get_correlation_matrix` — are strategies truly independent?
7. `run_monte_carlo` — simulating possible futures
8. `compare_blocks` — comparing portfolios side by side
9. `what_if_scaling` — what if I ran less (or more) of this?

**Requirements:** TradeBlocks MCP must be installed and connected, with at least one block of trade data loaded.

**Trigger:** Type `/tradeblocks-tutorial` in Claude, or say "I want to learn how to use TradeBlocks."

[Download tradeblocks-tutorial.zip](tradeblocks-tutorial/tradeblocks-tutorial.zip)

---

## How to Install a Skill

### Claude Desktop
1. Download the `.zip` file for the skill you want
2. Click **Customize** in the left sidebar
3. Click **Skills**
4. Click the **+** button and select **Upload a skill**
5. Select the zip file — it will appear in your `/` slash command menu

### Claude Code (CLI)
Unzip the skill folder into `~/.claude/skills/`:
```bash
unzip tradeblocks-install.zip -d ~/.claude/skills/
unzip tradeblocks-tutorial.zip -d ~/.claude/skills/
```

---

## About TradeBlocks

TradeBlocks is an MCP server for analyzing options trading strategies through Claude. It reads your trade history (exported as CSV from Option Omega) and makes it available for analysis through plain conversation — no spreadsheets, no coding.

For installation help, use the `tradeblocks-install` skill above.

For more on TradeBlocks, visit the [TradeBlocks GitHub repository](https://github.com/davidromeo/tradeblocks).
