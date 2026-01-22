---
description: Sync transactions from configured data source (YNAB or CSV)
argument-hint: "[--full] [--since YYYY-MM-DD]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - WebFetch
---

# Sync Transactions

Fetch new transactions from your configured data source and update benefit tracking.

## Usage

```
/credit-card-benefits:sync [options]
```

### Options

- `--full` - Full 12-month resync (useful if data seems off)
- `--since YYYY-MM-DD` - Sync from specific date
- (no options) - Incremental sync from last sync date

## How It Works

### Incremental Sync (Default)

Fetches transactions since `config.sync.lastSyncDate`:

```
Last sync: 2026-01-15

Fetching transactions from 2026-01-15 to today...

AMEX PLATINUM (via YNAB)
────────────────────────
New transactions: 12
Benefit matches: 4
Credits received: 3

CHASE SAPPHIRE RESERVE (via YNAB)
─────────────────────────────────
New transactions: 8
Benefit matches: 2
Credits received: 2

Updating checklist...
Done. Next sync will start from 2026-01-22.
```

### Full Sync

Re-fetches 12 months of data:

```
/credit-card-benefits:sync --full

WARNING: This will re-analyze 12 months of transactions.
Existing benefit tracking will be recalculated.

Proceed? [y/n]
```

## Data Source Handling

### YNAB MCP Server

If MCP server is configured and available:

```python
# Pseudo-code for MCP approach
for card in enabled_cards:
    account_id = config.ynab.accountMapping[card]
    transactions = mcp__ynab__get_transactions(
        budget_id=config.ynab.budgetId,
        account_id=account_id,
        since_date=last_sync_date
    )
    process_transactions(card, transactions)
```

### YNAB API

If using direct API:

```bash
TOKEN=$(cat ~/.config/credit-card-benefits/ynab-token)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://api.youneedabudget.com/v1/budgets/${BUDGET_ID}/accounts/${ACCOUNT_ID}/transactions?since_date=${SINCE_DATE}"
```

### CSV Import

For CSV-based tracking, sync prompts for file import:

```
CSV-based sync requires importing new statement files.

Please download recent statements:
- Amex Platinum: amex.com → Statements & Activity
- Chase Sapphire: chase.com → Activity → Download

Then run:
/credit-card-benefits:import <file.csv> --card <card-name>
```

## Transaction Processing

### 1. Detect Annual Fees

```
Looking for annual fee transactions...

Found: Jan 15 - ANNUAL MEMBERSHIP FEE $895.00 (Amex Platinum)
→ Updated lastAnnualFeeDate: 2026-01-15
→ Calculated nextAnnualFeeDate: 2027-01-15
→ Added to annualFeeHistory
```

### 2. Match Benefits

```
Matching transactions to benefits...

Amex Platinum:
  Jan 16: UBER EATS $18.50 → Uber Cash (January)
  Jan 18: LULULEMON $92.00 → Lululemon Q1 (over limit by $17)
  Jan 20: DISNEY PLUS $15.99 → Entertainment (January)

Chase Sapphire Reserve:
  Jan 17: LYFT $24.00 → Travel Credit
  Jan 19: AIRPORT PARKING $35.00 → Travel Credit
```

### 3. Identify Statement Credits

```
Finding statement credits...

Amex Platinum:
  Jan 17: UBER CASH CREDIT -$15.00 ✓
  Jan 19: LULULEMON CREDIT -$75.00 ✓
  Jan 21: ENTERTAINMENT CREDIT -$15.99 ✓

Chase Sapphire Reserve:
  Jan 18: TRAVEL CREDIT -$24.00 ✓
  Jan 20: TRAVEL CREDIT -$35.00 ✓
```

### 4. Update Checklist

```yaml
# Updates to checklist.yaml

amex-platinum:
  lastAnnualFeeDate: 2026-01-15
  benefits:
    uber-cash:
      monthsUsed: [1]  # January now tracked
    lululemon-q1:
      used: 75         # Credit amount, not spend amount
      transactions:
        - date: 2026-01-18
          amount: 92.00
          merchant: LULULEMON
          creditReceived: 75.00
          creditDate: 2026-01-19

config:
  sync:
    lastSyncDate: 2026-01-22
```

## Sync Summary

After sync completes, show summary:

```
Sync Complete
=============
Period: 2026-01-15 to 2026-01-22

Cards synced: 3
Transactions processed: 45
Benefit matches: 12
Credits received: 9

Benefits Updated:
─────────────────
Amex Platinum:
  • Uber Cash (Jan): ✓ Used ($15)
  • Lululemon Q1: $75 of $75 used
  • Entertainment (Jan): ✓ Used ($15.99)

Chase Sapphire Reserve:
  • Travel Credit: $59 of $300 used

Upcoming Expirations:
─────────────────────
  • Resy Q1 (Amex): $100 remaining - expires Mar 31
  • Saks H1 (Amex): $50 remaining - expires Jun 30

Next sync will fetch from: 2026-01-22
```

## Error Handling

### YNAB API Errors

```
Error: YNAB API returned 401 Unauthorized

Your YNAB token may have expired. To fix:
1. Go to https://app.ynab.com/settings/developer
2. Generate a new token
3. Update: ~/.config/credit-card-benefits/ynab-token
```

### Missing Account Mapping

```
Warning: No YNAB account mapped for 'delta-reserve'

To map this card:
1. Run /credit-card-benefits:configure
2. Or manually add to checklist.yaml:
   ynab.accountMapping.delta-reserve: "your-account-id"
```

### Sync Conflicts

```
Warning: Found transactions older than last sync date.

This can happen if:
- Transactions posted with earlier dates
- Manual edits to lastSyncDate

Recommendation: Run /credit-card-benefits:sync --full
```
