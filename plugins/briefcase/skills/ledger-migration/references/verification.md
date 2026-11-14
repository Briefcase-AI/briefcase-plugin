# Import checks

`briefcase_verify_ledger_import` runs twelve checks on an imported Briefcase Ledger client. Each has a `status` (`PASS`, `WARNING`, `FAIL` or `SKIPPED`), a `summary`, an `action` when there is something to do, and rows with the figures (`expected`, `actual`, `difference`, as decimal strings).

## Verdicts

| Verdict | Means |
|---|---|
| `TIES` | Every check passed or was skipped. The books came across intact. |
| `TIES_WITH_WARNINGS` | Nothing failed, but at least one thing needs a look. Usually set-up left to do, such as the lock date or depreciation. |
| `DOES_NOT_TIE` | At least one check failed. The books in Briefcase differ from the files the import used. |
| `NOT_FINISHED` | History or open items are still posting. Run the checks again in a few minutes. |

## The checks

| Code | What it checks | A `WARNING` means | A `FAIL` means |
|---|---|---|---|
| `POSTING_COMPLETE` | Everything the import read was posted | Still posting | Posting failed and what it had posted was removed, or not everything read was posted. Delete the client and import again |
| `JOURNALS_BALANCED` | Every journal the import posted balances, and history sits inside its window | — | An import fault, not something to fix by hand. The user contacts support@briefcase.so |
| `OPENING_BALANCES_NIL` | Opening Balances nets to nil at the trial balance date | It holds an amount the import meant to leave there, for example uncleared bank items, and the summary says why | It holds more than the import meant to leave: part of the trial balance didn't reach its own account. Usually debtors or creditors imported without aged reports |
| `TRIAL_BALANCE_TIES` | Every balance agrees with the trial balance at the day before the start date, and at the previous year end when the import had last year's figures | Some trial balance rows couldn't be matched to an account; check them against the balance sheet by hand | Accounts disagree. The rows give each account and both figures |
| `BANK_TIES` | Each bank account opened at the balance the import was given, and agrees with a connected feed | A feed difference, usually a feed that started late or lines still to come | A bank account opened at the wrong balance |
| `OPEN_ITEMS_TIE` | The imported invoices and bills add up to the aged reports, and those to the debtors and creditors in the trial balance | — | They don't. Usually aged reports run at a different date from the trial balance |
| `MAPPING_REVIEW` | Accounts imported as a different type from Briefcase's suggestion, and suggestions made with low or medium confidence | Each account listed deserves a look against what moves through it. An account money moves through should be a bank account | — |
| `PROFIT_BY_YEAR` | Each year's profit in Briefcase matches the old books | A year's profit moved, because an account moved between the profit and loss and the balance sheet, or a profit and loss account differs. Matters most for a year already filed | — |
| `CONNECT_CLIENT_STILL_ACTIVE` | No Xero or QuickBooks client for the same company is still running in Briefcase | One is, so it can keep posting to the old books: Autopilot, depreciation, prepayment and deferred income schedules | — |
| `FIXED_ASSETS_WITHOUT_SCHEDULES` | Every fixed asset account with a balance has a depreciation schedule | The import doesn't bring schedules across, so depreciation has stopped on the accounts listed | — |
| `ACTIVITY_SINCE_IMPORT` | Nothing posted since the import changed a balance at the start | Later journals dated before the start changed balances. These are later changes, not import errors | — |
| `LOCK_DATE` | The ledger is locked to the day before the start date | It isn't, so entries can still land inside the imported books | — |

`SKIPPED` means there was nothing to compare, for example no record of Briefcase's account suggestions for an older import. Say so in the report; it is not a pass.

Two facts the checks rely on:

- **Retained earnings** can differ from the trial balance by profit that Briefcase keeps in the profit and loss accounts (from the history, or a year's profit folded in at the import). The check's summary states that amount; it is not a difference.
- **The checks read the journals the import posted,** so a difference caused by later bookkeeping shows only under `ACTIVITY_SINCE_IMPORT`.

## Report template

```
## <Client>: Briefcase Ledger import check

**Verdict:** <one or two sentences: intact or not, how many problems, what they block>

Imported from <source> on <date>. First day in Briefcase <start date>; balances at <trial balance date>; history <from> to <to>, or no history.

### What checks out
- **<Check title>.** <the check's summary in a sentence>

### Problems
**1. <Plain statement of the problem>**
<the figures that matter: account, expected, in Briefcase, difference>
- **What it means:** <effect on the balance sheet, profit, VAT or a filed year>
- **What to do:** <the check's action; while little has happened since, delete and import again>

### To look at
- **<Check title>.** <summary>. <action>

### Not checked
- <skipped checks and why; the company facts if they couldn't be checked>

### Finishing the move
- [ ] <each step left: lock date, depreciation, the old client, bank feeds, HMRC, inviting the business owner>
```

Use plain figures (£, dates) and short sentences, and keep IDs out. Then offer to run the checks again once the accountant has made changes.
