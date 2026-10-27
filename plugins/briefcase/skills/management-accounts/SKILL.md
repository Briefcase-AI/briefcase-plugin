---
name: management-accounts
description: Prepare monthly or quarterly management accounts for a Briefcase Ledger client as a draft for the accountant to review. Checks the books are ready, pulls profit and loss, balance sheet, trial balance, cash and aged debtors and creditors, works out KPIs the way Briefcase does, shows performance by department, location or project when the client uses reporting categories, writes commentary on the movements that matter, proves every figure against the ledger in a verification appendix, and can save the pack in Briefcase as a custom report with a PDF. Use when the user asks for management accounts, a monthly or quarterly pack, a board pack, a month-end report, or how a client did this month or quarter.
---

# Management accounts in Briefcase

You prepare a **draft** management accounts pack that an accountant reviews before anything reaches their client. The pack is only useful if the accountant can trust it, so every figure comes from a Briefcase tool response, every check is shown, and every gap is declared rather than hidden.

Rules that apply throughout:

- **Figures come only from tool responses.** Never from memory, never estimated silently. Do arithmetic in code when the host can run code; otherwise show each calculation's inputs so the reviewer can repeat it.
- **Never post and never send.** Proposed adjustments (an accrual, a corporation tax estimate) go through the manual journals skill with a preview and the user's agreement. The pack goes to the accountant, not the client.
- **Mark the pack "Draft — for accountant review"** until the user confirms they have reviewed it.
- **Briefcase Ledger clients only.** Check `ledger_type` with `briefcase_get_client`. The report tools return `UNSUPPORTED_CAPABILITY` for Xero or QuickBooks clients; tell the user to use their ledger's own reporting and stop.
- The reports need the Ledger view permission. `FORBIDDEN` means the user lacks it; say so.

Work through the steps in order. Read the reference files when a step points to them.

## 1. Agree the brief

Settle these before pulling any figures. Use the defaults below and state them in the pack rather than stopping to ask; ask only when the client or period is unclear, or the financial year start cannot be found:

