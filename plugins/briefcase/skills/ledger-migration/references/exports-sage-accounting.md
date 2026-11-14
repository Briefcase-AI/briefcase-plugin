# Exports from Sage Accounting

`briefcase_start_ledger_import` returns these reports with their exact dates and steps; give the user those. This page explains them.

The start date is the first day in Briefcase. "The day before" below means the day before the start date.

| Report | Required | Run it | Brings in |
|---|---|---|---|
| Trial Balance | Yes | Reports → All reports → Trial Balance, select Try the new report, set As Of to the day before, export to Excel | The opening balances |
| Audit Trail | No | Reports → All reports → Audit Trail, Detailed, period Custom from the start of the previous financial year to the day before, under More set Status to Exclude deleted, export as CSV | The history: every transaction, so the earlier transactions and comparatives are in Briefcase |
| Aged Debtors | No | Reports → All reports → Aged Debtors, date the day before, Calculate, then Detailed, export as CSV | Unpaid sales invoices and credit notes |
| Aged Creditors | No | Reports → All reports → Aged Creditors, date the day before, Calculate, then Detailed, export as CSV | Unpaid bills and supplier credits |
| Customers | No | Contacts → Customers, the arrow next to New customer, Export all | Customer details |

Notes:

- Export the trial balance to Excel from the new report. Its CSV export has no title or date, and the old report isn't accepted, so both are refused.
- Sage 50 isn't supported here; use the import in Briefcase.
- Sage's detailed aged reports don't say whether they are debtors or creditors. Briefcase tells them apart from the trial balance's control accounts, so add the trial balance first. If an aged report is refused because its side is unknown, add it again once the trial balance is in.
- No year-end trial balance is needed: Sage's trial balance already carries last year's figures, with last year's profit closed into Prior Years.
- A large company can export the Audit Trail a year at a time; add each file.
- The aged reports must be run at the same date as the trial balance, or debtors and creditors won't agree with it.
