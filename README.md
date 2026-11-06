# Prospect CSV import: design and prototype

Backlog P0 card: [Self-Understandable / Self-Diagnosable Error Message for PROSPECT CSV upload Module](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10153297291)

| | |
|---|---|
| **[Clickable prototype](https://product-team-sh.github.io/prd-prospect-csv-import/prototype/)** | The revamped module end to end. List import, sequence import, and every failure state |
| **[PRD](PRD.md)** | Draft, design sign-off pending: decisions, the copy for every state, all 19 errors in the download file, acceptance criteria, measurement |
| **[Design canvas](https://product-team-sh.github.io/prd-prospect-csv-import/)** | 13 artboards (8 Sep): every screen, the reason taxonomy, and how this reconciles with the brief |

Scope: before anything is imported, the user sees what Import will do and, in one box, only the rows that need them, each with its action. Rows that need fixing come back as the user's own file: `row_number`, `error`, `how_to_fix`, then their columns, filterable by error. The fixed file comes straight back with its mapping restored.

Start with the prototype and pick a path on the intro card:

- `prospects-q3-outbound.csv`: the main demo. 4,000 rows: 280 need fixing, 160 already in your list, 20 repeated
- `verified-list-sept.csv`: nothing wrong, so nothing extra is shown
- `linkedin-outreach.csv`: no email column, said plainly on Map Fields
- **Import into a sequence**: the same file from a sequence's Prospects tab; the step is picked on Review, with who won't be added and why

Failure states: the **Account** menu in the top bar switches to a plan limit, low verification credits, a failed Import, or another import running. On Upload, `contacts-export.xlsx` and `event-leads.csv` are rejected files, and `webinar-signups.csv` is a file where every row fails.

All "today" behaviour shown in the prototype was reproduced live in `my.saleshandy.com` on 5 September 2026.

Design system: Saleshandy DS v3, Blue 700 `#1D4ED8` primary, Inter, tokens inlined so both pages render standalone.
