# Keywords and Targeting

This document covers how the platform manages keywords, product targeting, search terms, placements, and negative targeting across Sponsored Products, Sponsored Brands, and Sponsored Display campaigns.

---

## What You Will Learn

- How keywords are managed (add, edit, pause, archive)
- How product targeting works for Sponsored Products
- How search terms are captured and managed
- How placement modifiers work (Top of Search, Product Pages, Rest of Search)
- How negative keywords prevent wasted spend
- How bulk operations work for targeting

---

## Keywords (Sponsored Products and Sponsored Brands)

Keywords determine which search queries trigger your ads.

### Keyword Levels

Campaign hierarchy: Campaign > Ad Group > Keywords

### Keyword Match Types

| Match Type | Description | Example |
| ---------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| Exact | Ad shows only for the exact search term | Keyword "coffee mug" matches search "coffee mug" |
| Phrase | Ad shows for the phrase and close variations | Keyword "coffee mug" matches "blue coffee mug" |
| Broad | Ad shows for related searches | Keyword "coffee mug" matches "travel mug" or "coffee cup" |

### Keyword Management

The keyword list view shows:
- Keyword text
- Match type (badge)
- Status (enabled, paused, archived)
- Bid amount
- Performance metrics (impressions, clicks, spend, sales, ACOS)

Users can:
- Add new keywords individually or in bulk (paste a list)
- Edit bids for single or multiple keywords
- Change match type
- Pause or enable keywords
- Archive keywords
- Copy keywords from one ad group to another

### Bulk Keyword Operations

Adding keywords in bulk is handled through a text input where users paste one keyword per line with optional match type and bid. The frontend parses the text, validates each row, and shows a preview before submitting. Duplicate keywords within the same ad group are flagged.

---

## Product Targeting (Sponsored Products)

Product targeting lets ads show on competitor product detail pages and category pages.

### Targeting Options

**Individual ASINs** — Target specific competitor products

**Category targets** — Target entire categories:
- Similar products to yours
- Related products in the same category
- Complementary products shoppers buy together

### Product Targeting Management

The product targeting view shows:
- Target type (ASIN or category)
- Target name/identifier
- Bid (per-target or inherited from ad group)
- Status
- Performance metrics

Users can add product targets by:
1. Searching for ASINs by ASIN number or product name
2. Browsing category trees
3. Pasting a list of ASINs for bulk addition

---

## Search Terms

Search terms are the actual customer queries that triggered your ads. Unlike keywords (which you choose), search terms are what customers actually typed.

### Why Search Terms Matter

Search terms reveal how customers are finding your products. They often include queries you never thought to add as keywords. The search term analysis workflow:
1. View search terms report for a campaign or ad group
2. Identify high-performing terms with good ACOS
3. Add them as exact match keywords (keyword harvesting)
4. Identify low-performing terms with high spend but no sales
5. Add them as negative keywords

### Search Term Table

The search terms table shows:
- Customer search query
- Match type that triggered the ad
- Impressions, clicks, spend
- Sales and attributed units
- ACOS
- Actions (add as keyword, add as negative)

### Search Term Categorization

Search terms can be categorized by performance:

| Category | Criteria | Suggested Action |
| ---------- | --------------------------------------------- | ------------------------------ |
| Winning | Good ACOS, consistent sales | Add as exact match keyword |
| Promising | Low impression but good conversion | Monitor and increase bids |
| Wasting | High spend, no sales, no clicks | Add as negative keyword |
| Borderline | High impressions but low conversion | Review relevance |
| New | Recently appeared, low data | Wait for more data |

---

## Placements (Sponsored Products)

Placement modifiers let you adjust bids based on where your ad appears.

### Placement Types

| Placement | Typical CPC Impact | Description |
| -------------------------- | ------------------ | ----------------------------------- |
| Top of Search (first page) | Premium | Highest visibility, most expensive |
| Product Pages | Standard | On product detail pages |
| Rest of Search | Discounted | Remaining search result positions |

### Placement Management

Users can set bid adjustment percentages for each placement. The placement list shows current adjustment percentages and allows inline editing.

---

## Negative Keywords

Negative keywords prevent ads from showing for specific search queries.

### Negative Keyword Types

**Negative Exact** — Prevents the ad from showing only for the exact search term
**Negative Phrase** — Prevents the ad from showing for the phrase and close variations

### Negative Keyword Sources

Negative keywords come from two sources:
1. **Manual addition** — User identifies a search term and adds it as a negative
2. **Automated harvesting** — The strategy engine identifies underperforming search terms and adds them as negatives

---

## Interview Talking Points

**On the complexity of search term analysis:** "Search terms are where the real PPC insights live. Keywords are what you think customers will search for, but search terms are what they actually search for. The gap between them is where optimization opportunities exist. Making it easy to analyze search terms by performance category and take bulk actions was a key UX decision."

**On match types and bidding strategy:** "Each match type serves a different purpose. Exact match gives control but limits reach. Broad match casts a wide net but can waste spend. The strategy engine handles this by automatically harvesting winning broad match search terms into exact match keywords."

**On negative keyword management:** "Negative keywords are as important as positive ones. Every dollar spent on an irrelevant click is wasted. Making negative keyword management a first-class feature with dedicated tables and automated harvesting was essential."

---

## Related Documents

- [PPC Module Overview](overview.md) - Module-level architecture
- [Campaign Management](campaign-management.md) - Campaign CRUD
- [Strategy Automation](strategy-automation.md) - Automated keyword harvesting
- [Bulk Operations](bulk-operations.md) - Bulk keyword and targeting edits
- [Performance Analytics](performance-analytics.md) - Keyword performance metrics
