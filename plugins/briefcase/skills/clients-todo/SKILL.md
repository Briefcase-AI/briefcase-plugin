---
name: clients-todo
description: See which Briefcase clients need attention, across every client at once, and act on it. Lists bills and receipts to review, expense claims to publish, schedules to post, unreconciled bank lines, bank feeds and HMRC connections that have lapsed or are about to, VAT returns and MTD for Income Tax updates due, month-ends still open or not started, paperwork clients still owe and overdue debtors, then offers the next actions and carries them out with a preview first. Use when the user asks which clients need attention, what is due this week, what they are waiting on from clients, what is outstanding across their clients or what they should work on, including when they say entities or companies.
---

# Which clients need attention

One read covers every client the user connected. Turn it into a short list of what to do, most urgent first, then help do it. Work as the signed-in Briefcase user.

Rules that apply throughout:

- **Nothing changes without the user's yes.** Preview first wherever the tool has a preview, show the preview, and execute only after they agree.
- **One client at a time.** A yes never covers changes for more than one client.
- **Figures come only from tool responses,** quoted as returned. Never estimate a count, an amount or a date, and never state a VAT or MTD date the tool didn't return.
- **Use the user's own word** for a client: "client", "entity" or "company".
- **The user's own rules win** over the defaults below, for this request: "treat Initech as quarterly", "ignore schedules", "VAT within 14 days only", "debtors over £5,000", "show everything".

## 1. Read every client

1. `briefcase_list_client_attention`. Pass `page.next_cursor` as `after` until `page.has_next_page` is false, and write nothing until every page is read. Pass `due_within_days` only when the user wants a window other than 30 days; overdue dates always come back.
2. Each client has its `items`, a `ledger_type` (`BRIEFCASE` for Briefcase Ledger, `XERO` or `QUICKBOOKS`) and a `web_url`. Only items with something to report are returned.
3. `unavailable_kinds` lists what the user has no permission to see. Name each once, with the permission it needs. Never present a kind you couldn't read as clear.
4. Bank lines, bank feeds, VAT, MTD for Income Tax, the HMRC connection and debtors exist for Briefcase Ledger clients only. Month-end exists for Xero clients and Ledger limited companies. When the list includes Xero or QuickBooks clients, say once which of these Briefcase doesn't have for them.

## 2. Read debtors for Ledger clients

For up to 15 Briefcase Ledger clients, in the order the list returned them, call `briefcase_get_aged_report` with `subledger: ACCOUNTS_RECEIVABLE`. A client gets a debtors line when `totals.two_to_three_months` or `totals.more_than_3_months` is above zero, which is more than two months overdue; quote those totals. Otherwise leave its debtors out. If there were more than 15 Ledger clients, say how many you didn't read and offer to read them.

## 3. Decide what gets a line of its own

The tool returns everything. Long-standing backlogs are grouped so the list stays readable; "show everything" gives every client its own line.

| Item | A line of its own when | Otherwise |
|---|---|---|
| `TRANSACTIONS_TO_REVIEW` | `newest_at` is in the last 30 days | Summed under "older transaction backlogs" |
| `UNRECONCILED_BANK_LINES` | `newest_date` is in the last 30 days | Summed under "older unreconciled bank lines" |
| `SCHEDULES_TO_POST` | `has_posted_before` is true | Summed under "draft schedules where none has ever been posted" |
| `MONTH_END`, state `OPEN` | `days_since_period_end` is 30 to 120 | Over 120: summed under "month-ends open for more than 120 days". Under 30: not mentioned, the close is in progress |
| `MONTH_END`, state `NOT_STARTED` | Always | – |
| `VAT_RETURN_DUE`, `INCOME_TAX_UPDATE_DUE` | Always | – |
| `BANK_FEED`, `PAPERWORK_REQUESTS`, `EXPENSE_CLAIMS_TO_PUBLISH` | Always | – |
| `HMRC_CONNECTION` | State `EXPIRED`, `EXPIRING_SOON` or `DATES_NOT_CONFIRMED`, or `NOT_CONNECTED` with a `dates_last_confirmed_at` | `NOT_CONNECTED` with no `dates_last_confirmed_at`: summed under "never connected to HMRC" |
| Debtors | More than two months overdue (step 2) | Not mentioned |

## 4. Write the list

Four groups, in this order. Leave out a group with nothing in it.

