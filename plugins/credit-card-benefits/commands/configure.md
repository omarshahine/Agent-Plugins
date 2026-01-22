---
description: Configure data sources and initial setup for credit card benefits tracking
argument-hint: "[--data-source ynab|csv|manual]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - AskUserQuestion
---

# Configure Credit Card Benefits Tracker

Set up data sources and perform initial configuration for tracking your credit card benefits.

## Usage

```
/credit-card-benefits:configure
```

Interactive setup that guides you through:
1. Selecting which cards you have
2. Choosing your data source(s)
3. Initial data import (12-month lookback)
4. Setting up incremental sync

## Data Source Options

### Option 1: YNAB (Recommended if you use YNAB)

Two methods available:

**A. YNAB MCP Server** (if installed)
- Uses the `mcp__ynab__*` tools if available
- No API token management needed
- Real-time transaction access

**B. YNAB API Direct**
- Requires Personal Access Token from https://app.ynab.com/settings/developer
- Token stored securely at `~/.config/credit-card-benefits/ynab-token`

### Option 2: CSV Import

- Download CSV exports directly from each credit card website
- Most flexible - works with any card
- Manual process but straightforward

### Option 3: Manual Entry

- Track benefits by manually recording usage
- No external data source required
- Good for simple tracking

## Configuration Flow

### Step 1: Select Your Cards

```
Which premium credit cards do you have?

[ ] American Express Platinum ($895/year)
[ ] Capital One Venture X ($395/year)
[ ] Chase Sapphire Reserve ($795/year)
[ ] Bank of America Alaska Atmos ($395/year)
[ ] Delta SkyMiles Reserve ($650/year)
```

### Step 2: Choose Data Source

```
How would you like to track transactions?

1. YNAB MCP Server (auto-detect if available)
2. YNAB API (requires token setup)
3. CSV Import (download from card websites)
4. Manual Entry (record usage yourself)
```

### Step 3: Data Source Setup

**For YNAB:**
- Check if MCP server is available: look for `mcp__ynab__` tools
- If not, prompt for API token
- Fetch account list and map to cards

**For CSV:**
- Explain where to download CSVs for each card:
  - Amex: Account → Statements & Activity → Download
  - Chase: Activity → Download account activity
  - Capital One: Account → Download Transactions
  - etc.

### Step 4: Initial Sync (12-Month Lookback)

For YNAB:
```
Performing initial sync (last 12 months)...

This will:
- Find annual fee posting dates → set anniversaries
- Identify past benefit transactions
- Detect statement credits received

Fetching transactions since 2025-01-22...
```

For CSV:
```
To perform initial setup, please import CSV files covering the last 12 months.

Download statements from:
- Amex Platinum: amex.com → Statements & Activity
- Chase Sapphire: chase.com → See Activity → Download

Then run:
/credit-card-benefits:import <file.csv> --card amex-platinum
```

### Step 5: Save Configuration

Store configuration in checklist.yaml:

```yaml
config:
  version: "1.0.0"
  setupComplete: true
  setupDate: 2026-01-22

  dataSource:
    primary: ynab-mcp          # or: ynab-api, csv, manual
    ynab:
      method: mcp              # or: api
      budgetId: "uuid-here"
      accountMapping:
        amex-platinum: "account-uuid"
        chase-sapphire-reserve: "account-uuid"
    csv:
      importDirectory: ~/Downloads  # Where to look for CSVs

  sync:
    initialSyncDate: 2025-01-22     # 12 months back
    lastSyncDate: 2026-01-22
    autoSync: false                  # Future: auto-sync on session start

  cards:
    enabled:
      - amex-platinum
      - chase-sapphire-reserve
      - capital-one-venture-x
    disabled:
      - alaska-atmos-summit
      - delta-reserve
```

## Initial Sync Logic

### What to Look For (12 months)

**Annual Fees:**
```
Patterns: "ANNUAL FEE", "ANNUAL MEMBERSHIP", "YEARLY FEE"
Amounts: $895 (Amex), $795 (Chase), $650 (Delta), $395 (Venture X, Alaska)

Action: Set lastAnnualFeeDate, calculate nextAnnualFeeDate
```

**Benefit Transactions:**
```
Match merchants to benefits (see import.md for patterns)
Track spending against each benefit period
```

**Statement Credits:**
```
Look for negative amounts / credits
Match to benefits by timing and amount
```

### Incremental Sync Logic

After initial setup, syncs only need to fetch from `lastSyncDate`:

```
/credit-card-benefits:sync

Fetching transactions since 2026-01-15...
Found 23 new transactions.

New benefit usage detected:
- Jan 18: LULULEMON $89 → Q1 credit
- Jan 20: UBER EATS $22 → Uber Cash (Jan)

Statement credits received:
- Jan 19: LULULEMON CREDIT $75

Update checklist? [y/n]
```

## MCP Server Detection

Check for YNAB MCP server:

```
# In agent, check available tools for mcp__ynab__ prefix
# If found, use MCP; otherwise fall back to API or CSV
```

Available YNAB MCP tools (if server installed):
- `mcp__ynab__get_budgets`
- `mcp__ynab__get_accounts`
- `mcp__ynab__get_transactions`

## Post-Setup

After configuration, the checklist is ready for:
- `/credit-card-benefits:status` - See all benefits
- `/credit-card-benefits:sync` - Pull new transactions
- `/credit-card-benefits:remind` - Get expiration alerts