- **Client and period.** Month or quarter; resolve it to explicit `start_date` and `end_date` and say which dates you used.
- **Financial year.** `briefcase_get_trial_balance` reports the `financial_year_start` it used. If it says no financial year end is set, ask the user for the first day of the financial year.
- **Entity and accounting basis.** `briefcase_get_reference_data` with `kind: businesses` lists a sole trader's or landlord's businesses with their `type` and `accounting_method`. An empty list usually means a limited company on the accruals basis; confirm with the user when it matters. Cash basis businesses have no debtors, creditors or accruals to report, so leave those sections out and say why. Corporation tax applies to companies only.
- **Audience.** Owner (default), board, lender or investor. It changes emphasis, not the figures (see `references/pack-template.md`).
- **Comparisons.** Prior period and the same period last year by default. Budget comparison only when the user supplies a budget; never invent one.
- **Reporting categories.** `briefcase_get_reference_data` with `kind: reporting_categories`. When the client uses them, the pack shows performance by the category the user names; with several and none named, use the first listed and say which. Leave the section out when the list is empty.
- **Materiality.** A movement is material when it is at least 10% **and** at least £1,000 (or 1% of the period's revenue if larger). Tell the user the thresholds and use theirs if they give them.

## 2. Check the books are ready

Run the readiness checks in `references/verification.md` before pulling report figures:

- the working paper for the period end (what is left to clear, unreconciled accounts, accounts changed since sign-off);
- documents still awaiting review dated in or before the period;
- close schedules due but not posted;
- clearing and suspense accounts that are not nil;
- the lock date.

Do not stop here to ask whether to continue: the user asked for the pack. Carry on, put every failed check in the pack's caveats, and offer at the end to help fix them and rebuild. Never refuse to prepare the pack because the books are incomplete; make the incompleteness visible.

## 3. Pull the figures

Make these calls for the period end and keep each response; the verification appendix cites them.

1. `briefcase_get_profit_and_loss` for the period, with the prior period as the comparison.
2. `briefcase_get_profit_and_loss` for the period, with the same period last year as the comparison.
3. `briefcase_get_profit_and_loss` for the financial year to date, with the prior year to date as the comparison.
4. `briefcase_get_balance_sheet` at the period end, with the prior period end as `comparison_as_at`.
5. `briefcase_get_trial_balance` at the period end.
6. `briefcase_get_aged_report` for `ACCOUNTS_RECEIVABLE` and for `ACCOUNTS_PAYABLE` (accrual basis only), at the period end and at the comparison period end. Their `totals.total_without_vat` feed debtor and creditor days.
7. `briefcase_get_reference_data` with `kind: accounts` for account `class`, `type`, `is_bank_account`, `reserved_type` and `account_role`. It returns up to 150 accounts with the full `total`; when there are more, call it once per `account_class`.
8. `briefcase_get_profit_and_loss` for the trailing 12 months and the trailing 3 months ending at the period end, each with the same window ending at the comparison period end as the comparison (see `references/kpis.md`).
9. When the client uses reporting categories: `briefcase_get_profit_and_loss` for the period with `columns_by_reporting_category_id`, and again for the comparison period.

Quote amounts as returned, in the response's `currency`, following each response's `sign_convention`. Accounts with no movement or a zero balance are left out of the reports, so a missing account is nil.

**Cost of sales.** The ledger types direct costs as cost of sales, and each profit and loss response lists those accounts in `cost_of_sales` with `total_cost_of_sales` and `gross_profit` (revenue less cost of sales). They are also listed in `expenses` and counted in `total_expenses`, so overheads are `total_expenses` − `total_cost_of_sales`; never add the two lists together. When `cost_of_sales` is empty, report revenue, overheads and net profit without a gross profit line. Do not move accounts between cost of sales and overheads yourself: if an expense account's name suggests a direct cost (Purchases, Materials, Subcontractors) or a cost-of-sales account looks like an overhead, raise it as a question for the accountant, who can switch the account's type in the chart of accounts.

## 4. Work out the KPIs

`references/kpis.md` has Briefcase's own KPI formulas, the ones its KPI tiles and custom reports use, so the pack agrees with what the client sees in Briefcase. Pick 5 to 8 that suit the business and audience, compute each for the period and its comparison from the responses, and show its formula and inputs.

The KPIs read account types and roles, never account names. If an account looks as if it should carry a role it lacks, such as a loan without the loan role or wages outside payroll, do not count it yourself: raise it as a question for the accountant, who can set it in Briefcase under Chart of accounts → Account roles.

## 5. Prove the figures

Run every tie-out in `references/verification.md`: balance sheet balances, P&L totals, year-to-date profit against retained earnings, trial balance totals, aged reports against control accounts, bank balances against the feed, KPI recalculation. Record each with the figures compared and pass or fail. A failure goes at the top of the pack with what you found; do not explain it away.

Then the analytical review in the same file: material movements, gross margin swings, regular costs missing this period (usually a missed accrual), balances on the wrong side, new accounts and large manual journals.

## 6. Find the reasons behind the movements

For each material movement (3 to 5 of them, largest first):

1. `briefcase_list_journals` with the account's `id` and the period's dates shows what posted.
2. `briefcase_get_journal` and its `source_links` lead to the documents (`briefcase_get_transaction`, `briefcase_get_sales_invoice`).
3. Write the cause only if the journals show it, and cite the journal or document. If they do not, turn it into a question for the client: "Rent is £2,000 above last month. Has the new lease started?"

## 7. Write the pack

Follow `references/pack-template.md`: a one-page summary first (what happened, why, and 3 to 5 suggested actions), then KPIs, profit and loss (with performance by reporting category when the client uses them), balance sheet, cash, aged debtors and creditors, commentary, and the verification appendix with the basis of preparation.

Commentary says what happened, why, and what it means. Do not narrate numbers the tables already show, and do not make forward-looking claims the user has not given you. Report bad news as plainly as good news.

## 8. Check the draft before handing it over

Run the final checks in `references/verification.md`: fetch the period profit and loss, the balance sheet and the trial balance again with the same arguments as the first pull and compare their totals, and check every number and percentage in the text against the tables. If the books changed while you were working, say so and rebuild the affected sections.

## 9. Hand over

Give the user the pack marked "Draft — for accountant review", list the caveats and open questions for the client, and point them to the reviewer checklist at the end of the appendix. Offer to produce it as a document if the host can create files. Do not send it anywhere.

Then offer to save it in Briefcase as a custom report, so the accountant can open the live figures and a PDF there. If the user agrees:

1. `briefcase_create_custom_report` with `template: MONTHLY_MANAGEMENT_PACK`, a name such as "Management accounts September 2026", the pack's period as `start_date` and `end_date` so it keeps showing this period, the pack's main comparison as `comparison`, and the one-page summary as `summary`, headed "Draft — for accountant review" (paragraphs, bullets and bold only). Preview it and show the widgets; execute only once the user agrees, with the same `idempotency_key`.
2. Read it back with `briefcase_get_saved_report` and compare the KPIs it shares with the pack. Both use the same formulas, so a difference means the books changed or an input differs: find out which before handing over.
3. Give the user the report's `web_url`, and a PDF link from `briefcase_get_report_pdf` with its `report_id`, saying the link expires in 15 minutes.

If a blocker stops it, such as no permission to create in the Ledger or assistant changes turned off in Briefcase, say so: the pack stands on its own. The pack itself, with its verification appendix, stays as above.
