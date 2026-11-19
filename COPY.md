# Copy spec: Prospect CSV import errors

Companion to [PRD.md](PRD.md). **See every string rendered on its surface: [copy.html](copy.html)** (built from the prototype's styles). Every string here ships as written; `{…}` are values. Section numbers match the PRD's §5. Code links: webui `f9425f2`, edge `23c74c8`, background-jobs `7d18274`.

### 5.1 Upload file

A rejected file: the error shows inside the drop zone, centred. Error icon (red, in a soft red circle), the file name in grey, the **title**, one sentence on the fix, then a secondary button "Try again" (opens the file picker). The zone still takes a dropped file, so the fixed file goes straight back in. Next stays disabled. No toast and no "Oops!" anywhere in this flow. See it: [copy.html §5.1](copy.html#s5-1).

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
| Repeated column names | Two or more headers match (yes or no) | Some columns have the same name | Rename or delete the repeated columns. |
| Columns with no name | Two or more blank headers (yes or no) | Some columns have no name | Every column needs a name in the first row. Name the empty ones, or delete those columns. |
| Mixed separators | Header uses more than one of `, ; | tab` (moves from the server, [edge L1380](https://github.com/saleshandy/saleshandy-edge/blob/23c74c892b99392f27f5fb81bd5e3d610d1f4c11/src/common/response/messages/api-errors.ts#L1380-L1382)) | The header row mixes separators | Use one separator throughout, or put quotes around headers that contain a comma. |
| Not UTF-8 | Unreadable bytes (existing `is-utf8-file` util) | This file isn't saved as UTF-8 | In Excel, choose **File › Save As** and pick **CSV UTF-8 (Comma delimited)**. |
| Unreadable | Browser read error | We couldn't read this file | If it's open in Excel or still syncing, close it or wait, then try again. |

Every check that today runs only after Import (duplicate headers, mixed separators, leading empty line, upper-case `.CSV`) runs here, so Import never fails for a reason the file already showed.

### 5.2 Map Fields

- Email row marked with a red *. Row look as production: neutral selects, no row tint. Header "Fields from file", "Sample", "Field" (left edge of the selects).
- No line about matched columns. A column with no field yet: its "Select field" select is amber; the row stays plain.
- No column mapped to Email: "Map a column to Email to continue." Red, one line. Next disabled. No email-like column at all: "This file has no email column. Add one, or choose another file." Red, one line, Next disabled.
- **After Import** in the table footer, as production, with "Import into [Step 1]" before it on a sequence import; (i) "Please select the action to perform on your prospect details after importing"; each option shows its text on hover: production's four labels and descriptions verbatim ([conflict-action.ts](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/map-fields/conflict-action.ts)), default Add Missing Fields.
- An undecided column shows "Select field"; one set aside shows "Don't import".
- Header band, sentence case: "Fields from file", "Sample", "Field". Cross button on a mapped field: aria-label "Remove field from {column}".
- Field list footer: "+ Create custom field", which opens production's modal: title "Create Custom Field"; "Field Name" ("Enter Field Name", prefilled with the column name); "Field Type" (Text, Number, Long Text, Date, Dropdown, Currency); "Fallback Text (Optional)" ("Enter Fallback Text") for Text, Number, Long Text; "+ Add Option" for Dropdown; "Currency" for Currency; "Tags" ("Enter a tag"); Cancel, Save. No matches: "No field matches “{search}”."
- One column per field, as production: a field another column uses isn't offered in the list.
- Cancel, Esc or the close button keeps the mapping while the dialog's file is kept.
- Field picker: search keeps focus while typing; arrow keys move through options; Enter on the search picks the first match; Esc closes the list.

### 5.3 Checking

Inside the summary card the result lands in: badge spinner, "Reviewing rows", "{done} of {rows} rows", a spinner where each count will appear. Footer: Cancel, and Import disabled.

### 5.4 Review and Import

One summary card, then a line per problem that needs the user, then Tags and Verification on the card's inner edge. Card header (light band): badge, headline (15px, semibold), "{rows} rows checked". Rows: label left (13.5px), count right (semibold, tabular), short grey note beside a label. Two groups split by one line: what goes in, what does not. A row with a count of 0 is dropped.

| Element | Copy |
|---|---|
| Headline | {n} new prospects will be added. None new: "No new prospects will be added". Plan full: "No new prospects can be added". Every row fails: "None of the {rows} rows can be imported" |
| Card rows | "New prospects" {n}; "Already in your list" {e} with the After Import note; "Over your plan's limit" {o}; "Repeated rows" {r} with "first one kept"; "Rows to fix" {f} with [Download CSV] beside the label |
| Note on "Already in your list", by After Import | Add Missing Fields "empty fields filled"; Update All Fields "every field updated"; Overwrite "filled-in fields updated"; Skip Existing Prospects "left unchanged" |
| Repeats | Repeats are rows with the same email (case and spaces ignored); the first is imported. If that email is invalid, every copy is a row to fix |
| Clean file | Card with one row, "New prospects" {n}. No problem lines |
| Rows to fix | The card's row: "Rows to fix" {n} [Download CSV]. Not repeated as a red line. Rows over the plan limit are their own row. The per-error breakdown is not on Review; it stays in the file |
| Amber line: over the limit | {over} new prospects are over your plan's limit [Upgrade plan]. With no rows to fix, the line also carries [Download CSV] |
| Amber line: plan full | Amber badge; headline "No new prospects can be added". "You've reached your plan's prospect limit." [Upgrade plan]. Tags and Verification hidden. Button "Update {n} prospects"; disabled when nothing can be updated |
| Every row fails | Headline "None of the {rows} rows can be imported". When one reason covers over half, quiet line: "{n} of {rows} have the same problem, which often means the wrong column is mapped to Email." [Check the mapping]. Red line "{n} rows to fix", no link: "Download CSV" is the primary button, "Upload fixed file" on the left |
| Verification | Label "Verification". ☑ "Verify emails after import", ticked by default. Under it: "Addresses already verified in the last 15 days are skipped." [Change the 15 days]. Credits line: "Up to {n} credits. You have {have}." Unticked: "No credits will be used. You can verify these from the prospect list later." |
| Verification, credits short (amber) | "You need up to {n} credits and have {have}. Add credits, or untick this to import now and verify later." [Add credits] button. Import disabled |
| Tags | Label "Tags"; the box is always there: "Add tags to categorize your prospects…". Enter or a comma adds a tag as a chip ("Add another" after the first), Backspace removes the last, a repeat is ignored |
| Another import running (amber line) | Another import is still running. You can import when it finishes. [Check again]. Import disabled |
| Import failed (red line) | The connection dropped, so nothing was imported. Click Import again when you're back online. Button becomes "Import again" |
| Limit reached meanwhile (amber line) | Your plan's limit was reached while importing, so nothing was imported. [Upgrade plan]. Button becomes "Import again" |
| Buttons | "Import {n} new prospects"; "Update {n} prospects" when none are new; "Importing…" while sending |

### 5.5 After Import

| Surface | When | Copy |
|---|---|---|
| Progress card | Running, every page: production's card, unchanged ([async-progress-bar.tsx](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/shared/components/async-progress-bar/async-progress-bar.tsx), [header.tsx L1260](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/home/components/header/header.tsx#L1260)) | Importing {3.5k} Prospects · View List. **Closes when the import finishes.** Today it then says "Imported {total} Prospects", which counts failed rows as imported ([header-helper.tsx L247](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/home/components/header/header-helper.tsx#L247)); the toast carries the outcome instead |
| Toast, nothing left | Finished, every row imported | **{imported} prospects imported.** Green; closes on its own |
| Toast, rows left | Finished with rows to fix or over the limit | **{imported} prospects imported.** {n} rows to fix. [Download CSV]. Over the limit only: **{imported} prospects imported.** {m} over your plan's limit. [Upgrade plan] · [Download CSV]. Both: "{n} rows to fix, {m} over your plan's limit." {imported} = new plus updated prospects; {n} = rows in the CSV the user can fix; {m} = new prospects over the plan's limit. Amber, never green. Stays until the user closes it (icon button "Dismiss"). The reasons are in the CSV. Sequence imports keep their own toast (§5.8) |
| List line | Finished, on the list it went into | **List updated.** {n} new prospects from {file} are at the top. [Show only these]. Sequence: "**List updated.** {n} prospects added to Step {k} from {file} are at the top." Grey, between the tabs and the table; stays while the user is on the list. The list refreshes itself; [Show only these] applies the imported-file filter, then reads [Show all prospects] |
| Toast, after a download | Download clicked | CSV downloaded. Green, short |
| Bell | Every import | As production today: title "Prospect import is completed"; production's sentence "Your file {file} is imported having {total} prospects: {imported} Imported and {failed} Failed" (file name cut at 15 characters, values bold); "Download Failed Report" when {failed} is not 0; "Just now"; "View List" on the right; panel header "Notifications" and "Mark all as Read" ([notification-log-templates.ts L4-L5](https://github.com/saleshandy/saleshandy-background-jobs/blob/c2c60a6ee24b9910b7aa6e67b54876bca132785c/src/notification/constants/notification-log-templates.ts#L4-L5), [notification.tsx L542-L559](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/notification/notification.tsx#L542-L559)). {imported} = new plus updated prospects, the toast's number; {failed} = rows in the CSV (rows to fix plus rows over the plan's limit). The two don't have to add up to {total}: repeated rows and rows not added that need nothing are counted on Review only. No second line. Unread state has an accessible name, not colour alone |

All counts after import come from the import. Rows whose batch failed are counted in "rows to fix" and appear in the file as "Didn't finish". The toast, the bell and the email carry the outcome and the download.

### 5.6 Completion email

Sender "{Brand} <no-reply@{brand domain}>", the account's white-label brand. Subject "Import finished: {file}". Body, one count per line, a line with a count of 0 dropped except the first, with the toast's numbers: "{imported} prospects imported" / "{n} rows to fix" / "{m} over your plan's limit"; then "Download the CSV to fix them." and the button "Download CSV". No per-error breakdown; the reasons are in the CSV. When every row failed: "0 prospects imported" / "{n} rows to fix" Replaces "Congratulations! Your Prospect Import is completed." for every outcome ([prospect-import.hbs L11](https://github.com/saleshandy/saleshandy-background-jobs/blob/c2c60a6ee24b9910b7aa6e67b54876bca132785c/src/app-mailer/email-templates/prospect-import.hbs#L11)). A client user's import names them and the sequence.

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
| Counts | Lines equal "rows to fix" plus "over your plan's limit" in the toast and the email, and Failed in the bell |

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
| 11 | Over plan limit | New prospect past the plan limit | Nothing is wrong with this row. Your plan's prospect limit was reached before it. Upgrade your plan or delete prospects you no longer need, then upload it again. | Import failed due to prospect limit reached |
| 12 | Didn't finish | Its batch crashed | Nothing is wrong with this row. It didn't import because of a problem on our side. Upload it again. | **Rows vanish** |
| 13 | In another sequence | Admin setting | Already in "{sequence}". Your account allows a prospect in one sequence at a time, so it wasn't added. An admin can change this in Settings. {saved} | Prospect already present in another sequence |
| 14 | Active in another sequence | Admin setting | Active in "{sequence}". Your account doesn't allow a prospect to be active in two sequences, so it wasn't added. Add it after that sequence ends for them, or ask an admin to change the setting. {saved} | Prospect is active in another sequence |
| 15 | Email bounced before | Sequence import | Emails to this address bounced before, so it wasn't added. Check the address in {col}; if it's wrong, fix it and upload the row again. | Bounced prospect |

`{saved}`: "Its details were updated from your file.", or under Skip Existing Prospects, "Its details were left as they were." Assignment sites in [contacts-import.service.ts](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts): repeat L2534, email L2543, field type L2551, length L2563, owner L1115, limit L1834, Skip Existing L2747, other sequence L2974, active L2924, bounced L4510, batch crash L6470. An invalid phone (imported, phone left blank) and a Do Not Contact match (enrolled, not emailed) import as today and are not in the file. **Delete, never assigned:** `NoUpdate`, `NoOverwrite`, `ProspectInCallDNC`, `ContactInDncList`. `MultipleSeparator` is reachable only from a sync route the web app doesn't call.

### 5.8 Import via CSV into a sequence

Same dialog, Upload and Map Fields as the list import ([sequence path today](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/shared/add-to-step-select/add-to-step-select.tsx#L13)). Differences:

| Where | Requirement |
|---|---|
| Map Fields | Production's table footer, unchanged: "Import into [Step 1 ▾]" beside "After Import (i) [Add Missing Fields ▾]". Default: the lowest step |
| Headline | "{n} prospects will be added to Step {k}". Counts new plus existing, because production adds existing prospects to the step under every After Import option |
| Card rows | "Added to Step {k}" {n} with the note "{e} already in your list · {d} on Do Not Contact, won't be emailed" ({d} is inside {n}); "Repeated rows" {r} "first one kept"; "Won't be added" {w} "in this sequence or unsubscribed"; "Rows to fix" {f} [Download CSV], {f} = rows in the download |
| Step needs (grey, under the card) | Only when the field is mapped but some prospects have it empty: "{n} prospects have no phone number for the Step {k} call." Same for LinkedIn: "{n} prospects have no LinkedIn URL for the Step {k} task." They still import |
| Map Fields: step needs (amber) | When no column is mapped to a field a step needs, one line in the message slot at the top of Map Fields (replaces the grey match line; a red Email error wins over it): "No column is mapped to {fields}, which {Step k / Steps j and k} {needs / need}." e.g. "No column is mapped to LinkedIn Profile or Phone Number, which Steps 2 and 3 need." Covers every step in the sequence; Next stays enabled |
| Button | "Add {n} to Step {k}" |
| Download | Includes the rows not added that the user can act on (errors 13 to 15). Already in this sequence, unsubscribed, and rows a step can't work are counts on Review only |
| After | Toast "**{n} added to Step {k}.** {m} rows to fix. [Download CSV]", amber, with {m} the rows in the download; green "{n} added to Step {k}." when nothing is left. Bell: production's sentence with "is imported in {sequence} having" (§5.5). Email subject "Import finished: {file} into {sequence}"; body "{n} prospects added to Step {k} of {sequence}" / "{m} rows to fix", then "Download the CSV to fix them." and [Download CSV] |

The admin setting is "Allow adding duplicate prospects in multiple sequences" (Settings › Admin Settings › Prospects). It only affects prospects already in the list.

### 5.9 Bringing a file back

A file with our three columns (`row_number`, `error`, `how_to_fix`) is a rows-to-fix file. Map Fields opens with the mapping of the import it came from:

- "Mapping restored from your import of {file}." [Clear and map again]. Grey, one line. Our three columns (`row_number`, `error`, `how_to_fix`) are set to Don't import.
- Map Fields is not skipped. Any other file maps as production does.
