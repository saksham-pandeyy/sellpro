# Bulk Operations

The bulk operations system lets users make changes to multiple campaigns, ad groups, keywords, and targets at once. Without bulk operations, managing hundreds of campaigns would require repetitive manual work.

---

## What You Will Learn

- How bulk selection works across tables
- How bulk edit modals handle different action types
- How the frontend validates bulk operations before submission
- How the backend processes bulk updates efficiently
- How error handling works when some items fail and others succeed

---

## Bulk Selection

Users select multiple items using checkboxes in the table. A master checkbox at the top selects or deselects all visible rows. When items are selected, a toolbar appears showing the count and available actions. Actions are enabled or disabled based on what can be done with the selected items.

---

## Bulk Edit Modals

Each bulk action opens a modal that handles the specifics of that operation.

### Bulk Bid Edit

Users can update bids for multiple keywords or targets at once. The modal offers three modes:
1. **Set to value** — Set all selected bids to the same amount
2. **Increase by percentage** — Increase all selected bids by X%
3. **Decrease by percentage** — Decrease all selected bids by X%

The modal shows a preview of how many items will be affected and the total impact.

### Bulk Status Change

Users can enable, pause, or archive multiple campaigns, ad groups, or targets at once. The flow:
1. User selects items and clicks "Change Status"
2. Modal opens with status options
3. User selects the target status
4. Confirmation modal shows the count and action
5. User confirms to submit

### Bulk Budget Edit

For campaigns, users can update budgets in bulk with the same set/percentage modes as bid editing.

---

## Frontend Validation

Before submitting a bulk operation, the frontend validates:
1. **Selection is not empty** — At least one item must be selected
2. **Values are valid** — Bids and budgets must be positive numbers
3. **Percentage values are in range** — Adjustments cannot exceed configured limits
4. **Status transitions are allowed** — Some transitions are prohibited by the external API

---

## Backend Processing

The backend processes bulk operations as a batch. It receives an array of item IDs and the action to perform.

### Processing Logic

1. Backend validates all items exist and belong to the user
2. Backend checks external API rate limits
3. Items are processed sequentially to respect rate limits
4. Each item's result (success or failure) is recorded
5. A summary response is returned

### Response

The response includes:
- Total items processed
- Number succeeded
- Number failed
- Reasons for each failure

The frontend displays a clear summary showing what happened.

---

## Bulk Actions by Entity

| Entity | Available Bulk Actions |
| ----------------- | ----------------------------------------------------------- |
| Campaigns | Budget update, Status change, Placement adjustment |
| Ad Groups | Status change, Default bid update |
| Keywords | Bid update, Status change, Match type change |
| Product Targets | Bid update, Status change |
| Search Terms | Add as keyword, Add as negative |
| Placements | Bid adjustment change |

---

## Interview Talking Points

**On partial success handling:** "Bulk operations can have partial failures. Some items might succeed while others fail due to API errors or validation issues. We handle this by processing each item individually and collecting results. The frontend shows a clear summary of what happened."

**On rate limit awareness:** "The external API has strict rate limits. A bulk update of 200 keywords cannot send all 200 requests at once. The backend processes them in batches with delays to stay within rate limits."

**On the UX tradeoff:** "There is a tension between showing immediate feedback and actually completing the operation. We use optimistic UI updates for selected items while the backend processes changes. If failures are reported, those specific items are reverted."

---

## Related Documents

- [PPC Module Overview](overview.md) - Module-level architecture
- [Campaign Management](campaign-management.md) - Campaign CRUD
- [Strategy Automation](strategy-automation.md) - Automated vs manual operations
- [Keywords and Targeting](keyword-and-targeting.md) - Keyword management
