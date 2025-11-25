# Price Checker

The price checker module lets sellers run A/B price experiments on their products. It tests different price points to find the optimal balance between sales volume and profit margin.

---

## What You Will Learn

- How price experiments are set up
- How the A/B testing logic works
- How experiment results are displayed
- How automated winner detection works
- How notifications work when experiments complete

---

## Experiment Setup

Creating a new price experiment:
1. **Select product** — Choose the ASIN to test
2. **Set control price** — The current/standard price (no change)
3. **Set test price** — The new price to test
4. **Set duration** — How long the experiment runs (default 14 days)
5. **Set success criteria** — Minimum sales data needed for a valid result
6. **Start experiment**

---

## How A/B Testing Works

The experiment tracks both price points. For each group, the system tracks:
- Units sold
- Revenue generated
- Profit margin (factoring in COGS and fees)
- Conversion rate
- Overall profitability

---

## Experiment Results

When an experiment completes, results are shown in a comparison view:

| Metric | Control Price ($19.99) | Test Price ($17.99) | Difference |
| ---------------- | ---------------------- | ------------------- | ---------- |
| Units Sold | 145 | 198 | +36.6% |
| Revenue | $2,898 | $3,562 | +22.9% |
| Profit | $870 | $1,068 | +22.8% |
| Conversion Rate | 8.2% | 11.4% | +39% |

---

## Experiment Charts

The experiment detail page shows charts:
- **Sales comparison** — Comparing sales at control vs test price over time
- **Profit comparison** — Comparing profit at both prices
- **Cumulative results** — Running total of the difference between test and control

---

## Notifications

When an experiment completes, the user gets a notification:
1. The experiment status changes to "completed"
2. A notification is sent via the notification system
3. The notification includes a link to view the full results

---

## Interview Talking Points

**On A/B testing at scale:** "Price testing is tricky because you cannot run true A/B tests on the marketplace — it does not let you show different prices to different customers simultaneously. Instead, we run sequential tests where the price changes over time and we compare before-and-after performance, accounting for seasonality and external factors."

---

## Related Documents

- [Dashboard Analytics](dashboard-analytics.md) - Dashboard overview
- [Notifications](../notifications/notification-architecture.md) - Notification system
- [Profit and Loss](profit-and-loss.md) - Profit tracking
- [Product Management](../inventory/product-management.md) - Product details
