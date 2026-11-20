---
name: financial-close-review
description: Review and prepare a Briefcase client's period close. Start a working paper, check what is left to clear and which balance sheet accounts are reconciled and signed off, set up the schedule lines that support each balance on a first close and update or remove them at each month end, then review and maintain prepayments, deferred income, accruals and fixed asset depreciation with the close trackers, drafting schedules and adjustments and publishing them with a preview first. Use when the user asks about month-end, year-end, starting a close, setting up or updating schedules or working papers, whether a period is ready to close, reconciliation or sign-off status, prepayment or accrual schedules, depreciation, or what is left to post for a period.
---

# Financial close review in Briefcase

Prepayment, deferred income, accrual and fixed asset schedules need the Financial close permission in Briefcase. Working papers exist for Briefcase Ledger limited companies and Xero clients, not QuickBooks clients. Reading one needs Ledger view for a Ledger client and Financial close view for a Xero client; starting one needs Working papers create; changing its schedule lines needs Working papers update. If a call returns `FORBIDDEN`, tell the user which permission is missing rather than trying another tool.

## Start with the working paper

For "is this period ready to close" or "what is left for month-end", read the working paper before the schedules.

1. `briefcase_get_working_paper` for the client. It lists recent working papers newest first and details the latest; pass a `working_paper_id` from that list for another period. If there is none, or none covers the period the user means, offer to start one (below). A QuickBooks client returns `UNSUPPORTED_CAPABILITY`.
2. Lead with the period (`start_date` to `target_date`), whether it is open or closed, and who closed it. For a Xero client, say when the balances were read from Xero (`balance_sheet.balances_fetched_at`); if the user has posted in Xero since, ask them to press Refresh on the working paper in Briefcase and read it again.
3. Then what is left to clear. For a Xero client `clearing_the_period` and `open_client_requests` are null, so start at `pending`:
   - `clearing_the_period.blockers`: each has a `kind`, the bank account and a count. `STATEMENT_COVERAGE_GAP` means statements are missing for part of the period, so request or upload them (document submission skill). `UNRECONCILED_TRANSACTIONS` means bank lines still need matching in Briefcase. `PENDING_BANK_STATEMENTS` means uploaded statements are still being processed. `OPENING_BALANCE_NOT_SET` and `FEED_UNHEALTHY` need fixing in the Briefcase app (set the opening balance, reconnect the feed).
   - `open_client_requests`: paperwork and statements still awaited from the client.
   - `pending.unpublished_invoice_count`: bills and receipts dated in the period but not yet published (transaction review skill). `pending.draft_adjustment_count`: draft schedules and accruals (publish them below).
4. Then the balance sheet: give `balance_sheet.summary`, then name every unreconciled account and every account with `changed_since_sign_off: true`. The latter was signed off at `ledger_balance_at_sign_off` and now stands at `closing_balance`, so it needs reconciling again. For signed-off accounts, say who signed off (`reconciled_by`) and when, and quote any `difference` with its `difference_reason`. Dormant accounts are counted in `dormant_accounts_left_out`, not listed.
5. Each listed account carries its `schedule`: the hand-entered lines that support its balance, with this period's `amount`, the `previous_amount`, the `note` and whether the amount was carried forward (`origin: ROLLED_FORWARD`). A large working paper leaves lines or accounts out and says so in `warnings`; pass `account_id` to read one account in full.

The assistant cannot sign accounts off or close a working paper. Those stay with the accountant in Briefcase, so point the user to Working papers in the app for them. It can start a working paper and set up and update the schedule lines that support each balance, below.

For every change below: preview, show the preview, and end your turn there. Execute only when the user's next message agrees to that preview. An answer to a question you asked, such as a start date, a close date or an amount, is not agreement: preview with it and ask again.

## Start a working paper

For a month-end that has no working paper yet, or when the user asks to start one.

1. Agree the close date (`target_date`), usually a month end. The client's first working paper also needs a `start_date`, the day the period starts; ask the user for it. Never pick it yourself.
2. `briefcase_create_working_paper` with `mode: preview`. Show the period, whether it is the first close and, on a first close, `earliest_start_date`, the earliest start Briefcase allows. Ask whether to create it, and stop.
3. A blocker stops it: a working paper still open (`OPEN_WORKING_PAPER_EXISTS`, close it in Briefcase first), a close date not after the last closed one, a start date that is missing, in the future, not before the close date or too early, or a Ledger client that isn't a limited company. Explain it and stop.
4. After the yes, execute with the `preview_id` and a fresh `idempotency_key`, and give the working paper's `web_url`. Then read it with `briefcase_get_working_paper`.

## Set up and update schedule lines

A balance sheet account is supported by schedule lines whose amounts for the period add up to its balance; the accountant then signs it off in Briefcase. `briefcase_update_working_paper_schedules` adds lines, sets this period's amounts and removes lines across up to 100 accounts and 500 lines in one call. Nothing posts to the books or to Xero.

