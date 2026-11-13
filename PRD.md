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
- **Accepted file:** one line (name; "First 3 of {rows} rows · {columns} columns"; Remove file) over its first 3 rows under the file's own headers, exactly as read. Values wider than a cell truncate, full value on hover. A table wider than the dialog scrolls sideways, with no caption. Never more than 3 rows.
- **Rejected file:** centred in the drop zone: the file name, what is wrong and one sentence on the fix. The zone stays live for the fixed file. Next disabled; no toast, no "Oops!".
- **Eleven checks, all on Upload:** Excel, other type or no extension, over 95 MB, over 100,000 rows, over 120 columns, no prospects, repeated column names, columns with no name, mixed separators, not UTF-8, unreadable. The ones that today fail only after Import (mixed separators, leading empty line, upper-case `.CSV`) move here; the server keeps its checks as a backstop.
- **Only what the check knows:** no row totals when parsing stopped at a limit, no header names or column positions from a yes-or-no check.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/u01.png?v=67055315"><img src="canvas/shots/u01.png?v=67055315" width="260"></a><br><sub><b>Upload</b>. Drop zone; Try sample file in the footer</sub></td><td valign="top" width="33%"><a href="canvas/shots/u02.png?v=0851d15a"><img src="canvas/shots/u02.png?v=0851d15a" width="260"></a><br><sub><b>File accepted</b>. One file line, then its first 3 rows as read; Next enabled</sub></td><td valign="top" width="33%"><a href="canvas/shots/u03.png?v=8ffd202d"><img src="canvas/shots/u03.png?v=8ffd202d" width="260"></a><br><sub><b>Excel file</b>. Says only what the check knows; the drop zone stays live</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/u04.png?v=721e0f06"><img src="canvas/shots/u04.png?v=721e0f06" width="260"></a><br><sub><b>Wrong file type</b>. Says only what the check knows; the drop zone stays live</sub></td><td valign="top" width="33%"><a href="canvas/shots/u05.png?v=d1bd8721"><img src="canvas/shots/u05.png?v=d1bd8721" width="260"></a><br><sub><b>Over 95 MB</b>. Says only what the check knows; the drop zone stays live</sub></td><td valign="top" width="33%"><a href="canvas/shots/u06.png?v=d24346e2"><img src="canvas/shots/u06.png?v=d24346e2" width="260"></a><br><sub><b>Over 100,000 rows</b>. Says only what the check knows; the drop zone stays live</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/u07.png?v=d78ceb4f"><img src="canvas/shots/u07.png?v=d78ceb4f" width="260"></a><br><sub><b>Over 120 columns</b>. Says only what the check knows; the drop zone stays live</sub></td><td valign="top" width="33%"><a href="canvas/shots/u08.png?v=dd053985"><img src="canvas/shots/u08.png?v=dd053985" width="260"></a><br><sub><b>No prospects</b>. Says only what the check knows; the drop zone stays live</sub></td><td valign="top" width="33%"><a href="canvas/shots/u09.png?v=d3bc3a32"><img src="canvas/shots/u09.png?v=d3bc3a32" width="260"></a><br><sub><b>Repeated column names</b>. Says only what the check knows; the drop zone stays live</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/u10.png?v=2135c1fa"><img src="canvas/shots/u10.png?v=2135c1fa" width="260"></a><br><sub><b>Columns with no name</b>. Says only what the check knows; the drop zone stays live</sub></td><td valign="top" width="33%"><a href="canvas/shots/u11.png?v=640d4d35"><img src="canvas/shots/u11.png?v=640d4d35" width="260"></a><br><sub><b>Mixed separators</b>. Says only what the check knows; the drop zone stays live</sub></td><td valign="top" width="33%"><a href="canvas/shots/u12.png?v=48ad1976"><img src="canvas/shots/u12.png?v=48ad1976" width="260"></a><br><sub><b>Not UTF-8</b>. Says only what the check knows; the drop zone stays live</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/u13.png?v=11a7bded"><img src="canvas/shots/u13.png?v=11a7bded" width="260"></a><br><sub><b>Can’t be read</b>. Says only what the check knows; the drop zone stays live</sub></td></tr></table>

