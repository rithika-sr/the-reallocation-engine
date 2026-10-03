# CHANGE-BRIEF — ds-newgrad-h1b-15-2051

- Author: rithika-sr · Date: 2026-10-02
- Branch: contrib/2026fa-rithika-sr-ds-newgrad-h1b-15-2051
- These are my ORIGINAL predictions. I will not edit them; revisions go in the
  "Revisions" section at the bottom.

## 1. Situation and engine layers

An international F-1 master's student in data science, graduating December 2026,
with an OPT EAD expected in January 2027, targeting entry-level **Data Scientist**
roles (BLS SOC 15-2051) that can start Jan–Feb 2027 at employers with an H-1B
sponsorship record.

Persona: **Sasha Jones (fictional)**, `sasha.jones@example.com`, defined in
`scripts/contrib/2026fa/rithika-sr-ds-newgrad-h1b-15-2051/fixtures/persona.example.json`.
Visa fields are modeled on `search/examples/priya-nair/profile.yml`; dates are
changed to my situation and labeled `your-input`.

Layers used:
- **80 Days to Stay** — sponsorship history and funding (the CSV).
- **The Cognitive Pivot** — BLS wage for 15-2051 (report only).
- **Job-Ops** — liveness, as a gate only.

## 2. What I reuse (exact paths) and what I propose

Reused:
- `data/80-days-to-stay/80-days-csv/mapped_student_employment_targets_v3.csv`
  (30,370 lines incl. header). Columns used: `company_name`, `Total Approvals`,
  `Total Denials`, `Approval_Rate`, `top_job_titles_sponsored`,
  `latest_funding_amount`, `latest_funding_date`.
- `data/bls/compact/soc_occupation_compact.csv`, row `15-2051.00`.
- `scripts/score/role-scorer.mjs` via
  `npm run score -- <roles.json> --out-dir <my folder>` (called, not copied).
- `scripts/ats/check-liveness.mjs` via `npm run ats:liveness -- <url>`
  (worked run only; never in tests, which are offline).

Deliberately NOT used:
- `data/sec/form-d/processed/sample/*.sample.json` — each holds only the first
  50 of ~16,000 filers, and the first 2026Q1 record is a pooled investment fund
  with $0 sold. A miss there means "not in sample," not "no funding." The CSV's
  funding columns are used instead.

Proposed additions:
- `[TODO: DATA SOURCE]` SOC-coded DOL LCA disclosure data. The CSV has no SOC
  column, so 15-2051-specific approval counts cannot be verified from repo data;
  the prototype only checks whether data-science titles appear in
  `top_job_titles_sponsored`.
- `[TODO: DEV]` `role-scorer.mjs` defaults a missing liveness to 1.0 labeled
  `record` (`num(role.liveness?.factor) ?? 1`). My prototype never sends an
  unchecked role to the scorer; it holds it at a gate instead.

## 3. Gates (hard stops) and what a human needs to see

| Gate | Testable condition | Human must see |
|---|---|---|
| G1 Sponsorship record | company found in CSV AND `Total Approvals` not blank | the matched CSV row; whether a miss could be a name-matching error |
| G2 Liveness | an `ats:liveness` result exists for the posting URL | the posting itself, opened by hand |
| G3 Visa timeline | factor computed from persona dates, run date, and stated hiring lag | the dates and the hiring-lag assumption |
| G4 Final decision | scorer output written to my folder | the Markdown report; human chooses Apply / Network / Skip |

## 4. Predicted failure cases and how I'll check each

1. **Company not in the CSV** → status `missing`, not scored. Check: a fixture
   company that doesn't exist; the test asserts it never reaches `roles.json`.
2. **Company in the CSV with blank approvals** (e.g. row 2, `$AVY INC`) →
   "no sponsorship record," never 0 approvals. Check: a blank-approval fixture row.
3. **SOC row with no O\*NET ability data** — confirmed: `15-2051.00` has empty
   ability columns and empty `cognitive_pivot_score`. Report says "wage only;
   abilities missing." The 15-2051 wage is shared by .00/.01/.02, so it's a
   group wage, not Data-Scientist-specific.
4. **Visa window already closed** (unemployment days exhausted or OPT end date
   before the run date) → timeline 0 → Skip (gated). Check: run the test with a
   `--today` date after the window.
5. **Liveness not checked** → role held at G2, not scored.

## 5. What I expect my prototype to get wrong on the first pass

Written 2026-10-02, before any prototype code exists. Each prediction says how
I will check it; results go in "Revisions" below.

- **P1 — Title matching too narrow.** A plain "data scientist" text match will
  miss titles a hiring team treats as the same job, such as "Applied Scientist"
  or "Machine Learning Scientist." Check: compare rows matched by
  "data scientist" alone with rows matched by a small alias list, and inspect
  five of the extra rows by hand.
- **P2 — Title matching too loose.** The same match will count titles like
  "Senior Data Scientist" or "Data Scientist Intern" as evidence of new-grad
  full-time hiring, which the CSV cannot show. Check: list matched titles
  containing "senior," "lead," "principal," or "intern."
- **P3 — Company names won't match.** Exact name matching will miss companies
  entered by their common name (e.g. "Airbnb" vs. the CSV's "AIRBNB INC"), and
  those misses will look like "no sponsorship record" even when the company
  sponsors. Check: one fixture role entered under a common name.
- **P4 — Skip rate.** I expect at least 50% of roles to end as Skip or held at
  a gate, because many postings won't have a liveness check and blank approval
  fields are common. Check: the scorer's printed skip rate plus my count of
  held roles.

## Revisions

<!-- Add dated revisions here. Never edit the sections above. -->