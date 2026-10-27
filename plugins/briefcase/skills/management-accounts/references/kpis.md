# KPIs

These are Briefcase's own KPI formulas, the ones its KPI tiles and custom reports use, so a figure in the pack matches the one the client sees in Briefcase. Choose 5 to 8 that suit the business and audience. Show each for the period and its comparison, with its formula and inputs. If an input is missing, leave the KPI out and say why rather than approximating.

## Inputs

- **Account types and roles** come from `briefcase_get_reference_data` with `kind: accounts` (`type` and `account_role`), matched to report lines by account `id`. Roles are set per account in Briefcase: `PAYROLL` (wages and salaries, in cost of sales or overheads), `INTEREST`, `TAX` (corporation tax), `STOCK` and `BORROWINGS` (loans and finance). Use them as they are; an account that looks as if it should have a role is a question for the accountant, not an input.
- **Current or not:** an asset is current unless its type is `FIXED_ASSET`, `ACCUMULATED_DEPRECIATION` or `NON_CURRENT_ASSET`. A liability is current unless its type is `NON_CURRENT_LIABILITY`.
- **Overheads** are `total_expenses` − `total_cost_of_sales`. **Depreciation** is the expense accounts of type `DEPRECIATION_EXPENSE`.
- **Balances** come from the balance sheet at the period end, as signed there (assets positive when debit, liabilities positive when credit). Cash at bank is the sum of the `BANK_ACCOUNT` balances, so an overdrawn account counts as negative. A `BORROWINGS` account with a debit balance reduces borrowings.
- **Trade debtors and creditors** are `totals.total_without_vat` from `briefcase_get_aged_report` at the period end, for `ACCOUNTS_RECEIVABLE` and `ACCOUNTS_PAYABLE`. They leave out VAT, so days compare like with like.
- **Trailing windows:** the 12 months ending at the period end, and the 3 months for months of cover, in whole calendar months when the period ends on a month end. When the books start inside a window, Briefcase scales by the months that have figures: check the window's first month, and if it has no revenue or expenses, find the first month that does. Then n is the number of months from it to the period end; otherwise n is 12 (or 3). Write "based on n months" beside any figure where n is shorter.

## Formulas

| KPI | Formula |
|---|---|
| Revenue | `total_revenue` |
| Gross profit, gross margin | `gross_profit`; ÷ `total_revenue`. Leave out when `cost_of_sales` is empty |
| Overheads % of revenue | Overheads ÷ `total_revenue` |
| Payroll % of revenue | `PAYROLL` accounts (cost of sales and overheads) ÷ `total_revenue` |
| Prime cost | (`total_cost_of_sales` + `PAYROLL` accounts not typed `COST_OF_SALES`) ÷ `total_revenue` |
| EBITDA, EBITDA margin | `net_profit` + `INTEREST` accounts + `TAX` accounts + depreciation; ÷ `total_revenue` |
| Net profit, net margin | `net_profit`; ÷ `total_revenue` |
| Cash at bank | Sum of `BANK_ACCOUNT` balances |
| Months of cover | Cash at bank × n ÷ (overheads − depreciation) over the trailing 3 months |
| Net current assets, current ratio | Current assets − current liabilities; current assets ÷ current liabilities |
| Debtor days | Trade debtors × 365 × n ÷ (trailing 12 months' `total_revenue` × 12) |
| Creditor days | Trade creditors × 365 × n ÷ (trailing 12 months' `total_expenses` less `PAYROLL` accounts and depreciation, × 12) |
| Stock days | `STOCK` account balances × 365 × n ÷ (trailing 12 months' `total_cost_of_sales` × 12) |
| Net cash | Cash at bank − `BORROWINGS` account balances |
| Net assets | `net_assets` |
| Gearing | (`BORROWINGS` account balances + overdrawn bank accounts as positive amounts) ÷ `net_assets`, as a percentage |

With n = 12 the days formulas are the familiar balance ÷ the year's flow × 365. `total_expenses` includes cost of sales, so creditor days measure suppliers across both.

- **No denominator:** when a ratio's denominator is zero or negative, show "–" as Briefcase does, for example gearing when net assets are nil or negative, margins with no revenue, or days with no flow.
- **Comparison figures:** work each KPI out at the comparison period's end the same way: balances from the balance sheet's comparison, aged reports at the comparison end date, and windows ending there.
- **Missing roles:** with no `PAYROLL`, `STOCK` or `BORROWINGS` accounts, the KPIs that need them count nothing. Show them only when the role is in use, and say which accounts each one counted.

## Other figures

Not Briefcase KPIs, so label them as the pack's own when you show them:

| Figure | Formula | When | Pitfalls |
|---|---|---|---|
| Quick ratio | (Current assets − `STOCK` account balances) ÷ current liabilities | Businesses holding stock | Use the stock role, never account names |
| Net burn and runway | Average monthly fall in cash at bank over the last three months; cash at bank ÷ net burn | Loss-making or investor-backed | Needs three month-end balance sheets; skip if they are not available |
| Director's loan account | Balance of the directors' loan account | Companies | A debit balance means the director owes the company; flag it for the accountant rather than giving tax advice |
| Lender covenants | The facility's own definitions | Lender packs | Ask for the definitions; never substitute the KPIs above |

## Sector KPIs

Use when the business clearly fits and the inputs exist in the ledger; otherwise mention that the sector KPIs need non-ledger data:

- Hospitality: food and drink cost %, wage cost %.
- Construction: contract margin, retentions held.
- Professional services: lock-up days ((work in progress + debtors) ÷ fees × 365).
- Retail and e-commerce: gross margin by channel, stock days.
- Subscription businesses: recurring revenue and churn usually need data from outside the ledger.

## RAG ratings

Only rate a KPI red, amber or green against a target the user gave you. Use: green at or better than target, amber within 10% of it, red beyond that. Without targets, show the direction of travel instead.