**5.2 Map Fields.**
- **Email is required.** Unmapped, or no email column in the file: say which, keep Next disabled.
- **One column per field,** as production: a field another column uses isn't offered. The picker searches; arrows move, Enter picks the first match, Esc closes it.
- **Sample values:** each column shows its first 3 values on one line. A long value truncates on its own, a value of 14 characters or fewer is never cut, the full value shows on hover.
- **After Import** below the fields: production's four options and descriptions verbatim, Add Missing Fields by default.
- **Mapping is kept** on Cancel, Esc or close. A rows-to-fix file gets its mapping back (§5.9).

<table><tr><td valign="top" width="33%"><a href="canvas/shots/m01.png?v=37585462"><img src="canvas/shots/m01.png?v=37585462" width="260"></a><br><sub><b>Columns to set</b>. Two columns highlighted; After Import below</sub></td><td valign="top" width="33%"><a href="canvas/shots/m02.png?v=d732f313"><img src="canvas/shots/m02.png?v=d732f313" width="260"></a><br><sub><b>Everything matched</b>. Nothing to set</sub></td><td valign="top" width="33%"><a href="canvas/shots/m03.png?v=ae3815eb"><img src="canvas/shots/m03.png?v=ae3815eb" width="260"></a><br><sub><b>Fixed file brought back</b>. Mapping restored from the import it came from; our three columns set to Don’t import</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/m04.png?v=2d2058c0"><img src="canvas/shots/m04.png?v=2d2058c0" width="260"></a><br><sub><b>Field picker</b>. Search; a field another column uses isn’t offered</sub></td><td valign="top" width="33%"><a href="canvas/shots/m06.png?v=438f352a"><img src="canvas/shots/m06.png?v=438f352a" width="260"></a><br><sub><b>Email not mapped</b>. Next disabled until a column is set to Email</sub></td><td valign="top" width="33%"><a href="canvas/shots/m07.png?v=ff2a8818"><img src="canvas/shots/m07.png?v=ff2a8818" width="260"></a><br><sub><b>No email column</b>. Said plainly; Next disabled</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/m08.png?v=1f1f1863"><img src="canvas/shots/m08.png?v=1f1f1863" width="260"></a><br><sub><b>After Import: Update All Fields</b>. Production label and text, shown under the select</sub></td><td valign="top" width="33%"><a href="canvas/shots/m09.png?v=8ae4b965"><img src="canvas/shots/m09.png?v=8ae4b965" width="260"></a><br><sub><b>After Import: Overwrite</b>. Production label and text, shown under the select</sub></td><td valign="top" width="33%"><a href="canvas/shots/m10.png?v=9d6b05fe"><img src="canvas/shots/m10.png?v=9d6b05fe" width="260"></a><br><sub><b>After Import: Skip Existing Prospects</b>. Production label and text, shown under the select</sub></td></tr></table>

**5.3 Checking.** Every row is checked, with the existing-prospect and sequence lookups, before Review. Progress in rows; "Nothing has been imported yet"; one Cancel, which imports nothing.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/c01.png?v=8f57cecd"><img src="canvas/shots/c01.png?v=8f57cecd" width="260"></a><br><sub><b>Checking every row</b>. Nothing imported yet; one Cancel</sub></td></tr></table>

**5.4 Review.**
- **One pattern:** every outcome is one row: grey icon, number, sentence. First row: new prospects to add, with the rows checked and file name under it. Then rows already in the list (worded by the After Import option, with Change), repeats, and anything that needs the user.
- **Colour only where the user acts:** red for rows to fix; amber for plan limit and step needs; a button only on those rows. Clean file: one row, "Every row is ready." No boxes, no bar. Tags and Verification stay below a rule, as production.
- **Rows to fix** list the counts by fixed error name and offer the download. When the download holds more than the rows to fix (over the plan limit, not added to the step), the row says so.
- **Import button** names the new prospects ("Import 3,540 new prospects"), or "Update {n} prospects" when none are new. Updates are stated on their own line.

**Review states:**

| State | Requirement |
|---|---|
| Plan limit | Amber row with the count over and the room left; Upgrade plan. Rows already in the list don't count |
| Plan full | No new prospects added; Import disabled unless rows already in the list can be updated |
| Credits short | Credits needed and the balance; Import disabled until credits are added or verification is unticked |
| Another import running | Known on arrival at Review, not on Import; Import disabled; Check again |
| Import fails, or the limit is reached between Review and Import | Dialog stays on Review; nothing imported; file and mapping kept; button becomes "Import again" |
| Every row fails | First row "0 of {rows}"; Download is primary, Upload fixed file on its left; when one reason covers over half the rows, suggest checking the Email mapping |

