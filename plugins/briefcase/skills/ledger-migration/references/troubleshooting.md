# Troubleshooting

## What each need means

`briefcase_get_ledger_import` lists what blocks the import in `needs`. Each has a `kind`, a `message` to pass on, and the `source_key`, `file_id` or `side` it concerns.

| Kind | Means | Clear it with |
|---|---|---|
| `MISSING_FILE` | A required report isn't in yet: the trial balance, or the trial balance at the last year end when the history needs it | The user exports it; add it as in `files.md` |
| `FILE_PROBLEM` | A file can't be used, including a year-end trial balance that doesn't balance. The message says what was expected and how to export it | Remove it with `remove_file_ids` and add a fresh export |
| `TRIAL_BALANCE_PROBLEM` | The trial balance is at the wrong date or on the cash basis, or no account rows were found in it | Export it again as the message says. For a workbook with several sheets, pass `trial_balance_sheet` with the right sheet's name |
| `HISTORY_PROBLEM` | The history can't be read, has problems against the trial balance, or is missing last year's journals | Export it again for the period the checklist gives, add the missing year, or remove the history files |
| `ACCOUNT_CHECK` | Briefcase isn't sure about an account: no match, a possible bank account, a type that disagrees, or an unsure match | Show it from `section: accounts`; the user confirms (`confirm_account_checks`) or picks another account (`mappings`) |
| `ACCOUNTS` | An account still needs a Briefcase account, a foreign currency bank account needs its balance in its own currency, or a bank account found only in the history needs its closing date | `mappings`; `bank_accounts` with `currency_statement_balance`; `bank_accounts` with `closed_on` |
| `AGED_AS_AT` | An aged report doesn't say the date it was run as at | Ask the user which date they ran it for. Pass `confirm_aged_as_at` with its side only when it was the day before the start date; otherwise export it again |
| `AGED_REPORT_PROBLEM` | An aged report can't be used: not the detailed report, the wrong side (creditors where debtors were expected), or the wrong date | Export it again as the message says |
| `CUSTOMERS_PROBLEM` | The customers file can't be read | Export it again, or remove it: it's optional |
| `HISTORY_DIFFERENCE` | Some accounts in the history don't agree with the trial balance | Show `section: history`. Pass `acknowledge_history_differences` only once the user accepts them |
| `PREVIEW_ERROR` | The import would break one of Briefcase Ledger's rules | The message names the account or file; usually a mapping or a file to fix |

`section: issues` lists the preview's errors and warnings in full.

## Common situations

| Situation | What to do |
|---|---|
| "This isn't Xero's Trial Balance report" (or another report) | The file isn't the report as the old software exported it. Ask for a fresh export following the message's steps; never edit the file |
| An `.xls`, PDF or screenshot | Export again as Excel (`.xlsx`) or CSV |
| A file larger than Briefcase reads, or too many rows | Export the history a year at a time |
| Sage aged report refused because its side is unknown | Add the trial balance first, then add the aged report again |
| `stage` stays `ANALYSING` | Briefcase is reading the history or matching accounts. Wait `retry_after_seconds` between reads; a large history can take a few minutes |
| The start date is refused for a VAT registered company | It must be the day after the last VAT period filed in the old software. Check that date with the user and pass the corrected `declarations` |
| Cash accounting or flat rate VAT | Not supported: Briefcase Ledger supports accrual VAT only. Stop |
| A sole trader or partnership | Not supported: the import creates limited companies only. Stop |
| Too many open imports | Discard ones no longer needed with `discard: true` |
| `stage` is `EXPIRED` or `DISCARDED` | Imports stay open 30 days. Start a new one |
| `PREVIEW_EXPIRED` on execute | The preview is over 10 minutes old or the import changed since. Preview again |
| `FORBIDDEN`, or the import tools aren't available | Either assistant changes are turned off in Briefcase (Settings → AI assistants, for an administrator), or the user lacks the Financial Close → Create permission |
| `stage` is `FAILED` after the import | Posting the history failed and what it had posted was removed. Delete the client in Briefcase and import again; contact support@briefcase.so if it fails twice |
| The new client was deleted | Its import can't be reused. Start a new import |
| The user asks to use a Xero, QuickBooks or Sage connector instead | Refuse with the wording in the skill's rules and list the reports to export |
