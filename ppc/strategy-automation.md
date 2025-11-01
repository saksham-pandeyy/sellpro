# Strategy Automation

The strategy engine is one of the most technically interesting parts of the PPC module. It lets users define automated rules for bid optimization, keyword harvesting, and campaign management. The system then evaluates campaigns against these rules and makes adjustments without manual intervention.

---

## What You Will Learn

- How strategies are structured (goals, rules, linked campaigns)
- How the evaluation engine works (scan, analyze, compare, adjust, log)
- How bid optimization decisions are calculated
- How keyword harvesting automatically adds winning search terms
- How the system prevents conflicting actions
- How every automated change is logged for audit

---

## What Is a Strategy?

A strategy is a set of rules that tells the system how to manage a group of campaigns. The user defines:
1. **Performance goals** — Target ACOS, target ROAS, target spend
2. **Linked campaigns** — Which campaigns the strategy controls
3. **Action rules** — What to do when performance deviates from targets
4. **Products** — Which ASINs are being managed

Once a strategy is active, the evaluation engine runs periodically and makes automated adjustments.

---

## Strategy Components

- **Strategy List** — Shows all strategies with status, linked campaigns count, and performance summary
- **Strategy Form** — Multi-section form where users define goals, select products, and link campaigns
- **Product Selector** — Modal for choosing which ASINs the strategy manages
- **Campaign Linker** — UI for selecting which campaigns are controlled by the strategy
- **Evaluation Engine** — Background process that runs strategy rules against campaign data

---

## Strategy Creation Flow

1. User enters strategy name and goals (target ACOS, ROAS)
2. User selects products (ASINs) the strategy manages
3. System shows matching campaigns for selected products
4. User links campaigns to the strategy
5. User sets action rules (bid adjustments, thresholds)
6. Strategy is saved and evaluation begins on the next cycle

---

## How the Evaluation Engine Works

The engine runs as a scheduled background process:

1. **Load** all active strategies
2. **For each strategy:**
   a. Get linked campaigns
   b. **For each campaign:**
      - Pull performance data (impressions, clicks, spend, sales, ACOS) from the external API
      - Compare actual performance against strategy targets
      - **If ACOS is above target** (too expensive): Calculate optimal bid reduction, update campaign bids, log the change
      - **If ACOS is below target** (opportunity to scale): Calculate optimal bid increase, update bids, log the change
      - **If campaign is wasting spend**: Check if pause threshold is met, pause underperformers, log the action
      - **If high-performing search terms are found**: Filter terms above threshold, add as keywords, log the addition
3. Update strategy's last-evaluated timestamp

---

## Bid Optimization Logic

The bid optimization uses a percentage-based adjustment model:

```
Actual ACOS = Total Spend / Total Sales  
Target ACOS = User-defined goal (e.g., 25%)

If Actual ACOS > Target ACOS:
    Overshoot % = (Actual ACOS - Target ACOS) / Target ACOS
    Bid reduction % = min(Overshoot % × adjustment factor, max reduction limit)

If Actual ACOS < Target ACOS:
    Undershoot % = (Target ACOS - Actual ACOS) / Target ACOS
    Bid increase % = min(Undershoot % × adjustment factor, max increase limit)
```

### Safety Limits

The engine has hard limits to prevent extreme adjustments:
- Maximum single adjustment: Configurable (default 20%)
- Minimum bid: Configurable floor for visibility
- Maximum bid: Configurable ceiling to control costs
- Cooldown period: Minimum time between adjustments for the same campaign

---

## Keyword Harvesting

When the engine finds search terms that perform well but are not tracked as keywords, it automatically adds them as exact match keywords.

### Harvesting Criteria

A search term is considered for harvesting when:
1. It has generated at least N sales (configurable)
2. Its ACOS is below the target ACOS
3. It is not already a keyword or negative keyword
4. It has sufficient impression volume (configurable)

### Harvesting Process

1. Engine pulls search term reports for linked campaigns
2. Search terms meeting all criteria are flagged for addition
3. Engine adds each term as an exact match keyword
4. The addition is logged in the audit history

---

## Actions the Engine Can Take

| Action | Description | When It Happens |
| ------------------- | ------------------------------------------------- | ---------------------------------------------------------------- |
| Increase bid | Raise bid by calculated percentage | Campaign/target is under-spending with good ACOS |
| Decrease bid | Lower bid by calculated percentage | Campaign/target is over-spending with high ACOS |
| Pause campaign | Disable a campaign entirely | Campaign has high ACOS with no improvement trend |
| Enable campaign | Reactivate a paused campaign | Campaign conditions have improved |
| Add keyword | Add a search term as a new keyword | Search term shows strong performance |
| Add negative keyword | Add a term as a negative keyword | Search term is wasting spend with no conversions |
| Adjust placement | Modify bid multiplier for placement | Placement performance deviates from targets |

---

## Interview Talking Points

**On the rule engine:** "The strategy engine runs as a scheduled background job that evaluates campaigns against configurable rules. It loads active strategies, pulls campaign performance from the external API, compares actual ACOS against targets, and makes calculated adjustments. Every change is logged with before/after values so users can audit what happened. Safety limits prevent extreme changes based on outlier data."

**On rate limit handling:** "The external API has strict rate limits. The strategy engine batches updates where possible and adds delays between API calls. If a rate limit is hit, the engine backs off and retries on the next cycle."

**On handling conflicting strategies:** "One edge case is what happens when two strategies target the same campaign. This is handled by preventing users from linking the same campaign to multiple active strategies during strategy creation."

**On the ACOS calculation:** "A percentage-based adjustment model is used instead of fixed amounts. If actual ACOS is 30% above target, bids are reduced by a calculated percentage. This makes the system responsive to campaigns with different budget levels."

---

## Related Documents

- [PPC Module Overview](overview.md) - Module-level architecture
- [Campaign Management](campaign-management.md) - Campaign CRUD and budget management
- [Keywords and Targeting](keyword-and-targeting.md) - Keyword management
- [Audit History](audit-history.md) - How strategy changes are tracked
- [Bulk Operations](bulk-operations.md) - Manual bulk campaign edits