Verification states the 15-day skip; it is a fixed default, with no link to change it. "Importing…" while the request is sent.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/r01.png?v=c213d429"><img src="canvas/shots/r01.png?v=c213d429" width="260"></a><br><sub><b>Review</b>. One row per outcome; colour and a button only on rows that need you</sub></td><td valign="top" width="33%"><a href="canvas/shots/r02.png?v=b8c77d93"><img src="canvas/shots/r02.png?v=b8c77d93" width="260"></a><br><sub><b>Clean file</b>. One row: every row is ready</sub></td><td valign="top" width="33%"><a href="canvas/shots/r03.png?v=470eb7e7"><img src="canvas/shots/r03.png?v=470eb7e7" width="260"></a><br><sub><b>Already in your list: Update All Fields</b>. The line says what this option does to them</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/r04.png?v=761c21aa"><img src="canvas/shots/r04.png?v=761c21aa" width="260"></a><br><sub><b>Already in your list: Overwrite</b>. The line says what this option does to them</sub></td><td valign="top" width="33%"><a href="canvas/shots/r05.png?v=308f8d55"><img src="canvas/shots/r05.png?v=308f8d55" width="260"></a><br><sub><b>Already in your list: Skip Existing Prospects</b>. The line says what this option does to them</sub></td><td valign="top" width="33%"><a href="canvas/shots/r06.png?v=efd5eee0"><img src="canvas/shots/r06.png?v=efd5eee0" width="260"></a><br><sub><b>Over the plan limit</b>. Amber: the account needs room; red stays for rows to fix</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/r07.png?v=1cb220e0"><img src="canvas/shots/r07.png?v=1cb220e0" width="260"></a><br><sub><b>Plan full</b>. Grey headline; only rows already in the list can be updated</sub></td><td valign="top" width="33%"><a href="canvas/shots/r08.png?v=c77bc98c"><img src="canvas/shots/r08.png?v=c77bc98c" width="260"></a><br><sub><b>Not enough credits</b>. Credits needed and held; Import disabled</sub></td><td valign="top" width="33%"><a href="canvas/shots/r09.png?v=6b04a2a7"><img src="canvas/shots/r09.png?v=6b04a2a7" width="260"></a><br><sub><b>Another import running</b>. Known on arrival; Import disabled</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/r10.png?v=930e882b"><img src="canvas/shots/r10.png?v=930e882b" width="260"></a><br><sub><b>Importing…</b>. The button while the file is sent</sub></td><td valign="top" width="33%"><a href="canvas/shots/r11.png?v=eb5bc041"><img src="canvas/shots/r11.png?v=eb5bc041" width="260"></a><br><sub><b>Import failed</b>. Dialog stays; Import again</sub></td><td valign="top" width="33%"><a href="canvas/shots/r12.png?v=ca66e5a1"><img src="canvas/shots/r12.png?v=ca66e5a1" width="260"></a><br><sub><b>Limit reached during import</b>. Nothing imported; file and mapping kept</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/r13.png?v=5211d7d6"><img src="canvas/shots/r13.png?v=5211d7d6" width="260"></a><br><sub><b>Every row fails</b>. Download is primary; Upload fixed file; check the mapping</sub></td><td valign="top" width="33%"><a href="canvas/shots/r14.png?v=8f211c9b"><img src="canvas/shots/r14.png?v=8f211c9b" width="260"></a><br><sub><b>Download toast</b>. What the file holds, right after Download</sub></td></tr></table>

**5.5 After Import.**
- Production's progress card while it runs, closed when finished.
- One toast says what happened, worded by the After Import option. With rows left it is amber, says how many weren't imported and why, and offers the download. Green only when every row imported.
- The list the file went into refreshes on its own when the import finishes: no reload, new prospects first, and the screen shows the list is up to date.
- The bell keeps the outcome and the download; its unread state has an accessible name, not colour alone.
- Every count comes from the import job, not the Review estimate. Rows of a failed batch count as not imported and are in the file as "Didn't finish".

