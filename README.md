# Starship V Petty Cash (hosted)

Petty cash tracking for Starship V: log expenses and cash added, attach
receipt photos, run Count cash audits against the book balance, and pull a
statement for any period. Money is handled in cents; dates are
America/New_York.

## Shared database

This app shares the existing Render Postgres named **starship-v-par-db**
with the other Starship V hosted apps. It creates only its own tables, all
prefixed `pc_` (`pc_meta`, `pc_transactions`, `pc_receipts`, `pc_audits`),
so it never touches the other apps' tables. render.yaml therefore declares
only a web service and no database. After the service is created, set its
DATABASE_URL env var to the connection string of starship-v-par-db.

## Deploy on Render (Blueprint)

1. Push this repository to GitHub.
2. In Render: New + > Blueprint > connect this repository. Render reads
   render.yaml and proposes one web service (Starter), no database. Apply.
3. When prompted, set APP_PASSWORD (the shared sign-in password) and
   DATABASE_URL (the shared starship-v-par-db connection string).
   REPORT_TOKEN is generated automatically; it protects the read-only
   statement feed used by the Monday email job.
4. First boot seeds the fund, the 10 existing entries, and the 1 existing
   cash audit when its tables are empty, so the balance opens at $2,400.35.

## Local run

    npm install
    node server.js          # http://localhost:8080, no password, file store

Set DATABASE_URL to use Postgres instead of the local file store.

## Seeded receipts

Five seeded entries had receipt photos in the previous app. The photo files
themselves live inside that app and cannot be extracted, so those entries
are seeded as "receipt on file" with no image bytes and are labelled in the
UI as "Receipt on file (photo in previous app)". A new photo can be attached
to any entry from the Entries view; new uploads are stored in the database
(pc_receipts) and served back, and they are appended to PDF statements.

## How audits work

Count cash · Audit compares the counted cash to the current book balance and
reports Balanced, Short, or Over with the exact difference. Audits are saved
permanently and never change the balance.

## Statement feed

GET /api/statement.json?token=<REPORT_TOKEN> returns the fund, totals, all
transactions, and all audits as JSON for the Monday email job. Add
&from=YYYY-MM-DD&to=YYYY-MM-DD to include a period statement as well.
Statements printed by the app carry the standing note from the seed data.
