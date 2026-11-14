# Exports from Xero

`briefcase_start_ledger_import` returns these reports with their exact dates and steps; give the user those. This page explains them.

The start date is the first day in Briefcase. "The day before" below means the day before the start date.

| Report | Required | Run it | Brings in |
|---|---|---|---|
| Trial Balance | Yes | As at the day before. Reporting → All reports → Trial Balance, export to Excel | The opening balances |
| Journal Report | No | From the start of the previous financial year to the day before. Reporting → All reports → Journal Report, tick Description under Columns, export to Excel | The history: every journal, so the earlier transactions and comparatives are in Briefcase |
| Trial Balance at the last year end | With the history, when the trial balance has no comparative column | As at the previous year end, exported to Excel | Last year's closing balances, so the history can be checked against them |
| Aged Receivables Detail | No | As at the day before, ageing by due date, export to Excel | Unpaid sales invoices and credit notes |
| Aged Payables Detail | No | As at the day before, ageing by due date, export to Excel | Unpaid bills and supplier credits |
| Customers | No | Contacts → Customers → Export, as CSV | Customer details |

Notes:

- Export from Xero's own reports, not a custom layout or a copy pasted into a spreadsheet. Briefcase recognises the report's title, its date line and Xero's column headings.
- A large company can export the Journal Report a year at a time; add each file.
- The aged reports must be run at the same date as the trial balance, or debtors and creditors won't agree with it.
- Use the Detail aged reports. Summary versions don't list the invoices.
