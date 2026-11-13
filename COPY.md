# Copy spec: Prospect CSV import errors

Companion to [PRD.md](PRD.md). **See every string rendered on its surface: [copy.html](copy.html)** (built from the prototype's styles). Every string here ships as written; `{…}` are values. Section numbers match the PRD's §5. Code links: webui `f9425f2`, edge `23c74c8`, background-jobs `7d18274`.

### 5.1 Upload file

A rejected file: a red alert above the drop zone. Alert icon, the **title** (red), the file name under it, one sentence on the fix (neutral text). Below it the empty drop zone: "Drop the fixed file here, or choose a file". The zone stays live, so the fixed file goes straight back in. Next stays disabled. No toast and no "Oops!" anywhere in this flow. See it: [copy.html §5.1](copy.html#s5-1).

**Only what the check knows.** Each message is limited to facts its check already has ([pre-select.tsx L41-L86](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/select-file/components/pre-select/pre-select.tsx#L41-L86), [select-file.tsx L49-L95](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/select-file/select-file.tsx#L49-L95)): the file name, the extension, and whether a limit was crossed. No row totals (parsing stops at the limit), no header names or column positions (the duplicate and blank checks return yes or no).

| State | Check knows | Title | Fix |
|---|---|---|---|
| Accepted | Rows and columns parsed | {file}, then "First 3 of {rows} rows · {columns} columns"; 3 rows or fewer: "{rows} rows · {columns} columns" ("1 row"). No caption under the preview; a wider table just scrolls sideways | |
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

- Email row marked with a red *. Row look as production: neutral selects, no row tint. Header "Fields from file", "First 3 rows", "Field" (left edge of the selects).
- Line above the rows: "{m} of {n} columns matched automatically." When all matched: "All {n} columns matched automatically." Grey, one line; unmatched columns are not highlighted.
- No column mapped to Email: "Map a column to Email to continue." Red, one line. Next disabled. No email-like column at all: "This file has no email column. Add one, or choose another file." Red, one line, Next disabled.
- **After Import** below the fields: production's four labels and descriptions verbatim ([conflict-action.ts](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/map-fields/conflict-action.ts)), description shown under the select, default Add Missing Fields.
- An undecided column shows "Select field"; one set aside shows "Don't import".
- One column per field, as production: a field another column uses isn't offered in the list.
- Cancel, Esc or the close button keeps the mapping while the dialog's file is kept.
- Field picker: search keeps focus while typing; arrow keys move through options; Enter on the search picks the first match; Esc closes the list.

### 5.3 Checking

"Checking your file", progress as "{done} of {rows} rows", "Nothing is imported yet." No footer; one text link, "Cancel", centred under the progress.

### 5.4 Review and Import

One headline, one quiet grey line, then one line per problem, then the options under a rule. The headline is a sentence styled as a heading (20px, semibold). Every other number sits in its sentence at body size and regular weight. A problem line is a small coloured dot, a plain sentence and one link; only rows that need the user get one. Red: rows to fix, or an import that failed. Amber: the plan limit, credits, another import running. No icons, no boxes, no bar. Lines about prospects say "prospects"; lines about repeats and rows to fix say "rows". See it rendered: [copy.html §5.4](copy.html#s5-4).

| Element | Copy |
|---|---|
| Headline | {n} new prospects will be added. None new: "No new prospects will be added". Plan full: "No new prospects can be added". Every row fails: "None of the {rows} rows can be imported" |
| Quiet line (grey) | {rows} rows checked. {n} prospects already in your list {outcome}, {m} repeated rows skipped. A clause with a count of 0 is dropped |
| Outcomes, by After Import | Add Missing Fields "will get their empty fields filled"; Update All Fields "will be updated (a blank cell clears the existing value)"; Overwrite "will have their filled-in fields updated"; Skip Existing Prospects "will be left unchanged" |
| Repeats | Repeats are rows with the same email (case and spaces ignored); the first is imported. If that email is invalid, every copy is a row to fix |
| Clean file | Headline and "{rows} rows checked. Every row is ready." No problem lines |
| Red line: rows to fix | {n} rows need fixing [Download them]. When the file holds more, grey after the link: "plus the {m} over the limit" or "plus the {m} not added to the step". The per-error breakdown is not on Review; it stays in the toast, bell and email |
| Amber line: over the limit | {over} new prospects are over your plan's limit [Upgrade plan]. With no rows to fix, the line also carries [Download them] |
| Amber line: plan full | Headline "No new prospects can be added". Your plan has no room for {over} new prospects [Upgrade plan]. Verify and tags hidden. Button "Update {n} prospects"; disabled when nothing can be updated |
| Every row fails | Headline "None of the {rows} rows can be imported". When one reason covers over half, quiet line: "{n} of {rows} have the same problem, which often means the wrong column is mapped to Email." [Check the mapping]. Red line "{n} rows need fixing", no link: Download {n} rows is the primary button, "Upload fixed file" on the left |
| Verify emails | ☑ Verify emails · up to {n} of your {credits} credits. Unticked: "· no credits will be used". Addresses verified in the last 15 days are skipped (fixed default, not shown) |
| Verify, credits short (amber) | ☑ Verify emails · needs up to {n} credits, you have {have} [Add credits]. Grey line under it: "Or untick it to import now and verify later." Import disabled |
| Tags | [+ Add tags] link; a click shows the tags input, "Add tags to categorize your prospects…" |
| Another import running (amber line) | {file} is still importing ({pct}% done). This file can import when it finishes. [Check again]. Import disabled |
| Import failed (red line) | The connection dropped, so nothing was imported. Click Import again when you're back online. Button becomes "Import again" |
| Limit reached meanwhile (amber line) | Your plan's limit was reached while importing, so nothing was imported. [Upgrade plan]. Button becomes "Import again" |
| Buttons | "Import {n} new prospects"; "Update {n} prospects" when none are new; "Importing…" while sending |

### 5.5 After Import

| Surface | When | Copy |
|---|---|---|
| Progress card | Running, every page: production's card, unchanged ([async-progress-bar.tsx](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/shared/components/async-progress-bar/async-progress-bar.tsx), [header.tsx L1260](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/home/components/header/header.tsx#L1260)) | Importing {3.5k} Prospects · View List. **Closes when the import finishes.** Today it then says "Imported {total} Prospects", which counts failed rows as imported ([header-helper.tsx L247](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/home/components/header/header-helper.tsx#L247)); the toast carries the outcome instead |
| Toast, nothing left | Finished, every row imported | **{added} prospects added** and {existing} {past outcome} from {file}. Green. Past outcomes: "had empty fields filled", "were updated", "had filled-in fields updated", "were left unchanged" |
| Toast, rows left | Finished with rows to fix or over the limit | Same first sentence, then one of: "{n} rows weren't imported because they need fixing." / "{n} rows weren't imported because they are over your plan's prospect limit." / "{n} rows weren't imported: {fix} need fixing, {over} are over your plan's prospect limit." [Download {n} rows]. Amber, never green |
| List line | Finished, on the list it went into | **List updated.** {n} new prospects from {file} are at the top. [Show only these]. Sequence: "**List updated.** {n} prospects added to Step {k} from {file} are at the top." Grey, between the tabs and the table; stays while the user is on the list. The list refreshes itself; [Show only these] applies the imported-file filter, then reads [Show all prospects] |
| Bell | Every import | {file}: {added} added, {existing} {past outcome}. Second line as the toast's "weren't imported" sentence, with a Download link. Unread state has an accessible name, not colour alone |

All counts after import come from the import. Rows whose batch failed are counted in "weren't imported" and appear in the file as "Didn't finish". The toast, the bell and the email carry the outcome and the download.

### 5.6 Completion email

Sender "{Brand} <no-reply@{brand domain}>", the account's white-label brand. Subject "Import finished: {file}". Body "{added} added, {existing} {past outcome}, {n} not imported: {top 3 error names with counts}. Download the rows to fix them." Replaces "Congratulations! Your Prospect Import is completed." for every outcome ([prospect-import.hbs L11](https://github.com/saleshandy/saleshandy-background-jobs/blob/c2c60a6ee24b9910b7aa6e67b54876bca132785c/src/app-mailer/email-templates/prospect-import.hbs#L11)). A client user's import names them and the sequence.

### 5.7 The rows-to-fix file

| Rule | Requirement |
|---|---|
| Columns | `row_number`, `error`, `how_to_fix`, then every column of the uploaded file, original order and headers |
| `row_number` | The row in the user's original file as a spreadsheet shows it: header is row 1 |
| Lines | One per row not fully imported, blank-email rows included. Only rows the user can act on. Imported, updated, repeated and skipped rows, people already in this sequence and unsubscribed people are not in it |
| Over-limit rows | The last new prospects in file order. Never rows already in the list |
| Several problems | `error` and `how_to_fix` cover the first problem found |
| Second round | When the uploaded file already has `row_number`, `error`, `how_to_fix`, the new file **replaces** those three columns (never adds a second set) and keeps the original `row_number`. Name stays `{name}-rows-to-fix.csv`, never doubled |
| Quoting | A cell with a comma, double quote, semicolon, tab or line break is quoted; inner quotes doubled. Without the semicolon rule, LibreOffice and European-locale Excel split `a@x.com; b@y.com` and shift the row |
| Encoding, name | UTF-8 with BOM. `{uploaded name}-rows-to-fix.csv` |
| Counts | Lines equal the "weren't imported" count in the toast, bell and email |

Error names are fixed text, identical on Review, in the file, toast, bell and email. `{col}` is the user's header; `{value}` the cell.

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
| 13 | In another sequence | Admin setting | Already in "{sequence}". Your account allows a prospect in one sequence at a time, so it wasn't added. An admin can change this in Settings. {saved} | Prospect already present in another sequence |
| 14 | Active in another sequence | Admin setting | Active in "{sequence}". Your account doesn't allow a prospect to be active in two sequences, so it wasn't added. Add it after that sequence ends for them, or ask an admin to change the setting. {saved} | Prospect is active in another sequence |
| 15 | Email bounced before | Sequence import | Emails to this address bounced before, so it wasn't added. Check the address in {col}; if it's wrong, fix it and upload the row again. | Bounced prospect |

`{saved}`: "Its details were updated from your file.", or under Skip Existing Prospects, "Its details were left as they were." Assignment sites in [contacts-import.service.ts](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts): repeat L2534, email L2543, field type L2551, length L2563, owner L1115, limit L1834, Skip Existing L2747, other sequence L2974, active L2924, bounced L4510, batch crash L6470. An invalid phone (imported, phone left blank) and a Do Not Contact match (enrolled, not emailed) import as today and are not in the file. **Delete, never assigned:** `NoUpdate`, `NoOverwrite`, `ProspectInCallDNC`, `ContactInDncList`. `MultipleSeparator` is reachable only from a sync route the web app doesn't call.

### 5.8 Import via CSV into a sequence

Same dialog, Upload and Map Fields as the list import ([sequence path today](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/shared/add-to-step-select/add-to-step-select.tsx#L13)). Differences:

| Where | Requirement |
|---|---|
| Entry | Sequence with no steps: blocked before the dialog opens, "Add at least one step to this sequence before adding prospects." [Go to Steps] |
| Map Fields | No step picker (production has one here; it moves to Review) |
| Review, above the headline | "Import into [Step 1 · Email ▾]", in Review's settings layout. Options name the step type ("Step 2 · LinkedIn connection request"). Default: the lowest step number, as production ([map-fields.tsx L92-L99](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/map-fields/map-fields.tsx#L92-L99)). Changing it updates every line below at once |
| Headline | "{n} prospects will be added to Step {k}". Counts new plus existing, because production adds existing prospects to the step under every After Import option |
| Quiet lines (grey) | "{rows} rows checked. {n} of the {added} are already in your list and {outcome}, {m} repeated rows skipped." Then "{n} prospects already in your list won't be added: {a} are in another sequence, {b} are in this one, {c} bounced, {d} unsubscribed." [Admin setting] |
| Quiet line: step needs (grey) | Only when the field is mapped but some prospects have it empty: "{n} prospects have no phone number. Their call task (Step {k}) will have nothing to dial. They still import." Same for LinkedIn ("LinkedIn profile URL", "LinkedIn task", "nothing to open"). Covers every step from the chosen one on |
| Red line: rows to fix | {n} rows need fixing [Download them] plus the {m} not added to the step |
| Map Fields: step needs (amber) | When no column is mapped to a field a step needs, above After Import, one line: "No column is mapped to {fields}, which {Step k / Steps j and k} {needs / need}." e.g. "No column is mapped to LinkedIn Profile or Phone Number, which Steps 2 and 3 need." Covers every step in the sequence; Next stays enabled |
| Button | "Add {n} to Step {k}" |
| Download | Includes the rows not added that the user can act on (errors 13 to 15). Already in this sequence, unsubscribed, and rows a step can't work are counts on Review only |
| After | Toast "{n} added to Step {k} of {sequence} from {file}." With rows left, amber as the list import, adding "The download also lists the {n} saved but not added to Step {k}." Bell "{file}: {n} added to Step {k} of {sequence}, {m} not added". Email subject "Import finished: {file} into {sequence}" |

The admin setting is "Allow adding duplicate prospects in multiple sequences" (Settings › Admin Settings › Prospects). It only affects prospects already in the list.

### 5.9 Bringing a file back

A file with our three columns (`row_number`, `error`, `how_to_fix`) is a rows-to-fix file. Map Fields opens with the mapping of the import it came from:

- "Mapping restored from your import of {file}." [Clear and map again]. Grey, one line. Our three columns (`row_number`, `error`, `how_to_fix`) are set to Don't import.
- Map Fields is not skipped. Any other file maps as production does.