<table><tr><td valign="top" width="33%"><a href="canvas/shots/a01.png?v=70d9de79"><img src="canvas/shots/a01.png?v=70d9de79" width="260"></a><br><sub><b>Importing</b>. Production’s progress card, top right on every page</sub></td><td valign="top" width="33%"><a href="canvas/shots/a02.png?v=f16748a5"><img src="canvas/shots/a02.png?v=f16748a5" width="260"></a><br><sub><b>Finished, rows to fix</b>. The list refreshed and says so; one amber toast with the download</sub></td><td valign="top" width="33%"><a href="canvas/shots/a03.png?v=8662a258"><img src="canvas/shots/a03.png?v=8662a258" width="260"></a><br><sub><b>Finished, over the limit only</b>. Nothing wrong with the rows; the plan needs room</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/a04.png?v=383b1ab2"><img src="canvas/shots/a04.png?v=383b1ab2" width="260"></a><br><sub><b>Finished, both</b>. Rows to fix and rows over the limit in one toast</sub></td><td valign="top" width="33%"><a href="canvas/shots/a05.png?v=f635bbc4"><img src="canvas/shots/a05.png?v=f635bbc4" width="260"></a><br><sub><b>Finished, nothing left</b>. The list refreshed and says so; green toast</sub></td><td valign="top" width="33%"><a href="canvas/shots/a06.png?v=833212fd"><img src="canvas/shots/a06.png?v=833212fd" width="260"></a><br><sub><b>Notifications</b>. The bell keeps the outcome and the download</sub></td></tr></table>

**5.6 Email.** Real counts and top three error names; never "Congratulations" when rows failed.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/e01.png?v=2ef607dc"><img src="canvas/shots/e01.png?v=2ef607dc" width="260"></a><br><sub><b>Email: rows to fix</b>. Real counts and the top errors</sub></td><td valign="top" width="33%"><a href="canvas/shots/e02.png?v=7bbde9ed"><img src="canvas/shots/e02.png?v=7bbde9ed" width="260"></a><br><sub><b>Email: every row failed</b>. Never “Congratulations”</sub></td><td valign="top" width="33%"><a href="canvas/shots/e03.png?v=71cf1b4e"><img src="canvas/shots/e03.png?v=71cf1b4e" width="260"></a><br><sub><b>Email: sequence import</b>. Names the sequence and the step</sub></td></tr></table>

**5.7 The rows-to-fix file.**
- Columns: `row_number` (as a spreadsheet shows it, header is row 1), `error`, `how_to_fix`, then the user's columns untouched.
- One line per row not fully imported, blank-email rows included. The file holds only rows the user can act on. Repeated rows, rows skipped under Skip Existing Prospects, people already in this sequence and unsubscribed people are counts on Review, not lines in the file. Over-limit lines are the last new prospects in file order, never rows already in the list.
- Its line count equals "weren't imported" in the toast, bell and email.
- `error` is one of 15 fixed names, the same words on every surface. `error` and `how_to_fix` cover the first problem found; `how_to_fix` names the user's column and value.
- Second round: our three columns are replaced, never added again; original `row_number` kept; name never doubled.
- Quote any cell with `,` `"` `;` tab or line break. UTF-8 with BOM. `{name}-rows-to-fix.csv`.
- Delete the unused reasons `NoUpdate`, `NoOverwrite`, `ProspectInCallDNC`, `ContactInDncList`.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/f01.png?v=1eec4c32"><img src="canvas/shots/f01.png?v=1eec4c32" width="260"></a><br><sub><b>The rows-to-fix file</b>. row_number, error, how_to_fix, then the user’s own columns</sub></td><td valign="top" width="33%"><a href="canvas/shots/f02.png?v=fffcf137"><img src="canvas/shots/f02.png?v=fffcf137" width="260"></a><br><sub><b>Filtered by error</b>. Fix one kind at a time</sub></td></tr></table>

