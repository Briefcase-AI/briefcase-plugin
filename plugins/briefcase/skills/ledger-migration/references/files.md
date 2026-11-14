# Getting the files to Briefcase

Briefcase needs the original files, exactly as the old software exported them: Excel (`.xlsx`) or CSV. It works out which report each file is from its content, so they can arrive in any order and under any name.

Never open, edit, convert, merge or retype a file, and never create one. If the user only has a PDF, a printout, a screenshot or figures copied into their own spreadsheet, ask them to export the report again from the old software. If a file is too large, export the history a year at a time.

## Which route

| Assistant | Route |
|---|---|
| Claude (claude.ai, desktop and mobile apps) | The files page. Claude can't pass a file to Briefcase's tools |
| ChatGPT | The user attaches the exports in the chat; pass each as `file` |
| Claude Code, Codex or another assistant that can run commands | An upload link per file |
| Any assistant | The files page always works |

## The files page

Give the user `files_page_url` from `briefcase_start_ledger_import`. They sign in to Briefcase, tick that they exported the files from the old software's own reports, and drop the files in. The page shows the reports to export and what Briefcase read from each file. It works for as long as the import is open (30 days).

When they say they're done, read the import with `briefcase_get_ledger_import`.

## An attachment (ChatGPT)

Pass the attached file to `briefcase_update_ledger_import` as `file`, one per call, with its `file_name`. Briefcase downloads the original from the chat and returns what it read in `files_added`.

## An upload link (Claude Code and others)

1. `briefcase_update_ledger_import` with `prepare_upload` set to the file's name. It returns `upload.upload_url` and `upload.file_id`; the link expires at `upload.expires_at`, an hour later.
2. Upload the file's bytes unchanged with an HTTP `PUT`, for example `curl -X PUT --upload-file "Trial Balance.xlsx" "<upload_url>"`.
3. `briefcase_update_ledger_import` with `attach_uploaded_file_ids` set to the `file_id`. It returns what Briefcase read in `files_added`.

## Files that came straight from the assistant

An attachment or an upload link skips the files page's tick box. Before the import runs, the preview shows `export_confirmation_needed: true`: ask the user to confirm the files are the old software's own exports, and pass `user_confirmed_files_are_source_exports: true` on execute only after they say so.

## A wrong file

A file Briefcase can't use comes back `REJECTED` with a problem that says what was expected and how to export it. Pass that on, remove the file with `remove_file_ids` and add the new export.
