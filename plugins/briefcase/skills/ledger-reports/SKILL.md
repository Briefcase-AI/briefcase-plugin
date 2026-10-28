---
name: ledger-reports
description: Read and explain a Briefcase Ledger client's profit and loss (also by department, location, project or any other reporting category), balance sheet, trial balance, aged creditors and aged debtors, compare periods, and drill from a report line into the journals and source documents behind it. Also reads saved and custom reports, creates and changes custom reports such as a management pack in Briefcase, and gives PDFs of any report. Use when the user asks how a client is doing, how a department, site or project performed, what makes up a balance, why a figure moved, who owes or is owed money, for a trial balance or nominal balances, for a saved or custom report, to build or change a report or dashboard in Briefcase, or for a report PDF.
---

# Ledger reports in Briefcase

These tools only work for Briefcase Ledger clients. Check `ledger_type` from `briefcase_list_clients` or `briefcase_get_client`; for a Xero or QuickBooks client they return `UNSUPPORTED_CAPABILITY`, so tell the user to read the reports in that ledger instead. Everything here needs the Ledger view permission; `FORBIDDEN` means the user lacks it.

## Pick the report

- **Profit and loss:** `briefcase_get_profit_and_loss` with `start_date` and `end_date`. Add `comparison_start_date` and `comparison_end_date` for "this month against last month" or "this year against last year". Filter with `business_id` or `property_id` only when the user asks about one business or property (`briefcase_get_reference_data` with `kind: businesses` or `properties`). Accounts typed cost of sales are listed again in `cost_of_sales`, with `total_cost_of_sales` and `gross_profit` (revenue less cost of sales); they stay in `expenses` too, so overheads are `total_expenses` − `total_cost_of_sales`.
- **Balance sheet:** `briefcase_get_balance_sheet` with `as_at` (defaults to today) and optionally `comparison_as_at`. Retained earnings is the profit to date shown within equity, so `total_equity` equals `net_assets`.
- **Trial balance:** `briefcase_get_trial_balance` with `as_at` (defaults to today). Every account with a balance sits in a debit or credit column. Balance sheet accounts show the balance at `as_at`; revenue and expense accounts show the financial year to date, and earlier years' profit or loss is one `retained_earnings_brought_forward` line. `totals.difference` is `0.00` when the ledger balances. The financial year comes from the client's year end; if the tool says none is set, ask the user for the first day of the financial year and pass it as `financial_year_start`. Use it when the user asks for a trial balance or nominal balances, or to read control accounts such as VAT, debtors, creditors and clearing accounts in one call (`reserved_type` marks them).
- **Aged creditors or debtors:** `briefcase_get_aged_report` with `subledger: ACCOUNTS_PAYABLE` (creditors) or `ACCOUNTS_RECEIVABLE` (debtors) and `as_at`. Totals cover every contact; contacts come largest balance first, 25 per page with up to 10 invoices each. Page with `after` for more contacts, and set `contact_id` to see one contact's invoices in full.

Resolve relative periods ("last quarter", "year to date") to explicit dates, and say which dates you used. Ask for the financial year end when the user says "this year" and it matters.

## Reporting categories

Briefcase Ledger clients can tag lines with reporting categories such as Department, Location or Project, each with its options (Sales, Leeds, Project Alpha).

- `briefcase_get_reference_data` with `kind: reporting_categories` lists the categories and their option ids. An empty list means the client uses none: say so rather than splitting the figures by account names.
- **Every option side by side:** add `columns_by_reporting_category_id` to `briefcase_get_profit_and_loss`. `columns_by_reporting_category.columns` has one column per option (`kind: OPTION`, the 50 with the largest net first), then `OTHER` for the remaining options and `UNASSIGNED` for lines with no option in that category. Each column carries the same totals as the period, and together they add up to it. The columns cover the period only; for a comparison by option, make a second call with the comparison dates as `start_date` and `end_date`. When a breakdown is too large the tool says so; filter it to the options needed.
- **One option or a few:** `reporting_category_filters` narrows every figure, the comparison included, to lines that match each category given: one of its `option_ids`, or with `include_unassigned`, no option in it. `briefcase_list_journals` takes the same filters, so you can drill into a filtered figure.
- **Reading the columns:** an option's contribution is its revenue less its cost of sales and overheads, which is its `net_profit`. Unassigned holds what was never tagged, usually central costs; say how large it is, because a large Unassigned column means the split is partial. An option with no revenue is a cost centre: judge it by its costs rather than a margin.
- The balance sheet, trial balance and aged reports are not split by reporting category.

## Saved and custom reports