**5.8 Import into a sequence.** Same dialog, with these differences:
- Entry blocked if the sequence has no steps.
- **Map Fields** has no step picker. It warns when no column is mapped to a field the sequence's steps need (every step, not only the one picked); Next stays enabled.
- **Review** starts with "Import into [Step · type]", lowest step by default; changing it updates every count. The first row counts prospects added to the step (new plus existing). Then existing prospects added, and those turned away: another sequence (per the admin setting), already in this sequence, bounced, unsubscribed. An amber row covers rows with a needed field empty, from the picked step onward. Button "Add {n} to Step {k}".
- **File** includes the rows not added that the user can act on: in another sequence, active in another sequence, bounced (errors 13 to 15). Already in this sequence, unsubscribed, and rows a step can't work (they import) are counts on Review only.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/m11.png?v=7199a916"><img src="canvas/shots/m11.png?v=7199a916" width="260"></a><br><sub><b>Sequence: no phone or LinkedIn column</b>. Warned while mapping, for every step; Next stays enabled</sub></td><td valign="top" width="33%"><a href="canvas/shots/m12.png?v=389271e6"><img src="canvas/shots/m12.png?v=389271e6" width="260"></a><br><sub><b>Sequence: phone mapped, LinkedIn missing</b>. The warning names only what is still missing</sub></td><td valign="top" width="33%"><a href="canvas/shots/s01.png?v=1daa73c6"><img src="canvas/shots/s01.png?v=1daa73c6" width="260"></a><br><sub><b>Into an email step</b>. Step picked on Review; who won’t be added and why</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/s02.png?v=f7a108de"><img src="canvas/shots/s02.png?v=f7a108de" width="260"></a><br><sub><b>Into a call step, some phones empty</b>. Phone mapped; the rows without one are counted</sub></td><td valign="top" width="33%"><a href="canvas/shots/s03.png?v=5b3bc94b"><img src="canvas/shots/s03.png?v=5b3bc94b" width="260"></a><br><sub><b>Into a LinkedIn step, some URLs empty</b>. Same warning for LinkedIn profile URLs</sub></td><td valign="top" width="33%"><a href="canvas/shots/s04.png?v=d7d5fa4d"><img src="canvas/shots/s04.png?v=d7d5fa4d" width="260"></a><br><sub><b>Sequence with no steps</b>. Blocked before the dialog opens</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/s05.png?v=d528e560"><img src="canvas/shots/s05.png?v=d528e560" width="260"></a><br><sub><b>After a sequence import</b>. The Prospects tab refreshed and says so; amber toast with the download</sub></td><td valign="top" width="33%"><a href="canvas/shots/s06.png?v=26f718bc"><img src="canvas/shots/s06.png?v=26f718bc" width="260"></a><br><sub><b>Notifications after a sequence import</b>. Added to the step, and how many weren’t</sub></td></tr></table>

**5.9 Bringing a file back.** A file with our three columns is a rows-to-fix file: Map Fields opens with the mapping of the import it came from, says "Mapping restored", and sets our three columns to Don't import. Map Fields is not skipped. Any other file maps as production does.

<table><tr><td valign="top" width="33%"><a href="canvas/shots/m03.png?v=ae3815eb"><img src="canvas/shots/m03.png?v=ae3815eb" width="260"></a><br><sub><b>Fixed file brought back</b>. Mapping restored from the import it came from; our three columns set to Don’t import</sub></td></tr></table>

### Acceptance criteria

