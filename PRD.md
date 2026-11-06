# PRD: Prospect CSV import errors users can understand and fix

**Design of record:** [Prospect CSV import prototype](https://product-team-sh.github.io/prd-prospect-csv-import/prototype/) (commit [`998e84b`](https://github.com/product-team-sh/prd-prospect-csv-import/commit/998e84b), 6 Nov) · **Status:** Draft, design sign-off pending

Card: [Self-Understandable / Self-Diagnosable Error Message for PROSPECT CSV upload Module](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10153297291). Code links: webui [`f9425f2`](https://github.com/saleshandy/saleshandy-webui/commit/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5), edge [`23c74c8`](https://github.com/saleshandy/saleshandy-edge/commit/23c74c892b99392f27f5fb81bd5e3d610d1f4c11), background-jobs [`7d18274`](https://github.com/saleshandy/saleshandy-background-jobs/commit/7d18274f935ac261db9fa538aa3afe3d72d6894c), read 6 Nov 2026.

---

## 1. Problem

**What users do.** CSV import is how most prospects enter Saleshandy: 71.0% of prospects created in the 90 days to 8 Sep 2026, from about 1,836 users a week (measured, PostHog `Prospects Added`, `method = csv`).

**What happens.** Nothing is checked until the import has run. The user then gets a green toast whatever the outcome, and a report with two columns, `email,failed_reason`: one line per email, no row number, none of their other columns ([writer](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L6838)). Six different email defects all read `Invalid Email` (measured, 5 Sep). Blank-email rows are counted as failed and left out of the report. A rejected file gets a 3-second toast or an empty "Oops!". If another import is running, the dialog closes with a green toast and the file and mapping are lost ([edge L4989](https://github.com/saleshandy/saleshandy-edge/blob/23c74c892b99392f27f5fb81bd5e3d610d1f4c11/src/contacts/services/contacts-import.service.ts#L4989)).

**Why nobody finds out.** Some failures leave no trace: an invalid phone is dropped, a crashed batch loses its rows, a Do Not Contact match is enrolled silently. We can't see failures either: 0 of 409,820 CSV import events in 90 days recorded a failed or zero-row outcome, because the only event fires on success (measured, 8 Sep).

**What the fix costs.** Checking every row before import is roughly a second pass over the file (assumed; §12 Q1). A file of rows to fix hands work to the user, so it must be easy to filter in Excel and bring straight back.

## 2. Why this approach

| Approach | Rejected because |
|---|---|
| Rewrite the strings in today's report | Still no row numbers or user columns, so the row can't be found in the file. Still nothing before commit |
| Failed rows in a grid inside the dialog (the card's original ask) | A grid can't fit a 720px dialog at 40,000 rows; a reason list is five lines at any size. The download carries the rows |
| **Check before commit, reasons on screen, the user's own rows back with an error and a fix per row** | Chosen |

`ProspectLimitReached` fires for 3 to 8 users a week (measured, 8 Sep): the plan-limit state is rare but real. The prototype's 4,000-row file with 280 rows to fix is **illustrative**. **We will not claim** fewer support tickets or failed imports: neither has a baseline.

## 3. What we're building

1. **Every row is checked before anything is imported.** Review shows counts by outcome and reason, with the After Import choice applied to rows already in the list.
2. **Nothing fails silently.** Every row not fully imported is in a file of the user's own rows with its row number, a fixed error name to filter by, and one sentence on how to fix it. Every failure in the wizard says what happened and keeps the file and mapping.
3. **The flow is instrumented** from day 1.

**Invariant:** nothing is written until Import is pressed, and no failure loses the user's file or mapping.

## 4. User stories

| ID | Story |
|---|---|
| U1 | Before I import, I see how many new prospects will be added, what happens to the ones already in my list, and how many rows need fixing and why |
| U2 | I download only the rows I have to act on, in my own columns, filter by error, fix them, and upload the file back without mapping again, as many rounds as it takes |
| U3 | If my file is rejected, I'm told why and how to fix it, and can pick another file without restarting |
| U4 | If Import fails or another import is running, I stay on Review with my file and mapping |
| U5 | If my plan or credits are short, I see how many fit and can import those now |
| U6 | Importing into a sequence, I pick the step on Review and see, before importing, who won't be added and who the step can't work (no phone for a call step) |
| U8 | Bringing back a file with the same columns, I go from Upload straight to the check |
| U7 | As the product team, we can read failure rates and reasons from day 1 |

## 5. Decisions

**D1 – Every row is checked before Import.** **Consequence:** a second pass over the file. If it can't finish within budget, Review says so (§12 Q1). Counts after import (strip, bell, email) come from the import itself, not the check.

**D2 – No grid of failed rows in the dialog.** Declines the card's preview-grid ask. **Consequence:** a user with 3 bad rows still downloads a file to see them.

**D3 – No fourth step.** Import closes the dialog; progress is the existing header card; leftover work is a strip on the list plus the bell. **Consequence:** the bell holds the only lasting record.

**D4 – All four After Import options stay, default Add Missing Fields**, as today ([map-fields.tsx L74-80](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/map-fields/map-fields.tsx#L74-L80)). Rows already in the list are an outcome, never a failure. **Consequence:** every surface states that option's outcome in its own words (§6.4, §6.5).

**D5 – The file holds only rows the user must act on.** Rows repeated in the file and rows skipped under Skip Existing Prospects are counts on Review. **Consequence:** a repeated row that carried different data is visible only as a count; the first row wins, as today.

**D6 – Email stays required this release.** Whether it should follow the channel is open with Dhruv (§12 Q3). **Consequence:** LinkedIn-only lists are still blocked, now with a message instead of a dead Next button.

**D7 – The Import button counts new prospects** ("Import 3,540 new prospects"), matching the headline. Updates are stated on their own line (Hitesh, 6 Nov). **Consequence:** the button no longer equals rows sent; a file of only existing prospects reads "Update {n} prospects".

**D8 – One release** (Hitesh, 6 Nov). **Consequence:** nothing lands early; the release carries D1's risk.

**D10 – In a sequence import, the step is picked on Review, not Map Fields** (Hitesh, 6 Nov). Reverses production's placement. Every count on Review depends on the step, so it sits above them. **Consequence:** Map Fields no longer shows where prospects will go; a user who wants the step first sees it one screen later.

**D11 – Rows missing what the step needs are warned about, not held back** (Hitesh, 6 Nov). **Consequence:** an email-only prospect still lands in a call step, as today, but the user is told before Import and the rows are listed.

**D12 – A fully restored mapping skips Map Fields** (Hitesh, 6 Nov). **Consequence:** a user who wants to change one column clicks Review mapping or Back; nothing is changed silently.

**D9 – Mapping is remembered by the set of headers**, ignoring `row_number`, `error`, `how_to_fix`; account-wide, as today. **Consequence:** teammates with identical headers share a mapping (§12 Q4).

## 6. Surfaces

`{…}` are values. Copy below is what ships.

### 6.1 Upload file

Reject a bad file **in the drop zone**: file name, reason, fix, and "Choose another file". Next stays disabled. No toast and no "Oops!" anywhere in this flow.

| State | When | Copy |
|---|---|---|
| Accepted | All checks pass | {file} · {rows} rows · {columns} columns |
| Excel file | .xlsx or .xls ([pre-select.tsx L53](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/select-file/components/pre-select/pre-select.tsx#L53)) | This is an Excel file (.xlsx), so it can't be imported as it is. In Excel, choose File › Save As › CSV UTF-8 (Comma delimited), then upload that file. |
| Other type | Not .csv, or no extension | {file} can't be imported. Upload a CSV file. |
| Too large | Over 95 MB, 100,000 rows or 120 columns (today's limits) | Your file has {n} rows; the limit is 100,000. Split it into two files. (Same pattern for MB and columns) |
| Header only | No data rows | Your file has column names but no rows. |
| Duplicate headers | Two columns share a name | This file can't be used yet: two columns are both named "{header}" (columns {C} and {F}). Rename or delete one of them, save the file, and upload it again. |
| Mixed separators | More than one of `, ; | tab` in the header ([edge L1380](https://github.com/saleshandy/saleshandy-edge/blob/23c74c892b99392f27f5fb81bd5e3d610d1f4c11/src/common/response/messages/api-errors.ts#L1380-L1382)) | Your header row mixes commas and semicolons (in "{header}"). Remove the extra character or wrap the header in quotes, then upload again. |
| Not UTF-8 | | This file isn't saved in UTF-8 format. In Excel, choose File › Save As › CSV UTF-8, then upload it again. |
| Unreadable | Locked or syncing | We couldn't read this file. If it's open in Excel or still syncing, close it and try again. |

Every check that today runs only after Import (duplicate headers, mixed separators, leading empty line, upper-case `.CSV`) runs here, so Import never fails for a reason the file already showed.

### 6.2 Map Fields

- Email row marked required. No column mapped to Email: "**Map a column to Email to continue.** Pick the column that holds email addresses. Every prospect needs one." Next disabled. No email-like column at all: "**This file has no email column.** Every prospect needs an email, so none of its {rows} rows can be imported. Choose another file, or add an email column to this one."
- **After Import** below the fields: production's four labels and descriptions verbatim ([conflict-action.ts](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/map-fields/conflict-action.ts)), description shown under the select, default Add Missing Fields.
- An undecided column shows "Select field"; one set aside shows "Don't import".
- One column per field. In the list, a field already used shows "· used by {column}"; picking it moves it and says "{field} moved from {column} to {column}. One column per field."
- Cancel, Esc or the close button keeps the mapping, so the same columns come back mapped next time.
- Headers match an earlier import: "**Mapping restored.** You used a file with these same columns {when}, so they are set the way you set them then." + "Clear and map again". If the file has our three columns: "The three columns we added to the download (row number, error, how to fix) are set to Don't import."
- Field picker: search keeps focus while typing; arrow keys move through options; Enter on the search picks the first match; Esc closes the list.

### 6.3 Checking

"Checking your file", progress as "{done} of {rows} rows", "Nothing has been imported yet." One button: Cancel check.

### 6.4 Review and Import

**Layout (Hitesh, 6 Nov):** the number first, then plain grey lines for what Import will do (rows already in the list, repeats, and in a sequence import the rows turned away). Below them, one box holding only what needs the user, one row each with its own action: step-needs warning (amber), over the plan limit [Upgrade plan], rows to fix [Download]. A clean file has no box. No progress bar, no coloured dots, no note under the download. Colour appears only inside the box.

| Line | Copy |
|---|---|
| Headline | {n} new prospects will be added, from {rows} rows |
| Already in list | {n} already in your list {outcome}. After Import: {option}. Change. Outcomes: Add Missing Fields "will get their empty fields filled from your file"; Update All Fields "will be updated from your file. A blank cell clears the existing value"; Overwrite "will have their filled-in fields updated from your file"; Skip Existing "will be left unchanged" |
| Repeated | {n} repeated in this file. Only the first of each will be imported. |
| Over limit | {n} new prospects are over your plan's prospect limit and won't be added. |
| Need fixing | {n} need fixing: {count} {error name} · … (§6.7 names, lower case) [Download {n} rows] |
| Clean file | Nothing needs your attention. |

| State | When | Copy and behaviour |
|---|---|---|
| Plan limit | New prospects exceed room | Row in the box: **{over} new prospects are over your plan's limit.** Your plan has room for {room} more; rows already in your list don't count. Upgrade to add all {n}, or import now and bring the other {over} back later. [Upgrade plan] |
| Plan full | Room is 0 | Headline in neutral grey, not green. "Your plan has no room for new prospects. Upgrade, or delete prospects you no longer need, then import again." Import disabled unless there are rows to update |
| Credits short | Verification on, credits < new prospects | You need up to {n} credits and have {have}. Add credits, or untick this to import now and verify later. [Add credits]. Import disabled, as today |
| Another import running | Known when Review opens | **Another import is still running: {file}, {pct}% done.** Only one import runs at a time. You can import this file when it finishes; your file and mapping stay here. [Check again]. Import disabled |
| Import failed | Connection drops or server error after Import | **The import didn't start. The connection dropped.** Nothing was imported. Your file and mapping are kept here. Check your connection and click Import again. Button becomes "Import again" |
| Quota used meanwhile | Room hit 0 after Review loaded | Your plan's prospect limit was reached while you were importing. Nothing was imported. Upgrade, or delete prospects you no longer need, then click Import again. |
| Every row fails | 0 rows importable | "0 of {rows} rows can be imported. Every row needs fixing." Footer: Upload fixed file (left link), Back, Download {n} rows (primary) |
| One reason dominates | Every row fails and over 50% share one error | Inside the fix row: {n} of {rows} have the same problem, which often means the wrong column is mapped to Email. [Check the mapping] |

Button: "Import {n} new prospects"; "Update {n} prospects" when there are no new ones; "Importing…" while sending. Failure messages sit above the footer; the dialog never closes on failure. Verification always shows the credit balance. "Change the 15 days" for admins only; others see "Ask your account owner to change this."

### 6.5 After Import

| Surface | When | Copy |
|---|---|---|
| Progress card | Running, every page: production's card, unchanged ([async-progress-bar.tsx](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/shared/components/async-progress-bar/async-progress-bar.tsx), [header.tsx L1260](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/home/components/header/header.tsx#L1260)) | Importing {3.5k} Prospects · View List. **Closes when the import finishes.** Today it then says "Imported {total} Prospects", which counts failed rows as imported ([header-helper.tsx L247](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/home/components/header/header-helper.tsx#L247)); the toast and strip carry the outcome instead |
| Toast | Finished | {added} prospects added and {existing} {past outcome} from {file}. Past outcomes: "had empty fields filled", "were updated", "had filled-in fields updated", "were left unchanged" |
| Strip, rows to fix | Finished with rows to fix | **{n} rows from {file} weren't imported because they need fixing.** Download them, fix them in that file, and upload it here. Your mapping is remembered. [Download {n} rows] [Upload fixed rows] |
| Strip, over limit only | | **{n} rows from {file} weren't imported because they are over your plan's prospect limit.** Nothing is wrong with them. Upgrade your plan or delete prospects you no longer need, then upload the downloaded file here. |
| Strip, both | | **{n} rows from {file} weren't imported:** {fix} need fixing and {over} are over your plan's prospect limit. Download them, fix the rows that need it, make room on your plan, then upload the file here. |
| Bell | Every import | {file}: {added} added, {existing} {past outcome}. Second line as the strip's bold sentence, with a Download link. Unread state has an accessible name, not colour alone |

All counts after import come from the import (D1). Rows whose batch failed are counted in "weren't imported" and appear in the file as "Didn't finish". The strip shows on the list imported into, until dismissed or the fixed file is uploaded.

### 6.6 Completion email

Subject "Import finished: {file}". Body "{added} added, {existing} {past outcome}, {n} not imported: {top 3 error names with counts}. Download the rows to fix them." Replaces "Congratulations! Your Prospect Import is completed." for every outcome ([prospect-import.hbs L11](https://github.com/saleshandy/saleshandy-background-jobs/blob/c2c60a6ee24b9910b7aa6e67b54876bca132785c/src/app-mailer/email-templates/prospect-import.hbs#L11)). A client user's import names them and the sequence.

### 6.7 The rows-to-fix file

| Rule | Requirement |
|---|---|
| Columns | `row_number`, `error`, `how_to_fix`, then every column of the uploaded file, original order and headers |
| `row_number` | The row in the user's original file as a spreadsheet shows it: header is row 1 |
| Lines | One per row not fully imported, blank-email rows included. Imported, updated, repeated and skipped rows are not in it |
| Over-limit rows | The last new prospects in file order. Never rows already in the list |
| Several problems | `error` names the first in table order; `how_to_fix` covers every problem on the row |
| Second round | When the uploaded file already has `row_number`, `error`, `how_to_fix`, the new file **replaces** those three columns (never adds a second set) and keeps the original `row_number`. Name stays `{name}-rows-to-fix.csv`, never doubled |
| Quoting | A cell with a comma, double quote, semicolon, tab or line break is quoted; inner quotes doubled. Without the semicolon rule, LibreOffice and European-locale Excel split `a@x.com; b@y.com` and shift the row (reproduced 6 Nov) |
| Encoding, name | UTF-8 with BOM. `{uploaded name}-rows-to-fix.csv` |
| Counts | Lines equal the "weren't imported" count on strip, bell and email |
| Expiry | An expired link says "This file has expired. Files are kept for {n} days. Import your file again to get a new one." Never raw JSON (§12 Q5) |

Error names are fixed text, identical on Review, in the file, strip, bell and email. `{col}` is the user's header; `{value}` the cell.

| # | `error` | When | `how_to_fix` | Today |
|---|---|---|---|---|
| 1 | Email empty | Blank email | {col} is empty. Add this person's email address, or delete the row. | Invalid Email, line dropped |
| 2 | More than one email | 2+ addresses in the cell | {col} has {n} email addresses. Keep one. To import both, copy the row and put one address in each. | Invalid Email |
| 3 | Email missing @ | No @ | {col} "{value}" has no @. Change it to a full email address, like name@company.com. | Invalid Email |
| 4 | Email not valid | Other email failures | {col} "{value}" isn't a valid email address. Check for spaces or a missing domain, like name@company.com. | Invalid Email |
| 5 | Not a number | Number field | {col} "{value}" isn't a number. Use digits only, like 2500. | Mismatch in custom field type |
| 6 | Amount not valid | Currency field | {col} "{value}" must be a plain amount, like 1500 or 1500.50. Remove currency signs and commas. | Invalid currency value |
| 7 | Date not valid | Date field | {col} "{value}" isn't a date we can read. Write it as year-month-day, like 2026-11-06. | Invalid date value |
| 8 | Not an option | Dropdown field | {col} "{value}" isn't an option for {field}. Use one of: {up to 5}, and {n} more. Or ask an admin to add it as an option. | Invalid dropdown value |
| 9 | Value too long | Over 1,000 chars (10,000 long text) | {col} has {n} characters. The limit for {field} is {limit}. Shorten it. | Field value length exceeded maximum limit |
| 10 | Owned by a teammate | Another user's prospect, sharing off | Nothing is wrong with this row. {email} is already a prospect owned by {owner}, and prospect sharing is off in your account. Ask {owner} or an admin to share it with you. | Prospect already present with different owner |
| 11 | Over plan limit | New prospect past the room left | Nothing is wrong with this row. Your plan's prospect limit was reached before it. Upgrade your plan or delete prospects you no longer need, then upload it again. | Import failed due to prospect limit reached |
| 12 | Didn't finish | Its batch crashed | Nothing is wrong with this row. It didn't import because of a problem on our side. Upload it again. | **Rows vanish** |
| 13 | Already in this sequence | Sequence import | Already in this sequence (step {n}), so it wasn't added again. {saved} | Prospect already present in another step of same sequence |
| 14 | In another sequence | Admin setting | Already in "{sequence}". Your account allows a prospect in one sequence at a time, so it wasn't added. An admin can change this in Settings. {saved} | Prospect already present in another sequence |
| 15 | Active in another sequence | Admin setting | Active in "{sequence}". Your account doesn't allow a prospect to be active in two sequences, so it wasn't added. Add it after that sequence ends for them, or ask an admin to change the setting. {saved} | Prospect is active in another sequence |
| 16 | Email bounced before | Sequence import | Emails to this address bounced before, so it wasn't added. Check the address in {col}; if it's wrong, fix it and upload the row again. | Bounced prospect |
| 17 | Unsubscribed | Sequence import | This person unsubscribed, so they weren't added. Nothing to fix. | Unsubscribed prospect |
| 18 | On Do Not Contact list | Sequence import | Added to the sequence, but won't be emailed: {email or domain} is on your Do Not Contact list. Remove it from the list to email them. | **Enrolled silently** |
| 19 | Phone not valid | Not 7 to 15 digits | Imported without the phone. {col} "{value}" needs 7 to 15 digits. Fix it and upload the row again to add it. | **Phone dropped silently** |

`{saved}`: "Its details were updated from your file.", or under Skip Existing Prospects, "Its details were left as they were." Assignment sites in [contacts-import.service.ts](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts): repeat L2534, email L2543, field type L2551, length L2563, owner L1115, limit L1834, Skip Existing L2747, same sequence L3021, other sequence L2974, active L2924, bounced and unsubscribed L4510, DNC L4530, batch crash L6470; phone [contact-helpers.ts L243](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/helpers/contact-helpers.ts#L243). **Delete, never assigned:** `NoUpdate`, `NoOverwrite`, `ProspectInCallDNC`, `ContactInDncList`. `MultipleSeparator` is reachable only from a sync route the web app doesn't call.

### 6.8 Import via CSV into a sequence

Same dialog, Upload and Map Fields as the list import ([sequence path today](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/shared/add-to-step-select/add-to-step-select.tsx#L13)). Differences (Hitesh, 6 Nov):

| Where | Requirement |
|---|---|
| Entry | Sequence with no steps: blocked before the dialog opens, "Add at least one step to this sequence before adding prospects." [Go to Steps] |
| Map Fields | No step picker (production has one here; it moves to Review) |
| Review, first row | "Import into [Step 1 · Email ▾]", in Review's settings layout. Options name the step type ("Step 2 · LinkedIn connection request"). Default: the lowest step number, as production ([map-fields.tsx L92-L99](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/map-fields/map-fields.tsx#L92-L99)). Changing it updates every line below at once |
| Headline | "{n} prospects will be added to Step {k}, from {rows} rows". Counts new plus existing, because production adds existing prospects to the step under every After Import option |
| Lines | "{n} already in your list will be added too, and {outcome}." · Step needs (amber): "{n} have no phone number, so their call task will have nothing to dial. They still import; the download lists them." When no column is mapped to the field: "{n} have no phone number: no column is mapped to Phone Number, so every call task will have nothing to dial." [Map a phone column] or pick another step. Same for LinkedIn steps (LinkedIn profile URL, "nothing to open") · "{n} are in another sequence and won't be added. Your admin setting allows one sequence per prospect." [Change setting], admins only · "{n} are already in this sequence and won't be added again." · "{n} bounced or unsubscribed before and won't be added." · Repeated and need fixing as the list import |
| Button | "Add {n} to Step {k}" |
| Download | Includes the rows not added (errors 13 to 17) and, when only some rows lack it, the step-needs warnings "No phone for call step", "No LinkedIn URL for LinkedIn step" |
| After | Toast "{n} added to Step {k} of {sequence} from {file}." Strip on the sequence's Prospects tab, adding "The download also lists the {n} saved but not added to Step {k}." Bell "{file}: {n} added to Step {k} of {sequence}, {m} not added". Email subject "Import finished: {file} into {sequence}" |

The admin setting is "Allow adding duplicate prospects in multiple sequences" (Settings › Admin Settings › Prospects). It only affects prospects already in the list.

### 6.9 Bringing a file back

When a file's headers match a mapping that was used for a check, every column comes back decided. Then Map Fields has nothing to ask, so Upload skips it (Hitesh, 6 Nov):

- Under the file: "**Mapping restored.** You used these same columns {when}, so all {n} are set." [Review mapping]
- Next becomes "Check {rows} rows" and goes straight to the check; the steps show Map Fields as done, and Back on Review opens it.
- Anything undecided, or Email not mapped: Map Fields as usual.
- Applies to the fixed file (our three columns are set to Don't import) and to any weekly import with the same columns.

### Acceptance criteria

| ID | Case | Expected | P |
|---|---|---|---|
| A1 | Upload .xlsx, header-only, duplicate headers, mixed separators | Each rejected in the drop zone with §6.1 copy; Next disabled; no toast | P0 |
| A2 | No column mapped to Email; then a file with no email column | The matching §6.2 banner; Next disabled | P0 |
| A3 | 4,000 rows: 220 no @, 40 blank, 20 with two emails, 160 existing, 20 repeated | Review: 3,540 new; 160 existing with the option's outcome; 20 repeated; 280 need fixing with three reasons; button "Import 3,540 new prospects" | P0 |
| A4 | A3 under each After Import option | Review, toast and bell use that option's words; 3 of the 160 read back in the list match the option | P0 |
| A5 | Download from A3 | 280 lines; header `row_number,error,how_to_fix,` + original headers; only §6.7 error names; 40 blank-email rows present; row numbers match the source in Excel | P0 |
| A6 | Open A5 in LibreOffice with tab, `;` and `,` separators | Every row has the same column count | P0 |
| A7 | Upload A5 back, import with a plan limit, download again | Mapping restored; one set of our three columns; original row numbers kept; name not doubled | P0 |
| A8 | Plan room 2,500; A3's file | 2,500 added; 1,040 "Over plan limit" lines, all new prospects, row numbers within the file | P1 |
| A9 | Import fails; another import running | Dialog stays on Review with §6.4 copy; file and mapping kept; busy known before Import | P0 |
| A10 | Every row fails | "0 of {rows}"; Upload fixed file and Download in the footer; dominant-reason banner when over 50% | P1 |
| A11 | Credits short | Credit copy with numbers; Import disabled until unticked or topped up | P1 |
| A12 | Force a batch to fail mid-import | Its rows in the file as "Didn't finish"; strip, bell and email count them | P0, **sign off by name, owner: Rajat** (rows that vanish are lost data) |
| A13 | Invalid phone on a new prospect | Imported, phone blank, "Phone not valid" line | P1 |
| A14 | Sequence import: in another sequence, bounced, unsubscribed, on DNC | Each in the file with errors 13 to 18 | P1 |
| A15 | Email for a partial and a fully failed import | §6.6 copy with real counts; never "Congratulations" | P1 |
| A16 | Keyboard only through the wizard | §8 keyboard rules hold; screen reader announces failures and the strip | P1 |
| A17 | 390px wide | Map rows stack, no sideways scroll; strip wraps with buttons below; footer buttons full width, primary last; toast never covers the footer | P1 |
| A18 | Every step's footer at 1440px | Secondary is the gray button immediately left of the primary, 16px apart, primary's right edge at the footer padding; Back on Map Fields and Review, Cancel on Upload | P2 |
| A19 | Map two columns to Email | The second pick moves Email and says so; never two columns on one field | P1 |
| A20 | Sequence import, step changed from Email to Call on a file with no phone column | Headline, lines and button update at once; the no-phone line says no column is mapped and offers Map a phone column | P1 |
| A21 | Sequence import with prospects already in another sequence, admin setting off | Review counts them before Import; they are in the download as "In another sequence"; the toast counts only those added | P0 |
| A22 | Upload the fixed file from A5 | Upload shows Mapping restored; Next reads "Check {rows} rows" and lands on Review without Map Fields | P1 |

## 7. Reconciliation with existing behaviour

| Path | Today | After |
|---|---|---|
| CRM list › Import CSV (`/contacts/import-async`) | Two-column report | This PRD |
| Sequence › Import via CSV (`/contacts/sequence-async/:stepId`) | Same, sequence reasons | This PRD, §6.8 |
| Public API import | 7 reasons have no status code, so `status` is undefined ([gateway enum](https://github.com/saleshandy/saleshandy-open-api-gateway/blob/3561cf80b480bb48cf0670c22a9c7f977012a4fc/src/common/enums/prospect-import-failed-reason.ts#L1)) | Copy out of scope; add the missing codes |
| Lead Finder bulk add | Shares the import service | Out of scope; must not change |
| Legacy sync `importFromFile` | Not called by the web app | Untouched |

**Overwrite** only writes fields the prospect already has a value in ([L3290](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L3290-L3300)). Production's description already says so ("Fields that are currently empty will not be filled in"), and §6.2 keeps it verbatim, so behaviour stays. Repeats are caught only within a 1,000-row chunk ([L2534](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L2534)); D1's check covers the whole file, so the import must too, or counts drift.

## 8. Cross-cutting

| Rule | |
|---|---|
| No green toast, success icon or "Congratulations" for an outcome with failures | Release blocker |
| No "Oops!", raw server text or JSON shown anywhere in this flow | Release blocker |
| One name per error, same words on every surface | Release blocker |
| Dialog: `role="dialog"`, `aria-modal`, labelled by its title; Tab stays inside; Esc closes an open list, then the dialog | |
| Focus stays where it was when the dialog re-renders; a new step moves focus to the dialog title | |
| Every action is a button or link, never plain text with a click handler | |
| Failures use `role="alert"`; strip and toast are live regions | |
| Text contrast at least 4.5:1, including inactive steps and placeholders; no functional text under 11px | |
| Below 720px: map rows stack (name and picker, samples beneath); strip buttons wrap below the message | |
| Dialog footer on every step, as production's import dialog ([_import-contacts-modal.scss L121](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/import-contacts-modal/_import-contacts-modal.scss#L121-L139)): 20px 24px padding, 16px gap. Left: an optional blue link with a 20px icon (Try sample file; Upload fixed file). Right: the gray secondary button ([`gray-secondary-btn`](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/assets/css/app.scss#L811)), labelled Cancel on Upload, Back on Map Fields and Review, Cancel check on Checking; then the primary. Below 720px, link then buttons, full width, primary last | |
| Links to places outside the dialog (Upgrade plan, Add credits, Change the 15 days) are real links to Billing, the credits checkout and Smart Verification settings | |
| White-label: no "Saleshandy" in any message | |

## 9. Implementation notes

- **Pre-import check (D1):** same row checks as the import plus the existing-prospect lookup, over the whole file. **Resolve before estimation:** time on 100,000 rows, and which service runs it (§12 Q1).
- **Failed-row storage** moves from one entry per (reason, email) via `$addToSet` ([L6717](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L6717)) to one per source row with its number and cells. **Resolve before estimation:** size for 100,000 rows of 120 columns.
- **All problems per row:** the check chain at [L2534-L2563](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L2534-L2563) stops at the first hit. If removing that is costly, ship the first problem only.
- **Busy check at Review** needs an endpoint for "is an import running for this user".
- Not adopted: in-dialog row grid (D2); in-app editing of failed rows.

## 10. Measurement

**Privacy:** never log emails, cell values or file names. Counts and error names only.

| Event | Fires when | Properties |
|---|---|---|
| `csv_import_step_viewed` | Each step opens | `step`, `path` (list / sequence), `file_rows`, `file_columns` |
| `csv_import_file_rejected` | Upload rejects a file | `reason` |
| `csv_import_blocked` | Next or Import disabled with a message | `reason` (`email_not_mapped`, `credits_short`, `plan_full`, `busy`) |
| `csv_import_reviewed` | Review shown | `new`, `existing`, `repeated`, `over_limit`, `to_fix`, `reasons`, `after_import` |
| `csv_import_failed` | Import press fails | `reason` (`connection`, `quota`, `server`) |
| `csv_import_completed` | Job finishes | `total_rows`, `added`, `updated`, `not_imported`, `reasons`, `duration_ms`, `path` |
| `csv_import_report_downloaded` | Any download | `rows`, `from` (review / strip / bell / email) |
| `csv_import_reuploaded` | A file with our three columns is uploaded | `mapping_restored`, `round` |

| Indicator | Definition | Healthy | Investigate |
|---|---|---|---|
| **Fix loop closed** (most important) | Imports with rows to fix followed by a re-upload within 7 days | Set at day 14 (no baseline) | Below 20% (assumed) |
| Silent dead end | Map viewed with no Review in the session | Falling week on week | Flat |
| Rows not imported | `not_imported / total_rows` | Set at day 14 | |
| Count drift | `completed.not_imported` vs `reviewed.to_fix` | Within 1% (target) | Over 1% |

**Guardrails:** median Checking time over 30 s for files under 10,000 rows (target, Rajat to confirm): move the check behind a fallback. Any import whose `not_imported` exceeds its file's lines: halt (A12 failing live).

Instrumentation ships with the feature.

## 11. Rollout

| Stage | Who | Gate |
|---|---|---|
| 1 | Internal accounts | A1 to A22 pass on the 17 QA fixtures; §10 events fire |
| 2 | 10% of accounts (blast-radius control, not an A/B test) | One week; count drift within 1%; Checking guardrail holds |
| 3 | Everyone | Stage 2 gates held |

## 12. Open questions

| # | Question | Why it matters | Recommendation |
|---|---|---|---|
| Q1 | Can the full check run on 100,000 rows in time? **Blocking** | D1 rests on it | Rajat sizes it. Fallback: check the first 10,000, say "Checked 10,000 of {rows}", let the import report the rest |
| Q2 | Can the job report progress to the header card? | §6.5 | Batches done over total batches |
| Q3 | Should the required field follow the channel? (Dhruv) | D6 | Ship with Email required; decide separately |
| Q4 | Per-user remembered mappings? (Hemanshu) | D9 | Keep account-wide this release |
| Q5 | How long is the report file kept? | §6.7 expiry copy | Find the S3 rule; if none, 30 days |
| Q6 | Are DNC-flagged prospects truly never emailed? | Error 18 wording | Rajat confirms before copy freeze |
| Q7 | Show the owner's name in error 10? | Privacy within a team | Show it; same account |

## 13. Priority & timeline

Card priority P0 (Backlog). Target: not set. Release gate: A1 to A22, A12 signed off by Rajat. **Owners still required:** backend owner for the check and file (Rajat to name), frontend owner, QA owner.

## Appendix – reference index

- Design: [prototype](https://product-team-sh.github.io/prd-prospect-csv-import/prototype/) (the Account menu in its top bar switches plan limit, low credits, Import fails, another import running; sample files cover rejected, no-email and every-row-fails), [design canvas](https://product-team-sh.github.io/prd-prospect-csv-import/). Saleshandy production styles (primary `#1D4ED8`, Inter, 720px dialog).
- QA: 17 CSV fixtures (malformed emails, ragged rows, separators, encodings, duplicate headers, header-only, LinkedIn-only) kept with the design files.
- Reviews: [7 Sep](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10153297291#__recording_10278488304), [8 Sep](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10153297291#__recording_10280390333).
