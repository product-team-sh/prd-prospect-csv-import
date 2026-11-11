# Copy spec: Prospect CSV import errors

Companion to [PRD.md](PRD.md). **See every string rendered on its surface: [copy.html](copy.html)** (built from the prototype's styles). Every string here ships as written; `{…}` are values. Section numbers match the PRD's §5. Code links: webui `f9425f2`, edge `23c74c8`, background-jobs `7d18274` (read 6 Nov 2026).

### 5.1 Upload file

A rejected file (Hitesh, 11 Nov): centred in the drop zone like the other Upload states. File icon and file name, a **title**, one sentence on the fix, a rule, then "Drop the fixed file here, or choose a file". The zone stays live, so the fixed file goes straight back in. Next stays disabled. No toast and no "Oops!" anywhere in this flow. See it: [copy.html §5.1](copy.html#s5-1).

**Only what the check knows.** Each message is limited to facts its check already has ([pre-select.tsx L41-L86](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/select-file/components/pre-select/pre-select.tsx#L41-L86), [select-file.tsx L49-L95](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/select-file/select-file.tsx#L49-L95)): the file name, the extension, and whether a limit was crossed. No row totals (parsing stops at the limit), no header names or column positions (the duplicate and blank checks return yes or no).

| State | Check knows | Title | Fix |
|---|---|---|---|
| Accepted | Rows and columns parsed | {file} · {rows} rows · {columns} columns | |
| Excel file | Extension .xlsx or .xls | Excel files can't be imported | In Excel, choose **File › Save As** and pick **CSV UTF-8 (Comma delimited)**. |
| Other type, or no extension | Any other extension | This file type can't be imported | Only CSV files can be imported. Save or export it as a CSV. |
| Too large | Over 95 MB | This file is over 95 MB | Split it into smaller files, or remove columns you don't need. |
| Too many rows | Past 100,000 rows (no total) | This file has more than 100,000 rows | Split it into files of 100,000 rows or fewer, and upload them one at a time. |
| Too many columns | Over 120 columns | This file has more than 120 columns | Remove the columns you don't need. |
| No prospects | No data row | This file has no prospects | It has column names but no rows. Add the prospects under the header row. |
| Repeated column names | Two or more headers match (yes or no) | Some columns have the same name | Each column needs its own name so we know where it goes. Rename or delete the repeated columns, then save the file. |
| Columns with no name | Two or more blank headers (yes or no) | Some columns have no name | Every column needs a name in the first row. Name the empty ones, or delete those columns. |
| Mixed separators | Header uses more than one of `, ; | tab` (moves from the server, [edge L1380](https://github.com/saleshandy/saleshandy-edge/blob/23c74c892b99392f27f5fb81bd5e3d610d1f4c11/src/common/response/messages/api-errors.ts#L1380-L1382)) | The header row mixes separators | Use one separator throughout, or put quotes around headers that contain a comma. |
| Not UTF-8 | Unreadable bytes (existing `is-utf8-file` util) | This file isn't saved as UTF-8 | In Excel, choose **File › Save As** and pick **CSV UTF-8 (Comma delimited)**. |
| Unreadable | Browser read error | We couldn't read this file | If it's open in Excel or still syncing, close it or wait, then try again. |

Every check that today runs only after Import (duplicate headers, mixed separators, leading empty line, upper-case `.CSV`) runs here, so Import never fails for a reason the file already showed.

### 5.2 Map Fields

- Email row marked required. No column mapped to Email: "**Map a column to Email to continue.** Pick the column that holds email addresses. Every prospect needs one." Next disabled. No email-like column at all: "**This file has no email column.** Every prospect needs an email, so none of its {rows} rows can be imported. Choose another file, or add an email column to this one."
- **After Import** below the fields: production's four labels and descriptions verbatim ([conflict-action.ts](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/map-fields/conflict-action.ts)), description shown under the select, default Add Missing Fields.
- An undecided column shows "Select field"; one set aside shows "Don't import".
- One column per field. In the list, a field already used shows "· used by {column}"; picking it moves it and says "{field} moved from {column} to {column}. One column per field."
- Cancel, Esc or the close button keeps the mapping, so the same columns come back mapped next time.
- Headers match an earlier import: "**Mapping restored.** You used a file with these same columns {when}, so they are set the way you set them then." + "Clear and map again". If the file has our three columns: "The three columns we added to the download (row number, error, how to fix) are set to Don't import."
- Field picker: search keeps focus while typing; arrow keys move through options; Enter on the search picks the first match; Esc closes the list.

### 5.3 Checking

"Checking your file", progress as "{done} of {rows} rows", "Nothing has been imported yet." One button: Cancel check.

### 5.4 Review and Import

The number first, then plain grey lines for what Import will do. Below, one box with only what needs the user, one row each with its action. A clean file has no box. No bar, no coloured dots. See it rendered: [copy.html §5.4](copy.html#s5-4).

| Element | Copy |
|---|---|
| Headline | {n} new prospects will be added |
| Already in list | {n} already in your list {outcome}. [Change]. Outcomes: Add Missing Fields "will get their empty fields filled from your file"; Update All Fields "will be updated from your file. A blank cell clears the existing value"; Overwrite "will have their filled-in fields updated from your file"; Skip Existing Prospects "will be left unchanged" |
| Repeated | {n} repeated rows: only the first of each is imported. |
| Clean file | Every row is ready. |
| Box row: rows to fix | **{n} rows need fixing** / {count} {error name} · … (§5.7 names, lower case) [Download {n} rows]. When the file holds more: "The download also lists the {m} over your plan's limit." or "…the {m} not added to the step." |
| Box row: over the limit (amber: the account, not the file) | **{over} new prospects are over your plan's limit** / Your plan has room for {room} more; rows already in your list don't count. Upgrade to add all {n}, or import now and bring the other {over} back later. [Upgrade plan] |
| Box row: plan full (amber) | Headline in grey. **Your plan has no room for new prospects** / Upgrade, or delete prospects you no longer need, then import again. [Upgrade plan]. Import disabled unless rows can be updated |
| Every row fails | Headline "0" in red, "of {rows} rows can be imported". Box row **{n} rows need fixing** with reasons; when one reason covers over half: "{n} of {rows} have the same problem, which often means the wrong column is mapped to Email." [Check the mapping] |
| Credits short | You need up to {n} credits and have {have}. Add credits, or untick this to import now and verify later. [Add credits]. Import disabled |
| Another import running | **Another import is still running: {file}, {pct}% done.** Only one import runs at a time. You can import this file when it finishes; your file and mapping stay here. [Check again]. Import disabled |
| Import failed | **The import didn't start. The connection dropped.** Nothing was imported. Your file and mapping are kept here. Check your connection and click Import again. Button becomes "Import again" |
| Limit reached meanwhile | **Your plan's prospect limit was reached while you were importing.** Nothing was imported. Upgrade, or delete prospects you no longer need, then click Import again. |
| Verification | "Addresses already verified in the last 15 days are skipped." The 15 days is a fixed default: no link to change it. The credit balance always shows |
| Buttons | "Import {n} new prospects"; "Update {n} prospects" when none are new; "Importing…" while sending |

### 5.5 After Import

| Surface | When | Copy |
|---|---|---|
| Progress card | Running, every page: production's card, unchanged ([async-progress-bar.tsx](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/shared/components/async-progress-bar/async-progress-bar.tsx), [header.tsx L1260](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/home/components/header/header.tsx#L1260)) | Importing {3.5k} Prospects · View List. **Closes when the import finishes.** Today it then says "Imported {total} Prospects", which counts failed rows as imported ([header-helper.tsx L247](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/home/components/header/header-helper.tsx#L247)); the toast and strip carry the outcome instead |
| Toast | Finished | {added} prospects added and {existing} {past outcome} from {file}. Past outcomes: "had empty fields filled", "were updated", "had filled-in fields updated", "were left unchanged" |
| Strip, rows to fix | Finished with rows to fix | **{n} rows from {file} weren't imported because they need fixing.** Download them, fix them in that file, and upload it here. Your mapping is remembered. [Download {n} rows] [Upload fixed rows] |
| Strip, over limit only | | **{n} rows from {file} weren't imported because they are over your plan's prospect limit.** Nothing is wrong with them. Upgrade your plan or delete prospects you no longer need, then upload the downloaded file here. |
| Strip, both | | **{n} rows from {file} weren't imported:** {fix} need fixing and {over} are over your plan's prospect limit. Download them, fix the rows that need it, make room on your plan, then upload the file here. |
| Bell | Every import | {file}: {added} added, {existing} {past outcome}. Second line as the strip's bold sentence, with a Download link. Unread state has an accessible name, not colour alone |

All counts after import come from the import. Rows whose batch failed are counted in "weren't imported" and appear in the file as "Didn't finish". The strip shows on the list imported into, until dismissed or the fixed file is uploaded.

### 5.6 Completion email

Subject "Import finished: {file}". Body "{added} added, {existing} {past outcome}, {n} not imported: {top 3 error names with counts}. Download the rows to fix them." Replaces "Congratulations! Your Prospect Import is completed." for every outcome ([prospect-import.hbs L11](https://github.com/saleshandy/saleshandy-background-jobs/blob/c2c60a6ee24b9910b7aa6e67b54876bca132785c/src/app-mailer/email-templates/prospect-import.hbs#L11)). A client user's import names them and the sequence.

### 5.7 The rows-to-fix file

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
| Expiry | An expired link says "This file has expired. Files are kept for {n} days. Import your file again to get a new one." Never raw JSON. {n} is how long the report file is kept |

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

### 5.8 Import via CSV into a sequence

Same dialog, Upload and Map Fields as the list import ([sequence path today](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/shared/add-to-step-select/add-to-step-select.tsx#L13)). Differences (Hitesh, 6 Nov):

| Where | Requirement |
|---|---|
| Entry | Sequence with no steps: blocked before the dialog opens, "Add at least one step to this sequence before adding prospects." [Go to Steps] |
| Map Fields | No step picker (production has one here; it moves to Review) |
| Review, first row | "Import into [Step 1 · Email ▾]", in Review's settings layout. Options name the step type ("Step 2 · LinkedIn connection request"). Default: the lowest step number, as production ([map-fields.tsx L92-L99](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/map-fields/map-fields.tsx#L92-L99)). Changing it updates every line below at once |
| Headline | "{n} prospects will be added to Step {k}". Counts new plus existing, because production adds existing prospects to the step under every After Import option |
| Lines | "{n} of them are already in your list; they {outcome}." [Change] · "{n} already in your list won't be added: {a} are in another sequence, {b} are in this one, {c} bounced or unsubscribed." [Admin setting] · repeated, as the list import |
| Box row: step needs (amber) | Only when the field is mapped but some rows are empty: **{n} have no phone number** / Their call task (Step {k}) will have nothing to dial. They still import, and the download lists them. Same for LinkedIn ("LinkedIn profile URL", "LinkedIn task", "nothing to open"). Covers every step from the chosen one on |
| Map Fields: step needs (amber, Hitesh 6 Nov) | When no column is mapped to a field a step needs, above After Import: **No column is mapped to {fields}.** {sequence} has {a call step (Step 3) and a LinkedIn step (Step 2)}, so {its call tasks / those tasks} will have nothing to {dial / open}. If your file has {phone numbers / them}, set {that column / those columns} above. You can still import without {it / them}. Covers every step in the sequence; Next stays enabled |
| Button | "Add {n} to Step {k}" |
| Download | Includes the rows not added (errors 13 to 17) and, when only some rows lack it, the step-needs warnings "No phone for call step", "No LinkedIn URL for LinkedIn step" |
| After | Toast "{n} added to Step {k} of {sequence} from {file}." Strip on the sequence's Prospects tab, adding "The download also lists the {n} saved but not added to Step {k}." Bell "{file}: {n} added to Step {k} of {sequence}, {m} not added". Email subject "Import finished: {file} into {sequence}" |

The admin setting is "Allow adding duplicate prospects in multiple sequences" (Settings › Admin Settings › Prospects). It only affects prospects already in the list.

### 5.9 Bringing a file back

When a file's headers match a mapping that was used for a check, every column comes back decided. Then Map Fields has nothing to ask, so Upload skips it (Hitesh, 6 Nov):

- Under the file: "**Mapping restored.** You used these same columns {when}, so all {n} are set." [Review mapping]
- Next becomes "Check {rows} rows" and goes straight to the check; the steps show Map Fields as done, and Back on Review opens it.
- Anything undecided, or Email not mapped: Map Fields as usual.
- Applies to the fixed file (our three columns are set to Don't import) and to any weekly import with the same columns.
