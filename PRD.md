# PRD: Prospect CSV import errors users can understand and fix

**Design of record:** [prototype](https://product-team-sh.github.io/prd-prospect-csv-import/prototype/), every state captured on [the canvas](https://product-team-sh.github.io/prd-prospect-csv-import/canvas/) · **Copy:** [COPY.md](COPY.md), rendered in [copy.html](https://product-team-sh.github.io/prd-prospect-csv-import/copy.html) · **Status:** Draft, design sign-off pending

---

## 1. Problem

- **Most prospects arrive this way.** 71% of new prospects come from CSV import.
- **Nothing is checked before import, and the result is unreadable.** Every outcome ends in a green toast. The report is `email,failed_reason`: one line per email, no row number, none of the user's columns. Six different email problems all read "Invalid Email".
- **Some failures leave no trace.** Blank-email rows and crashed batches produce no report line. No failure is ever recorded, because the only event fires on success.

## 2. Why this approach

| Option | Verdict |
|---|---|
| Better strings in today's report | Rejected: still no row numbers or user columns, still nothing before commit |
| A grid of failed rows in the dialog | Rejected: can't fit a 720px dialog at 40,000 rows; reasons fit in five lines at any size |
| **Check before commit, reasons on screen, the user's own rows back with a fix per row** | **Chosen** |

## 3. What we're building

**Goals**

1. Before Import, show what will be added, updated, skipped or blocked.
2. Explain every rejected file and every row problem in plain words, with one concrete fix.
3. Return every row not fully imported in a file of the user's own columns and row numbers.
4. Take the fixed file back with its mapping restored.
5. Handle plan limits, short credits, a running import and sequence rules without losing the file or mapping.
6. Instrument every step and outcome from launch.

Nothing is written until Import (or "Add {n} to Step {k}") is pressed.

## 4. User stories

| ID | As a user importing prospects, I… |
|---|---|
| U1 | see before importing how many will be added, what happens to ones already in my list, and what needs fixing and why |
| U2 | download only the rows I must act on, in my own columns, filter by error, and upload the fixed file with my mapping restored |
| U3 | am told why a file is rejected, and how to fix it, without restarting |
| U4 | stay on Review with my file and mapping if Import fails or another import is running |
| U5 | can import what fits when my plan or credits are short |
| U6 | pick the sequence step on Review and see who won't be added, and who the step can't work, before importing |
| U7 | find my mapping restored when a rows-to-fix file comes back |
| U8 | (product team) can read failure rates and reasons from day 1 |

## 5. Surfaces

Exact copy for every state below: [COPY.md](COPY.md), same section numbers; every string rendered on its surface in [copy.html](https://product-team-sh.github.io/prd-prospect-csv-import/copy.html).

**5.1 Upload.**
- **Empty:** the drop zone reads "Drag & drop your CSV with prospect details, or choose a file", where "choose a file" is a button that opens the file picker, so the zone works by keyboard; under it in grey, the limits the checks enforce: "CSV, up to 100,000 rows and 95 MB".
- **Accepted file:** one line (name; "First 3 of {rows} rows · {columns} columns"; Remove file) over its first 3 rows under the file's own headers, exactly as read. Values wider than a cell truncate, full value on hover. A table wider than the dialog scrolls sideways, with no caption. Never more than 3 rows.
- **Rejected file:** the error shows inside the upload component itself, which keeps its shape: the edge turns red, and centred in it sit an error icon in a soft red circle, the file name (small, grey, with a file icon, truncated, full name on hover), the error as the title, one sentence on the fix, and a secondary "Try again" button with an upload icon, which opens the file picker. The whole zone still accepts a drop. A new file replaces the alert with its own result, so errors never stack. Next disabled; no toast, no "Oops!".
- **Eleven checks, all on Upload:** Excel, other type or no extension, over 95 MB, over 100,000 rows, over 120 columns, no prospects, repeated column names, columns with no name, mixed separators, not UTF-8, unreadable. The ones that today fail only after Import (mixed separators, leading empty line, upper-case `.CSV`) move here; the server keeps its checks as a backstop.
- **Only what the check knows:** no row totals when parsing stopped at a limit, no header names or column positions from a yes-or-no check.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/u01.png?v=93d62c3b"><img src="canvas/shots/u01.png?v=93d62c3b" width="260"></a><br><sub><b>Upload</b>. Drop zone; Try sample file in the footer</sub></td><td valign="top" width="33%"><a href="canvas/shots/u02.png?v=a4728049"><img src="canvas/shots/u02.png?v=a4728049" width="260"></a><br><sub><b>File accepted</b>. One file line, then its first 3 rows as read; Next enabled</sub></td><td valign="top" width="33%"><a href="canvas/shots/u03.png?v=62f91563"><img src="canvas/shots/u03.png?v=62f91563" width="260"></a><br><sub><b>Excel file</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/u04.png?v=a87a614d"><img src="canvas/shots/u04.png?v=a87a614d" width="260"></a><br><sub><b>Wrong file type</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td><td valign="top" width="33%"><a href="canvas/shots/u05.png?v=babb3040"><img src="canvas/shots/u05.png?v=babb3040" width="260"></a><br><sub><b>Over 95 MB</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td><td valign="top" width="33%"><a href="canvas/shots/u06.png?v=763b15e5"><img src="canvas/shots/u06.png?v=763b15e5" width="260"></a><br><sub><b>Over 100,000 rows</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/u07.png?v=6f6794c9"><img src="canvas/shots/u07.png?v=6f6794c9" width="260"></a><br><sub><b>Over 120 columns</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td><td valign="top" width="33%"><a href="canvas/shots/u08.png?v=5dcb454c"><img src="canvas/shots/u08.png?v=5dcb454c" width="260"></a><br><sub><b>No prospects</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td><td valign="top" width="33%"><a href="canvas/shots/u09.png?v=0f10f164"><img src="canvas/shots/u09.png?v=0f10f164" width="260"></a><br><sub><b>Repeated column names</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/u10.png?v=ed149036"><img src="canvas/shots/u10.png?v=ed149036" width="260"></a><br><sub><b>Columns with no name</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td><td valign="top" width="33%"><a href="canvas/shots/u11.png?v=4d7826e5"><img src="canvas/shots/u11.png?v=4d7826e5" width="260"></a><br><sub><b>Mixed separators</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td><td valign="top" width="33%"><a href="canvas/shots/u12.png?v=60dcd339"><img src="canvas/shots/u12.png?v=60dcd339" width="260"></a><br><sub><b>Not UTF-8</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/u13.png?v=8e3e4356"><img src="canvas/shots/u13.png?v=8e3e4356" width="260"></a><br><sub><b>Can’t be read</b>. Error inside the drop zone: icon, file, error, fix, Try again</sub></td></tr></table>

**5.2 Map Fields.**
- **Production look:** the production mapping table: a bordered box with a grey header band ("Fields from file", "Sample", "Field", sentence case), compact rows with no row lines, neutral selects, no row tint or coloured names; the sample column shows one value per column, as production ("Sample"). The "Field" header aligns with the left edge of the selects. The Email row carries a red *.
- **Unmatched columns:** no count line. A column with no field yet shows its "Select field" select highlighted in amber; the row itself stays plain. Choosing a field or Don't import clears it. One message slot above the rows holds one line at a time: a red Email error first, then a sequence step warning (§5.8), then "Mapping restored" (§5.9).
- **Email is required.** Unmapped, or no email column in the file: say which, in one red line, keep Next disabled.
- **Remove a field:** a mapped column has a cross on the right of its select; it clears the field and the column is not imported ("Don't import"). Production shows a trash icon on the left; the cross on the right replaces it.
- **Create custom field:** the field list ends with "+ Create custom field". It opens production's Create Custom Field modal unchanged: Field Name (prefilled with the column name), Field Type (Text, Number, Long Text, Date, Dropdown, Currency), Fallback Text (Optional) for Text, Number and Long Text, options with "+ Add Option" for Dropdown, Currency for Currency, and Tags; Cancel and Save. Save creates the field and maps the column to it; an existing field with the same name is mapped instead of a duplicate.
- **One column per field,** as production: a field another column uses isn't offered. The picker searches; arrows move, Enter picks the first match, Esc closes it.
- **Sample value:** each column shows its first row's value. A long value ends in "…", the full value shows on hover.
- **Table footer, production's row:** "Import into [Step 1]" (sequence imports only) and production's After Import select, labelled "Already in your list", first option by default. Its four options keep production's behaviour (Add Missing Fields, Update All Fields, Overwrite, Skip Existing Prospects) but are named by outcome, each one's text under its name in the list, not on hover: "Fill empty fields" (Empty fields take your file's value. Filled fields stay as they are.), "Replace with your file" (Every field takes your file's value. A blank cell clears it.), "Update filled fields only" (Fields with a value take your file's value. Empty fields stay empty.), "Leave unchanged" (Nothing changes for these prospects.). The (i) is removed.
- **Mapping is kept** on Cancel, Esc or close. A rows-to-fix file gets its mapping back (§5.9).

<table><tr><td valign="top" width="33%"><a href="canvas/shots/m01.png?v=817b0934"><img src="canvas/shots/m01.png?v=817b0934" width="260"></a><br><sub><b>Columns to set</b>. Production table; unmatched selects amber; a cross removes a field</sub></td><td valign="top" width="33%"><a href="canvas/shots/m02.png?v=d9136160"><img src="canvas/shots/m02.png?v=d9136160" width="260"></a><br><sub><b>Everything matched</b>. Every select set; nothing highlighted</sub></td><td valign="top" width="33%"><a href="canvas/shots/m03.png?v=26d787fa"><img src="canvas/shots/m03.png?v=26d787fa" width="260"></a><br><sub><b>Fixed file brought back</b>. Mapping restored from the import it came from; our three columns set to Don’t import</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/m04.png?v=2d07f487"><img src="canvas/shots/m04.png?v=2d07f487" width="260"></a><br><sub><b>Field picker</b>. Search; a field another column uses isn’t offered</sub></td><td valign="top" width="33%"><a href="canvas/shots/m06.png?v=63ae04b5"><img src="canvas/shots/m06.png?v=63ae04b5" width="260"></a><br><sub><b>Email not mapped</b>. Next disabled until a column is set to Email</sub></td><td valign="top" width="33%"><a href="canvas/shots/m07.png?v=5578c11b"><img src="canvas/shots/m07.png?v=5578c11b" width="260"></a><br><sub><b>No email column</b>. Said plainly; Next disabled</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/m08.png?v=85ced9dd"><img src="canvas/shots/m08.png?v=85ced9dd" width="260"></a><br><sub><b>Already in your list: Replace with your file</b>. Production’s footer; options named by outcome, each one’s text under its name</sub></td><td valign="top" width="33%"><a href="canvas/shots/m09.png?v=9cc53046"><img src="canvas/shots/m09.png?v=9cc53046" width="260"></a><br><sub><b>Already in your list: Update filled fields only</b>. Production’s footer; options named by outcome, each one’s text under its name</sub></td><td valign="top" width="33%"><a href="canvas/shots/m10.png?v=ecd68952"><img src="canvas/shots/m10.png?v=ecd68952" width="260"></a><br><sub><b>Already in your list: Leave unchanged</b>. Production’s footer; options named by outcome, each one’s text under its name</sub></td></tr></table>

**5.3 Checking.** Every row is checked, with the existing-prospect and sequence lookups, before Review. It happens inside the summary card the result lands in (§5.4): a spinner in the badge, "Reviewing rows", "{done} of {rows} rows", and a spinner where each count will appear. The footer is Review's: Cancel, and Import disabled until the check ends. Cancel imports nothing.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/c01.png?v=24c57afa"><img src="canvas/shots/c01.png?v=24c57afa" width="260"></a><br><sub><b>Checking every row</b>. Inside the summary card: rows counted so far, a spinner where each count lands</sub></td></tr></table>

**5.4 Review.**
- **One summary card**, as production's Import New List review (`v2-lead/.../review-step/matching-summary-card.tsx`): a header on a light band with a badge (a green tick only when every row goes in and nothing needs the user; amber whenever rows are left to fix, rows are over the plan's limit or the plan is full), the headline and "{rows} rows checked", then every row of the file in two groups split by one line. First what goes in: "New prospects", "Already in your list"; then what does not: "Over your plan's limit", "Repeated rows", "Rows to fix". Label left, count right, every count in the same dark text; a row with a count of 0 is dropped. A short grey note sits beside a label instead of behind an icon. The counts add up to the rows in the file. The badge is a green tick, or an amber warning when nothing can be added.
- **The choice is asked once, on Map Fields.** Review says what happens in a grey note beside "Already in your list": Fill empty fields "empty fields filled"; Replace with your file "every field updated"; Update filled fields only "filled-in fields updated"; Leave unchanged "left unchanged". "Repeated rows" carries "first one kept".
- **Prospects and rows:** lines about rows that become or are prospects say "prospects"; lines about repeats and rows to fix say "rows". Numbers inside sentences are body size and regular weight.
- **Repeats:** repeats are rows with the same email (case and spaces ignored); the first is imported. If that email is invalid, every copy is a row to fix.
- **Problems that need the user** follow the card as one line each: a small coloured dot, a plain sentence, one link. Amber for the plan limit ("1,040 new prospects are over your plan's limit", Upgrade plan), another import, a limit reached while importing; red for a failed import. Rows to fix are not repeated as a line: the card's row carries the download.
- **Rows to fix** carry "Download CSV" beside the label in the card. Rows over the plan limit are their own row, never counted as rows to fix. The per-error breakdown is not on Review; it stays in the file.
- **Options** sit under the card, on its inner edge, as two labelled rows. **Tags:** the box is always there, "Add tags to categorize your prospects…"; Enter or a comma adds a tag as a chip, Backspace removes the last, a repeat is ignored. **Verification:** checkbox "Verify emails after import", ticked by default; under it "Addresses already verified in the last 15 days are skipped." with "Change the 15 days" (Smart Verification in settings, account owner and admins only); then "Up to {n} credits. You have {have}.", where {n} is the prospects being added and {have} the email verification credits left. Unticked: "No credits will be used. You can verify these from the prospect list later."
- **Import button** names the new prospects ("Import 3,540 new prospects"), or "Update {n} prospects" when none are new. "Importing…" while the request is sent.

**Review states:**

| State | Requirement |
|---|---|
| Plan limit | Amber line with the count over the limit; Upgrade plan. Rows already in the list don't count |
| Plan full | Amber badge; headline "No new prospects can be added"; the card shows "Over your plan's limit" with the count; amber line "You've reached your plan's prospect limit", Upgrade plan; no options; button "Update {n} prospects", disabled when nothing can be updated |
| Credits short | The credits line turns amber: "You need up to {n} credits and have {have}. Add credits, or untick this to import now and verify later." with an Add credits button. Import disabled until credits are added or verification is unticked |
| Another import running | Known on arrival at Review, not on Import. One short amber line: "Another import is still running. You can import when it finishes." Check again; Import disabled |
| Import fails | Dialog stays on Review; red line: nothing imported; file and mapping kept; button becomes "Import again" |
| Limit reached between Review and Import | As Import fails, with an amber line and Upgrade plan |
| Every row fails | Headline "None of the {rows} rows can be imported"; red line without a link; "Download CSV" is the primary button, Upload fixed file on its left; when one reason covers over half the rows, the quiet line suggests checking the Email mapping |

<table><tr><td valign="top" width="33%"><a href="canvas/shots/r01.png?v=56fd5d57"><img src="canvas/shots/r01.png?v=56fd5d57" width="260"></a><br><sub><b>Review</b>. Summary card: what goes in, what does not, Download CSV on Rows to fix; Tags and Verification under it</sub></td><td valign="top" width="33%"><a href="canvas/shots/r02.png?v=89ed7a82"><img src="canvas/shots/r02.png?v=89ed7a82" width="260"></a><br><sub><b>Clean file</b>. The card with one row; nothing needs the user</sub></td><td valign="top" width="33%"><a href="canvas/shots/r03.png?v=8b2cc876"><img src="canvas/shots/r03.png?v=8b2cc876" width="260"></a><br><sub><b>Already in your list: Replace with your file</b>. The note beside Already in your list follows the option chosen on Map Fields</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/r04.png?v=b65947de"><img src="canvas/shots/r04.png?v=b65947de" width="260"></a><br><sub><b>Already in your list: Update filled fields only</b>. The note beside Already in your list follows the option chosen on Map Fields</sub></td><td valign="top" width="33%"><a href="canvas/shots/r05.png?v=02be3635"><img src="canvas/shots/r05.png?v=02be3635" width="260"></a><br><sub><b>Already in your list: Leave unchanged</b>. The note beside Already in your list follows the option chosen on Map Fields</sub></td><td valign="top" width="33%"><a href="canvas/shots/r06.png?v=c49923f4"><img src="canvas/shots/r06.png?v=c49923f4" width="260"></a><br><sub><b>Over the plan limit</b>. Its own row in the card; amber line with Upgrade plan</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/r07.png?v=4fe0ae05"><img src="canvas/shots/r07.png?v=4fe0ae05" width="260"></a><br><sub><b>Plan full</b>. Amber badge; No new prospects can be added; Upgrade plan</sub></td><td valign="top" width="33%"><a href="canvas/shots/r08.png?v=b11f1a0d"><img src="canvas/shots/r08.png?v=b11f1a0d" width="260"></a><br><sub><b>Not enough credits</b>. The credits line turns amber with Add credits; Import disabled</sub></td><td valign="top" width="33%"><a href="canvas/shots/r09.png?v=a90e4f05"><img src="canvas/shots/r09.png?v=a90e4f05" width="260"></a><br><sub><b>Another import running</b>. Amber line on arrival, Check again; Import disabled</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/r10.png?v=87bf4adc"><img src="canvas/shots/r10.png?v=87bf4adc" width="260"></a><br><sub><b>Importing…</b>. The button while the file is sent</sub></td><td valign="top" width="33%"><a href="canvas/shots/r11.png?v=c35e77aa"><img src="canvas/shots/r11.png?v=c35e77aa" width="260"></a><br><sub><b>Import failed</b>. Red line; dialog stays; Import again</sub></td><td valign="top" width="33%"><a href="canvas/shots/r12.png?v=0d065501"><img src="canvas/shots/r12.png?v=0d065501" width="260"></a><br><sub><b>Limit reached during import</b>. Amber line, Upgrade plan; nothing imported; file and mapping kept</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/r13.png?v=5c86b55f"><img src="canvas/shots/r13.png?v=5c86b55f" width="260"></a><br><sub><b>Every row fails</b>. Headline says none; Download CSV is primary; Upload fixed file; check the mapping</sub></td><td valign="top" width="33%"><a href="canvas/shots/r14.png?v=f48c3250"><img src="canvas/shots/r14.png?v=f48c3250" width="260"></a><br><sub><b>Download toast</b>. Short green toast after Download</sub></td></tr></table>

**5.5 After Import.**
- Production's progress card while it runs, closed when finished.
- **One count model on every surface after import, in Review's numbers:** added = new prospects (a sequence: prospects added to the step); updated = prospects already in the list ("left unchanged" under "Leave unchanged"); rows to fix = rows in the CSV the user can fix; rows over the plan's limit are named apart, never called rows to fix. Toast, bell and email never show a number Review didn't, and the bell's counts add up to the file.
- One short toast: "{added} added, {updated} updated." (a count of 0 dropped). With rows left it is amber and adds "{n} rows to fix." with [Download CSV]. Over the limit: "{m} over your plan's limit." with [Upgrade plan] · [Download CSV]; both: "{n} rows to fix, {m} over your plan's limit." The reasons are in the CSV. Green only when every row imported. A toast with a link stays until the user closes it ("Dismiss"); a green toast without one may close on its own.
- The list the file went into refreshes on its own when the import finishes: no reload, new prospects first, and the screen shows the list is up to date.
- Downloading the file shows a short green toast: "CSV downloaded."
- The bell keeps production's notification (purple badge with the user icon, title "Prospect import is completed", file name cut at 15 characters and sequence at 25, values bold, the relative time, "View List" on the right, panel header "Mark all as Read"), with two changes: the sentence counts rows in Review's words so it adds up to the file, "Your file {file} is imported having {rows} rows: {added} added, {updated} updated, {n} to fix, {m} over your plan's limit and {repeated} repeated" (sequence: "is imported in {sequence} having {rows} rows: {added} added to Step {k}, {n} to fix and {x} not added"; a count of 0 dropped), and the download reads "Download CSV", with a download icon, when any row is in the CSV. Today's "{imported} Imported and {failed} Failed" never adds up to the file's total. Its unread state has an accessible name, not colour alone.
- Every count comes from the import job, not the Review estimate. Rows of a failed batch count as not imported and are in the file as "Didn't finish".

<table><tr><td valign="top" width="33%"><a href="canvas/shots/a01.png?v=70d9de79"><img src="canvas/shots/a01.png?v=70d9de79" width="260"></a><br><sub><b>Importing</b>. Production’s progress card, top right on every page</sub></td><td valign="top" width="33%"><a href="canvas/shots/a02.png?v=9652adf4"><img src="canvas/shots/a02.png?v=9652adf4" width="260"></a><br><sub><b>Finished, rows to fix</b>. The list refreshed and says so; one amber toast with the download</sub></td><td valign="top" width="33%"><a href="canvas/shots/a03.png?v=c941232f"><img src="canvas/shots/a03.png?v=c941232f" width="260"></a><br><sub><b>Finished, over the limit only</b>. Nothing wrong with the rows; the plan limit was reached</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/a04.png?v=f01bc94b"><img src="canvas/shots/a04.png?v=f01bc94b" width="260"></a><br><sub><b>Finished, both</b>. Rows to fix and rows over the limit in one toast</sub></td><td valign="top" width="33%"><a href="canvas/shots/a05.png?v=bbf51b2a"><img src="canvas/shots/a05.png?v=bbf51b2a" width="260"></a><br><sub><b>Finished, nothing left</b>. The list refreshed and says so; green toast</sub></td><td valign="top" width="33%"><a href="canvas/shots/a06.png?v=e37ed929"><img src="canvas/shots/a06.png?v=e37ed929" width="260"></a><br><sub><b>Notifications</b>. As production today: title, sentence, Download Failed Report, View List</sub></td></tr></table>

**5.6 Email.** The body is lines that scan at a glance, with the toast's numbers: "{added} prospects added", "{updated} updated", "{n} rows to fix", "{m} over your plan's limit", each on its own line, a line with a count of 0 dropped (except the first). Then "Download the CSV to fix them." and the button "Download CSV". No per-error breakdown; the reasons are in the CSV. A sequence import reads "{n} prospects added to Step {k} of {sequence}" and "{m} rows to fix". Never "Congratulations" when rows failed.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/e01.png?v=329531bd"><img src="canvas/shots/e01.png?v=329531bd" width="260"></a><br><sub><b>Email: rows to fix</b>. The toast’s numbers, then Download CSV</sub></td><td valign="top" width="33%"><a href="canvas/shots/e02.png?v=370011b8"><img src="canvas/shots/e02.png?v=370011b8" width="260"></a><br><sub><b>Email: every row failed</b>. Never “Congratulations”</sub></td><td valign="top" width="33%"><a href="canvas/shots/e03.png?v=2af4c3c5"><img src="canvas/shots/e03.png?v=2af4c3c5" width="260"></a><br><sub><b>Email: sequence import</b>. Names the sequence and the step</sub></td></tr></table>

**5.7 The rows-to-fix file.**
- Columns: `row_number` (as a spreadsheet shows it, header is row 1), `error`, `how_to_fix`, then the user's columns untouched.
- One line per row not fully imported, blank-email rows included. The file holds only rows the user can act on. Repeated rows, rows left as they were under "Leave unchanged", people already in this sequence and unsubscribed people are counts on Review, not lines in the file. Over-limit lines are the last new prospects in file order, never rows already in the list.
- Its line count equals "rows to fix" plus "over your plan's limit" in the toast and the email, and Failed in the bell.
- `error` is one of 15 fixed names, the same words on every surface. `error` and `how_to_fix` cover the first problem found; `how_to_fix` names the user's column and value.
- Second round: our three columns are replaced, never added again; original `row_number` kept; name never doubled.
- Quote any cell with `,` `"` `;` tab or line break. UTF-8 with BOM. `{name}-rows-to-fix.csv`.
- Delete the unused reasons `NoUpdate`, `NoOverwrite`, `ProspectInCallDNC`, `ContactInDncList`.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/f01.png?v=1dfe3491"><img src="canvas/shots/f01.png?v=1dfe3491" width="260"></a><br><sub><b>The rows-to-fix file</b>. row_number, error, how_to_fix, then the user’s own columns</sub></td><td valign="top" width="33%"><a href="canvas/shots/f02.png?v=51ad2219"><img src="canvas/shots/f02.png?v=51ad2219" width="260"></a><br><sub><b>Filtered by error</b>. Fix one kind at a time</sub></td></tr></table>

**5.8 Import into a sequence.** Same dialog, with these differences:
- A sequence with no steps is handled before the dialog opens, unchanged from production.
- **Map Fields** keeps production's "Import into [Step 1]" in the table footer, beside "Already in your list"; lowest step by default. It warns, in one line, when no column is mapped to a field the sequence's steps need (every step, not only the one picked); Next stays enabled. The warning uses the one message slot at the top of Map Fields: a red Email error wins, then this warning, then the grey match line.
- **Review** names the step picked on Map Fields. The headline counts prospects added to the step, new plus existing ("3,640 prospects will be added to Step 1"). The card's first row is "Added to Step {k}" with a grey note "{e} already in your list · {d} on Do Not Contact, won't be emailed" ({d} is inside the count: production adds them to the step marked blocked, so they are never emailed; [contacts-import.service.ts L3662-L3690](https://github.com/saleshandy/saleshandy-edge/blob/3e51d4654bb0579b461b09711ee43d336503a8ae/src/contacts/services/contacts-import.service.ts#L3662-L3690)). Then "Repeated rows", "Won't be added" (in this sequence or unsubscribed) and "Rows to fix" with Download CSV. What a later step needs is a grey line under the card ("1,092 prospects have no phone number for the Step 3 call."); those rows still import.
- **After:** toast "{n} added to Step {k}." and "{m} rows to fix." with [Download CSV] ({n} = prospects added to the step, {m} = rows in the download); the bell and email use the same numbers and wording as the list import with the sequence named.
- **File** includes the rows not added that the user can act on: in another sequence, active in another sequence, bounced (errors 13 to 15). Already in this sequence, unsubscribed, and rows a step can't work (they import) are counts on Review only.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/m11.png?v=f2954e15"><img src="canvas/shots/m11.png?v=f2954e15" width="260"></a><br><sub><b>Sequence: no phone or LinkedIn column</b>. Warned while mapping, for every step; Next stays enabled</sub></td><td valign="top" width="33%"><a href="canvas/shots/m12.png?v=956a8101"><img src="canvas/shots/m12.png?v=956a8101" width="260"></a><br><sub><b>Sequence: phone mapped, LinkedIn missing</b>. The warning names only what is still missing</sub></td><td valign="top" width="33%"><a href="canvas/shots/s01.png?v=b4e14d17"><img src="canvas/shots/s01.png?v=b4e14d17" width="260"></a><br><sub><b>Into an email step</b>. Added to Step 1 with who is already in the list and on Do Not Contact; who is not added</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/s02.png?v=9f24c9bf"><img src="canvas/shots/s02.png?v=9f24c9bf" width="260"></a><br><sub><b>Into a call step, some phones empty</b>. Phone mapped; a grey line under the card counts prospects without a phone</sub></td><td valign="top" width="33%"><a href="canvas/shots/s03.png?v=75e1aa48"><img src="canvas/shots/s03.png?v=75e1aa48" width="260"></a><br><sub><b>Into a LinkedIn step, some URLs empty</b>. A grey line under the card for missing LinkedIn URLs</sub></td><td valign="top" width="33%"><a href="canvas/shots/s05.png?v=4251017a"><img src="canvas/shots/s05.png?v=4251017a" width="260"></a><br><sub><b>After a sequence import</b>. The Prospects tab refreshed and says so; amber toast with the download</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/s06.png?v=23f87617"><img src="canvas/shots/s06.png?v=23f87617" width="260"></a><br><sub><b>Notifications after a sequence import</b>. As production today, with the sequence named</sub></td></tr></table>

**5.9 Bringing a file back.** A file with our three columns is a rows-to-fix file: Map Fields opens with the mapping of the import it came from, says "Mapping restored" in one line with a way to clear it, and sets our three columns to Don't import. Map Fields is not skipped. Any other file maps as production does.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/m03.png?v=26d787fa"><img src="canvas/shots/m03.png?v=26d787fa" width="260"></a><br><sub><b>Fixed file brought back</b>. Mapping restored from the import it came from; our three columns set to Don’t import</sub></td></tr></table>

### Acceptance criteria

| ID | Case | Expected | P |
|---|---|---|---|
| A1 | Upload each of the 11 rejected kinds in COPY.md §5.1 | The drop zone shows the error icon, the file name, its §5.1 title, the fix and Try again; no row total, header name or column letter appears; the zone accepts the next file and the alert is replaced by its result; Next disabled; no toast | P0 |
| A2 | No column mapped to Email; a file with no email column | §5.2 one-line red message; Next disabled | P0 |
| A3 | 4,000 rows: 220 no @, 40 blank, 20 two emails, 160 existing, 20 repeated | Review card: headline "3,540 new prospects will be added", "4,000 rows checked"; rows New prospects 3,540, Already in your list 160 with the option's note, Repeated rows 20, Rows to fix 280 with Download CSV; the rows add up to 4,000; no per-error breakdown on Review; button "Import 3,540 new prospects" | P0 |
| A4 | A3 under each After Import option | Review's note beside "Already in your list" uses that option's words; 3 of the 160 read back in the list match it | P0 |
| A5 | Download from A3 | 280 lines; header `row_number,error,how_to_fix,` + original headers; only the 15 names; blank-email rows present; row numbers match Excel | P0 |
| A6 | Open A5 in LibreOffice with tab, `;` and `,` separators | Every row has the same column count | P0 |
| A7 | Upload A5 back, import under a plan limit, download again | Mapping restored; one set of our columns; original row numbers; name not doubled | P0 |
| A8 | Room for 2,500; A3's file | Review: the card shows Over your plan's limit 1,040 and Rows to fix 280 with Download CSV; an amber line with Upgrade plan. Import: 2,500 new prospects; toast "2,500 added, 160 updated. 280 rows to fix, 1,040 over your plan's limit."; bell "4,000 rows: 2,500 added, 160 updated, 280 to fix, 1,040 over your plan's limit and 20 repeated" with Upgrade plan · Download CSV; 1,040 "Over plan limit" lines, new prospects only, numbers within the file | P1 |
| A9 | Import fails; another import running; plan limit reached between Review and Import | Dialog stays on Review with the matching one-line message; file and mapping kept; busy known before Import; nothing imported | P0 |
| A10 | Every row fails | Headline "None of the {rows} rows can be imported"; Download primary, Upload fixed file on the left; dominant-reason hint over 50% | P1 |
| A11 | Credits short | The credits line under Verify emails after import shows credits needed and held, in amber, with Add credits; Import disabled until unticked or topped up; "Change the 15 days" stays | P1 |
| A12 | Force a batch to fail mid-import | Its rows in the file as "Didn't finish"; toast, bell and email count them | P0, **sign off by name: Rajat** |
| A13 | A3 imported; then a sequence import of the same file | List: toast "3,540 added, 160 updated. 280 rows to fix."; bell "4,000 rows: 3,540 added, 160 updated, 280 to fix and 20 repeated"; email "3,540 prospects added" / "160 updated" / "280 rows to fix". Sequence: Review 3,640 + 20 repeated + 15 won't be added + 325 rows to fix = 4,000; toast "3,640 added to Step {k}. 325 rows to fix."; bell "4,000 rows: 3,640 added to Step {k}, 325 to fix and 35 not added". Every download link reads "Download CSV"; the toast stays until dismissed | P1 |
| A14 | Sequence import: same sequence, other sequence, bounced, unsubscribed | Other sequence and bounced in the file (errors 13 to 15); same sequence and unsubscribed counted on Review only | P1 |
| A15 | Email for a partial and a fully failed import | Real counts; never "Congratulations" | P1 |
| A16 | Keyboard only through the wizard | §7 rules hold; failures and the finished toast are announced | P1 |
| A17 | 390px wide | Map rows stack; no sideways scroll; footer buttons full width, primary last; toast never covers the footer | P1 |
| A18 | Every footer at 1440px | Gray secondary left of primary, 16px apart; Back on Map and Review, Cancel on Upload | P2 |
| A19 | Map a column to Email, then open another column's picker | Email isn't offered | P1 |
| A20 | Sequence import with a call and a LinkedIn step, no phone or LinkedIn column | Map Fields warns in one line naming both fields and steps; Next stays enabled; mapping a phone column drops the call step from the warning; Review shows no unmapped-field line | P1 |
| A21 | Sequence import, prospects in another sequence, admin setting off | Counted on Review before Import; "In another sequence" in the file; toast counts only those added | P0 |
| A22 | Upload the fixed file from A5 | Map Fields opens with "Mapping restored", every column as in A3, our three columns Don't import | P1 |
| A23 | Upload a valid 6-column and an 18-column file | One file line, then the first 3 rows under the file's headers as read; "First 3 of {rows} rows" sits under the file name; long values truncate with the full value on hover; the wider table scrolls sideways; nothing under the table | P1 |
| A24 | Map a file whose first rows hold a 70-character company name and a 120-character note | Every column row stays one line; long values end in "…", values of 14 characters or fewer are whole; hovering a value shows it in full | P2 |
| A25 | Import finishes while the user is on the list it went into | New prospects appear first without a reload; the screen shows the list was just updated | P1 |
| A26 | Sequence import where some prospects are on the sequence's or the account's Do Not Contact list | Review headline counts them (they are added to the step); the card's "Added to Step" note says "{d} on Do Not Contact, won't be emailed"; none of them is emailed | P1 |

## 6. Reconciliation with existing behaviour

| Path | Today | After |
|---|---|---|
| CRM list › Import CSV (`/contacts/import-async`) | Two-column report | This PRD |
| Sequence › Import via CSV (`/contacts/sequence-async/:stepId`) | Same report, picker on Map Fields | This PRD, §5.8 |
| Public API import | 7 reasons have no status code; `status` comes back undefined ([gateway](https://github.com/saleshandy/saleshandy-open-api-gateway/blob/3561cf80b480bb48cf0670c22a9c7f977012a4fc/src/common/enums/prospect-import-failed-reason.ts#L1)) | Add the missing codes; copy out of scope |
| Lead Finder bulk add | Shares the import service | Must not change |

- **Overwrite** writes only fields the prospect already has; production's description says so and stays verbatim. No behaviour change.

## 7. Cross-cutting

| Rule | |
|---|---|
| No green toast, success icon or "Congratulations" for an outcome with failures | Release blocker |
| No "Oops!", raw server text or JSON anywhere in the flow | Release blocker |
| One name per error, same words on every surface | Release blocker |
| Footer as production's import dialog: 20/24 padding, 16px gap, link left, gray secondary then primary right; full width below 720px | |
| Dialog semantics, Tab trapped, page behind not reachable by keyboard or screen reader, Esc closes a list then the dialog, focus back on the control that opened it, focus kept on re-render, failures `role="alert"` (a rejected file puts focus on Try again; a failed import on Import again), each field picker named "Field for {column}: {field}", Email's required mark also in words, the verification checkbox named by its action only with the hint as its description, text fields with a visible focus ring, checking progress and the Review result announced, the finished toast announced | |
| A disabled Import or Next stays reachable by keyboard and is described by the line that says why (credits, another import, no email mapped). Each field picker is named with its column. Text links have a hit area at least 24px high | |
| Contrast at least 4.5:1; no functional text under 11px | |
| Upgrade plan and Add credits open Billing and the credits checkout. "Change the 15 days" opens Smart Verification in settings (account owner and admins only) | |
| Colour: red for rows to fix and failed imports; amber for needs attention when nothing in the file is wrong (plan limit, credits, another import running); grey for what needs nothing from the user (step needs, prospects not added to a step); blue for the one action left | |
| No "Saleshandy" in any message (white-label) | |

At 390px:

<table><tr><td valign="top" width="33%"><a href="canvas/shots/p01.png?v=10456193"><img src="canvas/shots/p01.png?v=10456193" width="260"></a><br><sub><b>Rejected file</b>. Error inside the drop zone, full width</sub></td><td valign="top" width="33%"><a href="canvas/shots/p02.png?v=e1b4aa9a"><img src="canvas/shots/p02.png?v=e1b4aa9a" width="260"></a><br><sub><b>Map Fields</b>. Rows stack; picker under each column name</sub></td><td valign="top" width="33%"><a href="canvas/shots/p03.png?v=11035463"><img src="canvas/shots/p03.png?v=11035463" width="260"></a><br><sub><b>Review</b>. Lines wrap under their own text; footer buttons full width</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/p04.png?v=e52acabb"><img src="canvas/shots/p04.png?v=e52acabb" width="260"></a><br><sub><b>Sequence Review</b>. The card at phone width; counts stay on their rows</sub></td><td valign="top" width="33%"><a href="canvas/shots/p05.png?v=40b14743"><img src="canvas/shots/p05.png?v=40b14743" width="260"></a><br><sub><b>After import</b>. List line wraps; its link drops below the sentence</sub></td></tr></table>

## 8. Implementation notes

- **Pre-import check:** row checks plus existing-prospect and sequence-membership lookups over the whole file. **Resolve before estimation:** time on 100,000 rows.
- **Failed-row storage:** one entry per source row with its number and cells, replacing one per (reason, email) via `$addToSet` ([L6717](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L6717)). **Resolve before estimation:** size at 100,000 rows of 120 columns.
- **Mapping restore:** each import already keeps its mapping; a rows-to-fix file needs a way back to the import it came from.
- **List refresh:** the polling that drives the progress card already knows when the import finishes; refetch the open list then.
- **Busy check on Review** needs "is an import running for this user".
- **Upload checks move to the browser:** mixed separators and a leading empty line are read from the header line the browser already parses ([pre-select.tsx L61](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/select-file/components/pre-select/pre-select.tsx#L61)); not UTF-8 reuses the existing `is-utf8-file` util. The server keeps its checks as a backstop.

## 9. Measurement

**Never log** emails, cell values or file names; counts and error names only. Events ship with the feature.

| Event | Fires when | Key properties |
|---|---|---|
| `csv_import_step_viewed` | Each step opens | `step`, `path` |
| `csv_import_file_rejected` | Upload rejects | `reason` |
| `csv_import_blocked` | Next or Import disabled with a message | `reason` |
| `csv_import_reviewed` | Review shown | `new`, `existing`, `to_fix`, `reasons`, `after_import` |
| `csv_import_failed` | Import press fails | `reason` |
| `csv_import_completed` | Job finishes | `total_rows`, `added`, `updated`, `not_imported`, `reasons`, `path` |
| `csv_import_report_downloaded` | Any download | `rows`, `from` |
| `csv_import_reuploaded` | A file with our columns comes back | `mapping_restored`, `round` |

- **Most important signal:** fix loop closed, imports with rows to fix followed by a re-upload within 7 days. Target set at day 14 (no baseline).
- **Guardrails:** Checking median over 30 s for files under 10,000 rows (target, Rajat to confirm): rethink the pre-import check before release. Any import whose `not_imported` exceeds its file's lines: stop and fix.

## Appendix

- [Prototype](https://product-team-sh.github.io/prd-prospect-csv-import/prototype/): the intro card picks list or sequence import; the Account menu switches failure states; "Other rejected files" on Upload holds the rest of §5.1.
- [Every screen](https://product-team-sh.github.io/prd-prospect-csv-import/canvas/): 61 states captured from the prototype.
- [COPY.md](COPY.md): every string, the file rules and all 15 errors with their code sites.
