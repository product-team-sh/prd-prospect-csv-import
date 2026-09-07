# Prospect CSV import — design + prototype

Backlog P0 card: [Self-Understandable / Self-Diagnosable Error Message for PROSPECT CSV upload Module](https://app.basecamp.com/4378325/buckets/22269754/card_tables/cards/10153297291)

| | |
|---|---|
| **[Clickable prototype](https://product-team-sh.github.io/prd-prospect-csv-import/prototype/)** | The revamped module end to end. Three sample files walk three different outcomes |
| **[Design canvas](https://product-team-sh.github.io/prd-prospect-csv-import/)** | 13 artboards — every screen, the reason taxonomy, and how this reconciles with the brief |

Scope is deliberately narrow: the user sees **how many rows failed and for what reason**, downloads their own file back with a reason column, and imports the valid rows. The failing rows are never rendered on screen — that is what keeps the review in a dialog at any file size.

Start with the prototype. Press **Add Prospect → Import via CSV**, then pick one of the three files:

- `prospects-q3-outbound.csv` — the main demo. 4,000 rows, 460 need attention across five reasons
- `verified-list-sept.csv` — nothing wrong, showing that the new machinery disappears when there is nothing to report
- `linkedin-outreach.csv` — a LinkedIn-first list with no email column. Today this dead-ends silently; here it says why

All "today" behaviour shown in the prototype was reproduced live in `my.saleshandy.com` on 5 September 2026.

Design system: Saleshandy DS v3 — Blue 700 `#1D4ED8` primary, Inter, tokens inlined so both pages render standalone.