Which accounts it can change: unreconciled accounts of an open working paper with a balance. Leave out accounts payable and receivable (`reserved_type` `ACCOUNTS_PAYABLE` or `ACCOUNTS_RECEIVABLE`; other reserved accounts, such as VAT and retained earnings, take lines), accounts linked to a prepayment, deferred income, accrual or fixed asset schedule (`linked_schedule`), and accounts already signed off; the preview blocks them anyway.

**First close: set up the lines.**

1. Read the working paper and list the unreconciled accounts it can change, with their `closing_balance`.
2. Ask the user what supports each balance: a bank or loan statement, the share register, a payroll summary, a VAT return. Build each line from what they give you: a name for the evidence and its amount, with a note when they give one. Never invent evidence. If the user asks for each balance as one line, name the line after the evidence they name, or ask.
3. Amounts are in the balance sign convention of the read: assets positive when debit, liabilities and equity positive when credit.

**Month end: update amounts and remove cleared lines.**

1. Read the open working paper. For each account, compare each line's `amount` this period with its `previous_amount`. A carried-forward amount (`origin: ROLLED_FORWARD`) is last period's figure copied over.
2. Set the new amounts the user gives you (`line_item_id` and `amount`). Leaving `note` out keeps the existing note. A different amount or note on a carried-forward amount replaces it and removes its copied note and attachments; say so.
3. Remove lines that have cleared, such as a loan paid off (`line_item_id` and `remove: true`). A line created in this working paper is deleted with its amounts and attachments, which cannot be undone; a line carried from an earlier period is retired from this period on, and Briefcase can restore it while the working paper is open.

**Either way:**

1. `briefcase_update_working_paper_schedules` with `mode: preview`. Show each account with its `balance`, `schedule_after` and `difference_after`, and what each line does (`ADDS`, `SETS`, `REPLACES_CARRIED_FORWARD`, `KEEPS_CARRIED_FORWARD`, `DELETES`, `RETIRES`). Pass on every warning: an account still different from its balance (`STILL_DIFFERENT`), lines still without an amount, which sign-off will refuse (`LINES_WITHOUT_AMOUNT`), a line with the same name already there (`SAME_NAME_LINE_EXISTS`) and deletes that cannot be undone.
2. Blockers are listed per account and stop the whole call; fix or leave out those accounts and preview again.
3. After the yes, execute with the `preview_id` and a fresh `idempotency_key`. For more than 100 accounts or 500 lines, work in batches, each previewed and executed with its own key. After a failed execute, preview again and use a new key, because the same key replays the failure.
4. Read the working paper again and say which accounts now match their balance and are ready for the accountant to sign off in Briefcase.

## Read schedules before you write

1. `briefcase_get_close_tracker` for the client and period. Explain the grid: opening balance, movement in the period, closing balance, and which periods are posted, due or paused.
2. `briefcase_list_close_items` for one `module` at a time (`prepayment`, `deferred_income`, `accrual`, `fixed_asset`). The list gives each schedule's period summary; `briefcase_get_close_item` gives its periods (set `period_start_date` and `period_end_date` for a long schedule) and the detail the user asks about, including the source transaction where there is one.
3. Cross-check with `briefcase_list_transactions` when the user suspects a prepayment was published as a straight expense.

Report amounts in the client's currency exactly as returned. Do not recompute schedules yourself; the tracker is authoritative.

## Draft changes

- `briefcase_create_close_item` creates drafts only. Accruals are created as a draft adjustment and are not posted. Call it with `mode: preview` and a fresh `idempotency_key` first: `item` is the draft exactly as it would be saved, with its periods and accounts. Only after the user agrees, call again with the same arguments, `mode: execute`, the same `idempotency_key` and the `preview_id`.
- `briefcase_update_close_item` edits a draft or the future periods of a schedule. For an accrual it edits the adjustment amount, date and lines, not the recurring parent's estimate. Call it with `mode: preview` first: `changes` lists each field before and after, including every period that moves, and `resulting_item` is the item as it would be saved. Only after the user agrees, call again with the same arguments, `mode: execute` and the `preview_id`. It takes no `idempotency_key`.
- A preview with `blockers` can't be executed; explain each one in plain language. A `preview_id` lasts 10 minutes; `PREVIEW_EXPIRED` means preview again.
- Use `briefcase_get_reference_data` for balance sheet and expense accounts, and quote the account the schedule will post to.

## Publish, archive, restore

- Preview first: `mode: preview` on `briefcase_publish_close_item`, `briefcase_archive_close_item` or `briefcase_unarchive_close_item`. Summarise the journals that would post or reverse, the periods affected and any lock-date conflicts.
- Execute only after the user agrees, with the `preview_id`, `mode: execute` and a fresh `idempotency_key`.
- Publishing a schedule starts posting in Briefcase and the connected ledger; archiving reverses what was posted and stops future periods. Say this plainly before executing.
- Published fixed assets and committed accrual adjustments cannot be edited here; direct the user to the Briefcase app.

## Wrap up

Re-read the tracker after publishing so the user sees the updated position, and remind them the changes appear in the assistant activity under Settings → AI assistants. If the user is working towards closing the period, re-read the working paper so they see what is still outstanding.
