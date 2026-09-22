---
name: financial-close-review
description: Review and maintain prepayments, deferred income, accruals and fixed asset depreciation for a Briefcase client at period end, read the close trackers, draft new schedules and adjustments, and publish them with a preview first. Use when the user asks about month-end, year-end, prepayment or accrual schedules, depreciation, or what is left to post for a period.
---

# Financial close review in Briefcase

Everything here needs the FINANCIAL_CLOSE permission in Briefcase. If a call returns `FORBIDDEN`, tell the user which permission is missing rather than trying another tool.

## Read before you write

1. `briefcase_get_close_tracker` for the client and period. Explain the grid: opening balance, movement in the period, closing balance, and which periods are posted, due or paused.
2. `briefcase_list_close_items` for one `module` at a time (`prepayment`, `deferred_income`, `accrual`, `fixed_asset`). `briefcase_get_close_item` for the detail the user asks about, including the source transaction where there is one.
3. Cross-check with `briefcase_list_transactions` when the user suspects a prepayment was published as a straight expense.

Report amounts in the client's currency exactly as returned. Do not recompute schedules yourself; the tracker is authoritative.

## Draft changes

- `briefcase_create_close_item` creates drafts only. Accruals are created as a draft adjustment and are not posted. Always pass a fresh `idempotency_key`.
- `briefcase_update_close_item` edits a draft or the future periods of a schedule. For an accrual it edits the adjustment amount, date and lines, not the recurring parent's estimate.
- Use `briefcase_get_reference_data` for balance sheet and expense accounts, and quote the account the schedule will post to.

## Publish, archive, restore

- Preview first: `mode: preview` on `briefcase_publish_close_item`, `briefcase_archive_close_item` or `briefcase_unarchive_close_item`. Summarise the journals that would post or reverse, the periods affected and any lock-date conflicts.
- Execute only after the user agrees, with the `preview_id`, `mode: execute` and a fresh `idempotency_key`.
- Publishing a schedule starts posting in Briefcase and the connected ledger; archiving reverses what was posted and stops future periods. Say this plainly before executing.
- Published fixed assets and committed accrual adjustments cannot be edited here; direct the user to the Briefcase app.

## Wrap up

Re-read the tracker after publishing so the user sees the updated position, and remind them the changes appear under Settings → Connected assistants → Activity.
