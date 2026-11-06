# PRD: Prospect CSV import errors users can understand and fix

**Design of record:** [prototype](https://product-team-sh.github.io/prd-prospect-csv-import/prototype/) (commit [`998e84b`](https://github.com/product-team-sh/prd-prospect-csv-import/commit/998e84b)) · **Copy:** [COPY.md](COPY.md), rendered in [copy.html](https://product-team-sh.github.io/prd-prospect-csv-import/copy.html) · **Status:** Draft, design sign-off pending

Card: [Self-Understandable / Self-Diagnosable Error Message for PROSPECT CSV upload Module](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10153297291). Code read 6 Nov 2026: webui `f9425f2`, edge `23c74c8`, background-jobs `7d18274`.

---

## 1. Problem

- **Most prospects arrive this way.** 71.0% of prospects created in the 90 days to 8 Sep came from CSV import, from about 1,836 users a week (measured, PostHog `Prospects Added`, `method = csv`).
- **Nothing is checked before import, and the result is unreadable.** Every outcome ends in a green toast. The report is `email,failed_reason`: one line per email, no row number, none of the user's columns ([writer](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L6838)). Six different email defects all read "Invalid Email" (measured, 5 Sep).
- **Some failures leave no trace.** Blank-email rows, invalid phones, crashed batches and Do Not Contact matches produce no report line. 0 of 409,820 import events in 90 days recorded a failure, because the only event fires on success (measured, 8 Sep).

## 2. Why this approach

| Option | Verdict |
|---|---|
| Better strings in today's report | Rejected: still no row numbers or user columns, still nothing before commit |
| A grid of failed rows in the dialog (card's original ask) | Rejected: can't fit a 720px dialog at 40,000 rows; reasons fit in five lines at any size |
| **Check before commit, reasons on screen, the user's own rows back with a fix per row** | **Chosen** |

Not claimed: fewer tickets or failed imports. Neither has a baseline.

## 3. What we're building

1. Every row is checked before anything is imported; Review shows what Import will do and, separately, what needs the user.
2. Nothing fails silently: every row not fully imported is in a file of the user's own rows, filterable by error, with one fix per row.
3. The flow is instrumented from day 1.

**Invariant:** nothing is written until Import is pressed; no failure loses the user's file or mapping.

## 4. User stories

| ID | As a user importing prospects, I… |
|---|---|
| U1 | see before importing how many will be added, what happens to ones already in my list, and what needs fixing and why |
| U2 | download only the rows I must act on, in my own columns, filter by error, and upload the fixed file without mapping again |
| U3 | am told why a file is rejected, and how to fix it, without restarting |
| U4 | stay on Review with my file and mapping if Import fails or another import is running |
| U5 | can import what fits when my plan or credits are short |
| U6 | pick the sequence step on Review and see who won't be added, and who the step can't work, before importing |
| U7 | skip Map Fields when a file with the same columns comes back |
| U8 | (product team) can read failure rates and reasons from day 1 |

## 5. Decisions

| ID | Decision | Consequence to accept |
|---|---|---|
| D1 | Every row is checked before Import. Counts after import come from the import, not the check | A second pass over the file (§12 Q1) |
| D2 | No grid of failed rows in the dialog (declines the card's ask) | 3 bad rows still means a download |
| D3 | No fourth step: Import closes the dialog; production's progress card shows progress; a strip and the bell hold leftover work | The bell is the only lasting record |
| D4 | All four After Import options stay, default Add Missing Fields, as production ([map-fields.tsx L74-80](https://github.com/saleshandy/saleshandy-webui/blob/f9425f234ef9f23a3d1fe9a1bfd7b326b61b1ae5/src/components/sequence/components/sequence-single-content/components/sequence-single-contacts/components/map-fields/map-fields.tsx#L74-L80)). Existing rows are an outcome, never a failure | Every surface states the chosen option's outcome |
| D5 | The file holds only rows to act on; repeats and Skip Existing rows are counts on Review | A repeated row with different data shows only as a count |
| D6 | Email stays required this release (§12 Q3) | LinkedIn-only lists are still blocked, now with a message |
| D7 | Import button counts new prospects ("Import 3,540 new prospects") | No longer equals rows sent |
| D8 | One release | Nothing lands early; carries D1's risk |
| D9 | Mapping remembered by header set, ignoring our three columns; account-wide | Teammates with identical headers share it (§12 Q4) |
| D10 | Sequence import: the step is picked on Review, not Map Fields (reverses production) | Map Fields no longer shows the destination |
| D11 | Missing step needs are warned about, not held back: an unmapped field (phone, LinkedIn) on Map Fields, covering every step; rows with empty cells on Review | Email-only prospects still land in call steps, as today, but the user is told |
| D12 | A fully restored mapping skips Map Fields | Changing a column means Review mapping or Back |

## 6. Surfaces

Exact copy for every state below: [COPY.md](COPY.md), same section numbers; every string rendered on its surface in [copy.html](https://product-team-sh.github.io/prd-prospect-csv-import/copy.html).

**6.1 Upload.** Reject a bad file in the drop zone with its name, reason and fix; Next stays disabled; no toasts or "Oops!". Every check that today fails only after Import (duplicate headers, mixed separators, leading empty line, upper-case `.CSV`) runs here.

**6.2 Map Fields.** Email required; if unmapped, say so and disable Next. After Import below the fields, production's four options verbatim. One column per field (picking a used field moves it and says so). Cancel, Esc or close keeps the mapping. Our three columns come back as Don't import.

**6.3 Checking.** Progress in rows; "Nothing has been imported yet"; one Cancel.

**6.4 Review.** The number first, then plain grey lines for what Import will do. Below, one box with only what needs the user, one row each with its action: step-needs warning, over the plan limit, rows to fix. Clean file: no box. No bar, no coloured dots. States: plan limit, plan full, credits short, another import running (known on arrival, Import disabled), Import failed (dialog stays, "Import again"), every row fails (Download is primary, Upload fixed file on the left; a dominant reason suggests checking the Email mapping). When the download holds more than the rows to fix, the fix row says what else is in it.

**6.5 After Import.** Production's progress card, closed when finished (today it then claims every row imported). Toast for what was added; strip on the list for what's left, with Download and Upload fixed rows; bell keeps both. Counts come from the import.

**6.6 Email.** Real counts and top three error names; never "Congratulations" when rows failed.

**6.7 The rows-to-fix file.**
- Columns: `row_number` (as a spreadsheet shows it, header is row 1), `error`, `how_to_fix`, then the user's columns untouched.
- One line per row not fully imported, blank-email rows included.
- `error` is one of 19 fixed names, the same words on every surface. `how_to_fix` names the user's column and value.
- Second round: our three columns are replaced, never added again; original `row_number` kept; name never doubled.
- Quote any cell with `,` `"` `;` tab or line break (LibreOffice and European Excel split on `;`, reproduced 6 Nov). UTF-8 with BOM. `{name}-rows-to-fix.csv`.
- Delete the unused reasons `NoUpdate`, `NoOverwrite`, `ProspectInCallDNC`, `ContactInDncList`.

**6.8 Import into a sequence.** Same dialog. Entry blocked if the sequence has no steps. Review starts with "Import into [Step · type]"; the headline counts prospects added to the step (new plus existing, as production). Lines for existing prospects added, and those turned away (another sequence per the admin setting, already in this sequence, bounced or unsubscribed). Map Fields warns when no column is mapped to a field the sequence's steps need (every step, not only the one picked). Review's box warns only about rows with that field empty. Button "Add {n} to Step {k}".

**6.9 Bringing a file back.** Headers match a confirmed mapping, every column decided, Email mapped: Upload says "Mapping restored" and Next becomes "Check {rows} rows", skipping Map Fields.

### Acceptance criteria

| ID | Case | Expected | P |
|---|---|---|---|
| A1 | Upload .xlsx, header-only, duplicate headers, mixed separators | Rejected in the drop zone with COPY.md §6.1 copy; Next disabled; no toast | P0 |
| A2 | No column mapped to Email; a file with no email column | §6.2 message; Next disabled | P0 |
| A3 | 4,000 rows: 220 no @, 40 blank, 20 two emails, 160 existing, 20 repeated | Review: 3,540 new; 160 existing with the option's outcome; 20 repeated; box "280 rows need fixing" with three reasons; button "Import 3,540 new prospects" | P0 |
| A4 | A3 under each After Import option | Review, toast and bell use that option's words; 3 of the 160 read back in the list match it | P0 |
| A5 | Download from A3 | 280 lines; header `row_number,error,how_to_fix,` + original headers; only the 19 names; blank-email rows present; row numbers match Excel | P0 |
| A6 | Open A5 in LibreOffice with tab, `;` and `,` separators | Every row has the same column count | P0 |
| A7 | Upload A5 back, import under a plan limit, download again | Mapping restored; one set of our columns; original row numbers; name not doubled | P0 |
| A8 | Room for 2,500; A3's file | 2,500 added; 1,040 "Over plan limit" lines, new prospects only, numbers within the file | P1 |
| A9 | Import fails; another import running | Dialog stays on Review; file and mapping kept; busy known before Import | P0 |
| A10 | Every row fails | "0 of {rows}"; Download primary, Upload fixed file on the left; dominant-reason hint over 50% | P1 |
| A11 | Credits short | Credit message with numbers; Import disabled until unticked or topped up | P1 |
| A12 | Force a batch to fail mid-import | Its rows in the file as "Didn't finish"; strip, bell and email count them | P0, **sign off by name: Rajat** |
| A13 | Invalid phone on a new prospect | Imported, phone blank, "Phone not valid" line | P1 |
| A14 | Sequence import: other sequence, bounced, unsubscribed, DNC | Each in the file under errors 13 to 18 | P1 |
| A15 | Email for a partial and a fully failed import | Real counts; never "Congratulations" | P1 |
| A16 | Keyboard only through the wizard | §8 rules hold; failures and the strip are announced | P1 |
| A17 | 390px wide | Map rows stack; no sideways scroll; footer buttons full width, primary last; toast never covers the footer | P1 |
| A18 | Every footer at 1440px | Gray secondary left of primary, 16px apart; Back on Map and Review, Cancel on Upload | P2 |
| A19 | Map two columns to Email | Second pick moves it and says so | P1 |
| A20 | Sequence import with a call and a LinkedIn step, no phone or LinkedIn column | Map Fields warns naming both fields and steps; Next stays enabled; mapping a phone column drops the call step from the warning; Review shows no unmapped-field row | P1 |
| A21 | Sequence import, prospects in another sequence, admin setting off | Counted on Review before Import; "In another sequence" in the file; toast counts only those added | P0 |
| A22 | Upload the fixed file from A5 | "Mapping restored"; "Check {rows} rows" lands on Review without Map Fields | P1 |

## 7. Reconciliation with existing behaviour

| Path | Today | After |
|---|---|---|
| CRM list › Import CSV (`/contacts/import-async`) | Two-column report | This PRD |
| Sequence › Import via CSV (`/contacts/sequence-async/:stepId`) | Same report, picker on Map Fields | This PRD, §6.8 |
| Public API import | 7 reasons have no status code; `status` comes back undefined ([gateway](https://github.com/saleshandy/saleshandy-open-api-gateway/blob/3561cf80b480bb48cf0670c22a9c7f977012a4fc/src/common/enums/prospect-import-failed-reason.ts#L1)) | Add the missing codes; copy out of scope |
| Lead Finder bulk add | Shares the import service | Must not change |

- **Overwrite** writes only fields the prospect already has; production's description says so and stays verbatim. No behaviour change.
- Repeats are caught only within a 1,000-row chunk ([L2534](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L2534)); the import must match D1's whole-file check, or counts drift.

## 8. Cross-cutting

| Rule | |
|---|---|
| No green toast, success icon or "Congratulations" for an outcome with failures | Release blocker |
| No "Oops!", raw server text or JSON anywhere in the flow | Release blocker |
| One name per error, same words on every surface | Release blocker |
| Footer as production's import dialog: 20/24 padding, 16px gap, link left, gray secondary then primary right; full width below 720px | |
| Dialog semantics, Tab trapped, Esc closes a list then the dialog, focus kept on re-render, failures `role="alert"` | |
| Contrast at least 4.5:1; no functional text under 11px | |
| Upgrade plan, Add credits and Change the 15 days open Billing, the credits checkout and Smart Verification settings; the last is admins only | |
| No "Saleshandy" in any message (white-label) | |

## 9. Implementation notes

- **Pre-import check (D1):** row checks plus existing-prospect and sequence-membership lookups over the whole file. **Resolve before estimation:** time on 100,000 rows (§12 Q1).
- **Failed-row storage:** one entry per source row with its number and cells, replacing one per (reason, email) via `$addToSet` ([L6717](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L6717)). **Resolve before estimation:** size at 100,000 rows of 120 columns.
- **All problems per row:** the check chain stops at the first hit today ([L2534-L2563](https://github.com/saleshandy/saleshandy-background-jobs/blob/7d18274f935ac261db9fa538aa3afe3d72d6894c/src/contacts/services/contacts-import.service.ts#L2534-L2563)). If costly, ship the first problem only.
- **Busy check on Review** needs "is an import running for this user".

## 10. Measurement

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
- **Guardrails:** Checking median over 30 s for files under 10,000 rows (target, Rajat to confirm), put the check behind its fallback. Any import whose `not_imported` exceeds its file's lines, halt.

## 11. Rollout

| Stage | Who | Gate |
|---|---|---|
| 1 | Internal accounts | A1 to A22 pass on the 17 QA fixtures; events fire |
| 2 | 10% of accounts (blast radius, not an A/B test) | One week; Review and import counts within 1%; Checking guardrail holds |
| 3 | Everyone | Stage 2 held |

## 12. Open questions

| # | Question | Recommendation |
|---|---|---|
| Q1 | **Blocking:** can the full check run on 100,000 rows in time? | Rajat sizes it. Fallback: check the first 10,000, say so, let the import report the rest |
| Q2 | Can the job report progress to the card? | Batches done over total |
| Q3 | Should the required field follow the channel? (Dhruv) | Ship with Email required; decide separately |
| Q4 | Per-user remembered mappings? (Hemanshu) | Keep account-wide |
| Q5 | How long is the report file kept? | Find the S3 rule; if none, 30 days |
| Q6 | Are DNC-flagged prospects truly never emailed? | Rajat confirms before copy freeze |
| Q7 | Show the owner's name in "Owned by a teammate"? | Yes; same account |

## 13. Priority and owners

P0 (Backlog). Target not set. Release gate: A1 to A22, A12 signed off by Rajat. **Owners needed:** backend (Rajat to name), frontend, QA.

## Appendix

- [Prototype](https://product-team-sh.github.io/prd-prospect-csv-import/prototype/): the intro card picks list or sequence import; the Account menu switches failure states. [Design canvas](https://product-team-sh.github.io/prd-prospect-csv-import/) (8 Sep).
- [COPY.md](COPY.md): every string, the file rules and all 19 errors with their code sites.
- QA: 17 CSV fixtures (malformed emails, ragged rows, separators, encodings, duplicate headers, header-only, LinkedIn-only).
- Reviews: [7 Sep](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10153297291#__recording_10278488304), [8 Sep](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10153297291#__recording_10280390333).