| ID | Case | Expected | P |
|---|---|---|---|
| A1 | Upload each of the 11 rejected kinds in COPY.md §5.1 | Rejected in the drop zone with its §5.1 title and fix; no row total, header name or column letter appears; the zone accepts the next file; Next disabled; no toast | P0 |
| A2 | No column mapped to Email; a file with no email column | §5.2 message; Next disabled | P0 |
| A3 | 4,000 rows: 220 no @, 40 blank, 20 two emails, 160 existing, 20 repeated | Review: 3,540 new; 160 existing with the option's outcome; 20 repeated; red row "280 rows need fixing" with three reasons; button "Import 3,540 new prospects" | P0 |
| A4 | A3 under each After Import option | Review, toast and bell use that option's words; 3 of the 160 read back in the list match it | P0 |
| A5 | Download from A3 | 280 lines; header `row_number,error,how_to_fix,` + original headers; only the 15 names; blank-email rows present; row numbers match Excel | P0 |
| A6 | Open A5 in LibreOffice with tab, `;` and `,` separators | Every row has the same column count | P0 |
| A7 | Upload A5 back, import under a plan limit, download again | Mapping restored; one set of our columns; original row numbers; name not doubled | P0 |
| A8 | Room for 2,500; A3's file | Review: plan row amber with Upgrade plan, fix row red. Import: 2,500 added; 1,040 "Over plan limit" lines, new prospects only, numbers within the file | P1 |
| A9 | Import fails; another import running; plan limit reached between Review and Import | Dialog stays on Review with the matching message; file and mapping kept; busy known before Import; nothing imported | P0 |
| A10 | Every row fails | "0 of {rows}"; Download primary, Upload fixed file on the left; dominant-reason hint over 50% | P1 |
| A11 | Credits short | Credit message with numbers; Import disabled until unticked or topped up; no link to change the 15 days | P1 |
| A12 | Force a batch to fail mid-import | Its rows in the file as "Didn't finish"; toast, bell and email count them | P0, **sign off by name: Rajat** |
| A14 | Sequence import: same sequence, other sequence, bounced, unsubscribed | Other sequence and bounced in the file (errors 13 to 15); same sequence and unsubscribed counted on Review only | P1 |
| A15 | Email for a partial and a fully failed import | Real counts; never "Congratulations" | P1 |
| A16 | Keyboard only through the wizard | §7 rules hold; failures and the finished toast are announced | P1 |
| A17 | 390px wide | Map rows stack; no sideways scroll; footer buttons full width, primary last; toast never covers the footer | P1 |
| A18 | Every footer at 1440px | Gray secondary left of primary, 16px apart; Back on Map and Review, Cancel on Upload | P2 |
| A19 | Map a column to Email, then open another column's picker | Email isn't offered | P1 |
| A20 | Sequence import with a call and a LinkedIn step, no phone or LinkedIn column | Map Fields warns naming both fields and steps; Next stays enabled; mapping a phone column drops the call step from the warning; Review shows no unmapped-field row | P1 |
| A21 | Sequence import, prospects in another sequence, admin setting off | Counted on Review before Import; "In another sequence" in the file; toast counts only those added | P0 |
| A22 | Upload the fixed file from A5 | Map Fields opens with "Mapping restored", every column as in A3, our three columns Don't import | P1 |
| A23 | Upload a valid 6-column and an 18-column file | One file line, then the first 3 rows under the file's headers as read; "First 3 of {rows} rows" sits under the file name; long values truncate with the full value on hover; the wider table scrolls sideways; nothing under the table | P1 |
| A24 | Map a file whose first rows hold a 70-character company name and a 120-character note | Every column row stays one line; long values end in "…", values of 14 characters or fewer are whole; hovering a value shows it in full | P2 |
| A25 | Import finishes while the user is on the list it went into | New prospects appear first without a reload; the screen shows the list was just updated | P1 |

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
| Dialog semantics, Tab trapped, Esc closes a list then the dialog, focus kept on re-render, failures `role="alert"`, the finished toast announced | |
| Contrast at least 4.5:1; no functional text under 11px | |
| Upgrade plan and Add credits open Billing and the credits checkout. The 15-day verification window is a fixed default, with no link to change it | |
| Colour: red for rows to fix and failed imports; amber for needs attention when nothing in the file is wrong (plan limit, step needs); blue for the one action left | |
| No "Saleshandy" in any message (white-label) | |

At 390px:

<table><tr><td valign="top" width="33%"><a href="canvas/shots/p01.png?v=c9e76de8"><img src="canvas/shots/p01.png?v=c9e76de8" width="260"></a><br><sub><b>Rejected file</b>. Centred; drop line wraps as one</sub></td><td valign="top" width="33%"><a href="canvas/shots/p02.png?v=3657ef91"><img src="canvas/shots/p02.png?v=3657ef91" width="260"></a><br><sub><b>Map Fields</b>. Rows stack; picker under each column name</sub></td><td valign="top" width="33%"><a href="canvas/shots/p03.png?v=37c6e896"><img src="canvas/shots/p03.png?v=37c6e896" width="260"></a><br><sub><b>Review</b>. Box rows stack; footer buttons full width</sub></td></tr><tr><td valign="top" width="33%"><a href="canvas/shots/p04.png?v=6413c4da"><img src="canvas/shots/p04.png?v=6413c4da" width="260"></a><br><sub><b>Sequence Review</b>. Step picker above the numbers</sub></td><td valign="top" width="33%"><a href="canvas/shots/p05.png?v=7ffa24f2"><img src="canvas/shots/p05.png?v=7ffa24f2" width="260"></a><br><sub><b>After import</b>. List line wraps; its link drops below the sentence</sub></td></tr></table>

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
- [Every screen](https://product-team-sh.github.io/prd-prospect-csv-import/canvas/): 64 states captured from the prototype.
- [COPY.md](COPY.md): every string, the file rules and all 15 errors with their code sites.