- **Today:** a VAT return or MTD update that is overdue (`days_until_due` below zero) or due within 7 days; a bank feed that is `EXPIRED`, `DISCONNECTED` or in `ERROR`, or expires within 7 days; an HMRC connection with a line of its own, except one `EXPIRING_SOON` more than 7 days away.
- **This week:** VAT and MTD dates due later than that; the other expiring feeds and HMRC connections; and the work queues: transactions, expense claims, schedules, bank lines, month-end and debtors.
- **Waiting on others:** open paperwork requests, with how many are overdue (`overdue_request_count`) and when the last reminder went (`last_reminder_at`).
- **Long-standing:** one line per kind of backlog from step 3, with the total and every client named. For example: "3 clients have older transaction backlogs with nothing new in the last 30 days (60 documents): …".

In the first three groups write one line per client, most urgent first, with everything that client has in that group and its `web_url` as a link. A client can appear in more than one group. End with how many clients are clear: `totals.clients_checked` less `totals.clients_with_items`.

How to say each thing:

- **HMRC dates.** When `hmrc_dates_as_of` is set, say once when the dates were confirmed, in UK time: "HMRC dates as of 02:05 this morning". Briefcase refreshes them from HMRC each night; a date HMRC hasn't confirmed in the last 26 hours isn't returned.
- **HMRC connection.** For a client with an `HMRC_CONNECTION` item and no VAT or MTD item, say "HMRC dates unavailable since" its `dates_last_confirmed_at`, and why: the connection has expired, the client is no longer connected, or Briefcase could not confirm the dates with HMRC (`DATES_NOT_CONFIRMED`). Tell the user to check the deadline with HMRC. With `DATES_NOT_CONFIRMED` the connection is still in place, so don't tell them to reconnect; if it stays that way, they should contact Briefcase support.
- **Failed documents.** When `failed_count` is above zero, say how many of the documents failed processing.
- **Schedules.** Give the four counts by type: prepayments, deferred income, fixed assets and accruals.
- **Bank lines.** Give the count and the bank accounts with the most lines.
- **Bank feeds.** Name the bank (`institution_name`), the state, how many bank accounts it feeds and `consent_expires_on`.
- **Month-end.** Give `period_end`. For `NOT_STARTED`, say no working paper covers the period that ended then. `cadence` says whether the client closes monthly or quarterly; "treat X as quarterly" overrides it.

## 5. Offer what to do next

Offer up to three actions, most urgent first, each as something you can do now. List separately what only the user can do, with the client's link and where to go in Briefcase.

What you can do now:

| Item | Offer | How |
|---|---|---|
| Transactions to review | Go through the queue: explain each document, fix the coding, publish or archive | The transaction review skill |
| Expense claims to publish | Review each claim and publish it | The expense claims skill |
| Schedules to post | Review the drafts, correct them, publish | The financial close review skill |
| Unreconciled bank lines | List the lines with `briefcase_list_bank_transactions` and `unreconciled_only: true`. If the user has the missing bill or receipt, upload it so Briefcase can match it | The document submission skill |
| Debtors | Draft a chaser for each overdue customer, listing the invoices and what is outstanding: `briefcase_get_aged_report` with the `contact_id`, and `briefcase_get_contact` for who to write to. The user sends it; you never do | The sales invoicing skill, "See what is outstanding" |
| Month-end still open, Ledger client | Say what blocks the close with `briefcase_get_working_paper`, then clear the unpublished bills and draft schedules it counts | The financial close review skill |

What only the user can do, in Briefcase:

| Item | Where |
|---|---|
| Reconcile a bank line | The bank account under Bank |
| Reconnect or extend a bank feed | Bank → Manage accounts. It needs the user's own bank login |
| Connect or reconnect HMRC (`NOT_CONNECTED`, `EXPIRED`, `EXPIRING_SOON`) | Business settings → HMRC. It needs the user's own HMRC login |
| Review and submit a VAT return | VAT Return |
| Submit an MTD for Income Tax update | MTD ITSA |
| Start a month-end, sign accounts off or close the period; for a Xero client, also see what blocks the close | Working Papers |
| Chase paperwork | Tasks, where each open request and its reminders are |

Before offering a change, check once with `briefcase_get_connection` that `workspace.assistant_writes_enabled` is true. If it is false, or a tool returns `FORBIDDEN`, say once that changes are switched off or which permission is missing, and give links instead.

For Xero and QuickBooks clients, act through the same skills where one applies (transactions, expense claims, schedules) and give links for the rest.

## 6. Act, then read again

1. When the user picks an action, follow that skill's steps for that client: preview, show, and execute only on a yes.
2. Uploading a document has no preview, because how Briefcase reads and codes it is only known after processing. Upload only a file the user gave you, name the client first, and say so when Autopilot is on for that client and may publish it.
3. Afterwards call `briefcase_list_client_attention` with that client in `client_ids`, and say what is left for it.
4. Then offer the next action.
