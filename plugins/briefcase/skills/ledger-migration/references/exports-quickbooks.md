# Exports from QuickBooks Online

`briefcase_start_ledger_import` returns these reports with their exact dates and steps; give the user those. This page explains them.

The start date is the first day in Briefcase. "The day before" below means the day before the start date.

| Report | Required | Run it | Brings in |
|---|---|---|---|
| Trial Balance | Yes | As of the day before, on the accrual basis. Reports → Trial Balance, export to Excel | The opening balances |
| Journal | No | From the start of the previous financial year to the day before. Reports → Journal, export to Excel or CSV | The history: every journal, so the earlier transactions and comparatives are in Briefcase |
| Trial Balance at the last year end | With the history, when the trial balance has no comparative column | From the start of the previous financial year to its year end, on the accrual basis, exported to Excel | Last year's closing balances, so the history can be checked against them |
| A/R Ageing Detail | No | With the report date the day before, exported to Excel | Unpaid sales invoices and credit notes |
| A/P Ageing Detail | No | With the report date the day before, exported to Excel | Unpaid bills and supplier credits |
| Customer Contact List | No | Reports → Customer Contact List → Export → Export to Excel | Customer details |

Notes:

- Run the trial balance on the accrual basis. A cash basis trial balance blocks the import until it is run again.
- QuickBooks' trial balance carries no account types, so Briefcase matches each account from its name and balance. Expect more accounts to confirm than from Xero, and check each one: an account named after the company plus the last digits of an account number is usually a bank account, not a customer.
- Expenses coded straight to Accounts Payable appear in the A/P Ageing Detail as negative items and are imported as supplier credits. Mention it when the user codes expenses that way.
- A large company can export the Journal a year at a time; add each file.
- The ageing reports must be run at the same date as the trial balance, or debtors and creditors won't agree with it.