- **List:** `briefcase_get_saved_report` without `report_id` lists the reports the user can see, newest first, with `type` (`CUSTOM` or a saved `PROFIT_AND_LOSS` view), `template`, `can_edit` and `web_url`. `search_term` filters by name.
- **Read:** pass `report_id`. A custom report returns its settings and each widget with its figures for its saved period, or for `start_date` and `end_date` when given. Quote the figures as returned. When the response lists `omitted_widget_ids`, read those with `widget_ids`. A saved profit and loss view returns the `briefcase_get_profit_and_loss` arguments that reproduce it; call that tool with them for the figures.
- **Create:** `briefcase_create_custom_report`. It saves a report in Briefcase and changes nothing in the books.
  1. Pick a `template`: `MONTHLY_MANAGEMENT_PACK`, `REPORTING_CATEGORY_PERFORMANCE` (limited companies only, with the `reporting_category_id`), `CASH_AND_WORKING_CAPITAL`, or `BLANK` with your own `widgets`. `widgets` replaces a template's widgets; `briefcase_get_reference_data` with `kind: report_widgets` describes each widget type with its settings and defaults, the KPIs and each template's widgets, so read it before choosing widgets.
  2. Set the period: a `period` preset, which moves on with time (default `LAST_MONTH`), or a fixed `start_date` and `end_date`. Add `comparison` if the user wants one other than the template's.
  3. Call with `mode: preview` and a new `idempotency_key`. Show the user the name, period and comparison and the widgets in order, and explain any blocker in plain language: no permission to create in the Ledger, assistant changes turned off in Briefcase, an unknown category, option or account, or more than 40 widgets.
  4. Only after the user agrees, call again with `mode: execute`, the same `idempotency_key` and the `preview_id`. Return the `web_url`. The report is shared with the user to edit and added to their favourites unless they asked otherwise (`should_add_to_favourites: false`).
- **Change:** `briefcase_update_custom_report` changes a custom report the user can edit (`can_edit` in the list). Saved profit and loss views can only be changed in Briefcase. It changes nothing in the books.
  1. Read the report with `briefcase_get_saved_report` and its `report_id`. If the response lists `omitted_widget_ids`, read those too with `widget_ids` before changing widgets, so you have every widget.
  2. Pass the read's `updated_at` as `expected_updated_at`, and only the arguments that change: `name`, a `period` preset or `start_date` and `end_date`, `comparison`, `summary`. To change the widgets, send the full `widgets` list in the order the report should show them: keep a widget by its `widget_id` with its `type` and `width` (settings you leave out stay as they are, and `figures` are not sent back), add one by leaving `widget_id` out, and remove one by leaving it out of the list. A widget can't change type: remove it and add a new one.
  3. Call with `mode: preview` and a new `idempotency_key`. Show the user the `changes` and explain any blocker in plain language: the report changed since you read it (read it again), no edit access, an unknown or repeated widget id, or a setting the widget doesn't have.
  4. Only after the user agrees, call again with `mode: execute`, the same `idempotency_key` and the `preview_id`. Return the `web_url`. To change it again, use the `updated_at` the result returns.
- **PDF:** `briefcase_get_report_pdf` with `report_id` for a saved or custom report, or `report` with the arguments the standard report's own tool takes (its `comparison` is `NONE`, `PREVIOUS_PERIOD` or `PREVIOUS_YEAR` rather than dates). Give the user the `download_url` straight away and say it expires in 15 minutes; fetch a new one if they need it later. You cannot read the PDF, so take any figures from the report tools.

## Report the numbers faithfully

- Amounts are decimal strings in the report's `currency`. Quote them as returned and follow the `sign_convention` in each result; do not recompute totals or re-sign figures.
- Accounts with no movement or a zero balance are left out, so an account missing from the list means it is nil for that range.
- Lead with the few lines that explain the answer (largest movements, biggest balances, oldest debts) rather than reading out every account.

## Drill down

1. Take the account `id` from a report line.
2. `briefcase_list_journals` with `account_id` and the same date range shows that account's activity; its `debit`, `credit` and `totals` cover that account only.
3. `briefcase_get_journal` for one entry shows every line and `source_links`, such as `sales_invoice_id` or `transaction_id`. Follow them with `briefcase_get_sales_invoice` or `briefcase_get_transaction`.

For an aged report line, the invoice `source` tells you which tool opens it (`SALES_INVOICE` → `briefcase_get_sales_invoice`, `TRANSACTION` → `briefcase_get_transaction`).

## Limits

These reports read Briefcase's own ledger. There is no VAT return tool; point the user to the Briefcase app for that. For bank reconciliation progress and which balance sheet accounts are signed off for a period, use `briefcase_get_working_paper` as described in the financial close review skill. For how-to questions about the reports, use `briefcase_search_help` rather than guessing.
