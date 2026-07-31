# PathReview Contribution Journal

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Sub-tasks 1–3 of PLAN.md are done. `docs/API.md` now documents the `POST /profiles`
request body (`multipart/form-data`; `github_username` ≤255, `portfolio_url` ≤500,
`resume_file`; all optional) and the `POST /reviews` request body
(`application/json`; required `profile_id` UUID), each with a field table, a `curl`
example, and its error paths. Every field name and limit was copied from
`api/routes/profiles.py`, `api/schemas/profile.py`, `api/routes/reviews.py`, and
`api/schemas/review.py` rather than written from memory, which is the mitigation
PLAN.md listed for the documentation-drift risk.

Both Week 8 open questions are now resolved against the source:

- **`text/plain` resumes:** the route's allow-list really is `application/pdf`,
  `text/markdown`, and `text/plain`, even though its docstring and its 422 message
  say only "PDF or Markdown." I documented all three, since the reference should
  describe what the code enforces, and flagged the mismatch for the maintainer.
- **Bearer-token flow:** `POST /auth/login` uses FastAPI's
  `OAuth2PasswordRequestForm`, so it takes an `application/x-www-form-urlencoded`
  body with `username` (the email) and `password` — not JSON, which is what I would
  have guessed. It returns `{"access_token", "token_type"}`. The `curl` examples use
  a real token obtained this way, so they are runnable end to end.

Sub-task 4 is done: the three reproduction assertions in
`tests/unit/test_api_doc_reproduction.py` went from red to green.

**Next steps:**
Finish sub-task 5 (repo checks against a recorded baseline), extend the reproduction
file into regression guards so the reference cannot drift back out of sync, then
open the PR.

**Blockers:**
None blocking. One thing to work around: the repo has substantial pre-existing
failures on `main`, so I recorded a baseline *before* touching anything — 56 failed
/ 375 passed unit tests, 182 ruff errors, 103 mypy errors — to prove afterwards that
my change adds none. Also, `make check` runs `black .` in **write** mode, which would
reformat 52 pre-existing files and bury a small docs change in unrelated churn, so I
run `black` only on the file I touched plus `make lint` and `make typecheck` in full.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/423

**Branch:** `docs/89-api-request-schemas` — this working branch, which holds
JOURNAL.md and PLAN.md. The PR itself is opened from
`docs/89-api-request-body-schemas`, a branch cut from `upstream/main` carrying only
the three contribution commits. Course artifacts (JOURNAL.md, PLAN.md) are 245 lines
that do not belong in the upstream project, and including them would have made the
PR reviewable only after mentally subtracting them.

**What you built:**
`docs/API.md` documented `POST /profiles` and `POST /reviews` as one-line summaries
with no request body, so calling either endpoint meant reading the FastAPI routes and
Pydantic schemas first. The fix documents both request bodies — content type, a field
table with types and constraints, a runnable `curl` example, and the error paths —
plus a short note on obtaining the bearer token from `POST /auth/login`, without which
neither example runs. It is documentation-only: no application code, schema, or API
behavior changed.

**Tests added or updated:**
`tests/unit/test_api_doc_reproduction.py` only. The three Week 8 reproduction
assertions (profiles fields, `multipart/form-data`, `application/json`) now pass, and
I added five regression assertions: the accepted resume MIME types, the 255/500 length
limits, `profile_id` plus the JSON content type scoped to the Reviews section, the
bearer requirement, and an example request per endpoint. Assertions that a term
elsewhere in the file could satisfy are scoped to one section, so `profile_id`
documented as a *path* parameter under Profiles cannot stand in for the `POST /reviews`
request body. 8 tests, all passing.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes

Both boxes use the "introduces no new failures" definition, since this codebase has
documented pre-existing failures:

| Check | Before my changes | After |
| --- | --- | --- |
| `make test-unit` | 56 failed, 375 passed | 53 failed, 383 passed |
| `make lint` (ruff) | 182 errors | 182 errors |
| `make typecheck` (mypy) | 103 errors in 26 files | 103 errors in 26 files |

Comparing the *sets* of failing test IDs before and after, the only difference is my
three reproduction tests going red to green; nothing newly fails. The file I touched
passes `ruff check` and `black --check` individually. All of this is documented in the
PR description as well.

**Draft PR feedback received from:** none — I opened the PR directly as ready for
review rather than running a draft round first.

---

## Week 8 — Reproduction & solution planning

**Reproduction commit link:**
https://github.com/lec9243/pathreview/commit/28884e0ecea8cd5ae27d5886bd5259dd17704a1a

**Reproduction summary:**
I added a failing unit test, `tests/unit/test_api_doc_reproduction.py`, that reads
`docs/API.md` and asserts it documents the request bodies of `POST /profiles`
(`github_username`, `portfolio_url`, `resume_file`, and the `multipart/form-data`
content type) and `POST /reviews` (the `application/json` body). All three
assertions fail against the current documentation, confirming the gap is real and
pinning exactly which fields and content types are missing. The expected field
names were taken from `api/routes/profiles.py`, `api/schemas/profile.py`,
`api/routes/reviews.py`, and `api/schemas/review.py`.

**PLAN.md link:** https://github.com/lec9243/pathreview/blob/docs/89-api-request-schemas/PLAN.md

**Walkthrough video (recommended):** _(not recorded)_

**Blockers or open questions:**
- Confirm whether `text/plain` resumes should be documented as officially supported:
  the route accepts it (`api/routes/profiles.py`) but its docstring says "PDF or
  Markdown," and the issue title only mentions those two.
- Confirm the exact bearer-token flow from `POST /auth/login` so the `curl` examples
  in `docs/API.md` are runnable end to end.

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/89

**Issue title:** API reference doc is missing the `POST /profiles` request body
schema

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The API reference currently lists `POST /profiles` and `POST /reviews`, but it
does not explain the request body for either endpoint. A developer reading the
document cannot determine which fields are accepted, which values are required,
or which content type to send without inspecting the source code. This affects
the API documentation in `docs/API.md` and makes the endpoints harder to use
correctly. A successful contribution will document the fields and constraints
and provide valid example requests for both endpoints.

**Branch name:** `docs/89-api-request-schemas`

**Setup confirmation:** [x] App runs locally at http://localhost:5173

Setup was completed in WSL with Python 3.11.15 and Docker Desktop. `make setup`
completed successfully, `make run` started the FastAPI and Vite development
servers, and a request to `http://localhost:5173` returned the PathReview HTML
page titled "PathReview - AI Portfolio Review Assistant."

**Cohort ledger:** [x] Issue added to cohort ledger

**GitHub claim:** [x] Comment added to issue #89

### "Is this right for me?" selection notes

- **Scope:** The issue is limited to one documentation file, `docs/API.md`, and
  does not require changes to runtime application code.
- **Expected result:** The issue clearly asks for field descriptions and example
  values for two named POST endpoints, so the completion criteria are concrete.
- **Codebase understanding:** The relevant FastAPI routes and Pydantic schemas
  are easy to locate and can serve as the source of truth for the documentation.
- **Dependencies:** The fix does not require an external AI API, database change,
  frontend change, migration, or architectural work.
- **Testing:** The documentation can be checked directly against the route
  signatures and schemas, and its Markdown and request examples can be reviewed
  for correctness.
- **Time and risk:** The issue is estimated at 2–3 hours and is small enough to
  complete, review, and revise before the Week 9 PR deadline.
- **Tier reasoning:** Tier 1 is appropriate for a first contribution to this
  codebase because the work is well-scoped while still requiring navigation of
  the API routes, schemas, and contribution standards.

### Repository and setup log — 2026-07-16

Completed:

- Confirmed that the fork is hosted at `lec9243/pathreview`.
- Added `https://github.com/ascherj/pathreview.git` as the `upstream` remote.
- Created and pushed the working branch.
- Read `docs/CONTRIBUTING.md` for branch and commit conventions.
- Inspected `docs/API.md`, the profile/review routes, and their Pydantic schemas.
- Reproduced the documentation gap described in issue #89.
- Installed Python 3.11.15 in WSL and created the project virtual environment.
- Started the PostgreSQL, Redis, and ChromaDB Docker services; PostgreSQL and
  Redis remained healthy, while the pinned ChromaDB image exited with an
  upstream NumPy compatibility error described below.
- Ran `make setup` successfully, including dependency installation, database
  migrations, seed data, pre-commit setup, and frontend dependency installation.
- Ran `make run` and verified that `http://localhost:5173` serves the PathReview
  frontend and that the FastAPI application starts on port 8000.

Environment commands checked:

```text
python3 --version
node --version
npm --version
docker --version
docker compose version
make --version
```

Initial environment:

- Python 3.10.12
- Node.js 22.22.2
- npm 10.9.7
- GNU Make 4.3
- Docker was not initially accessible from the WSL execution environment

Completed setup environment:

- Python 3.11.15 virtual environment
- Docker Engine 27.5.1 through Docker Desktop WSL integration
- Healthy PostgreSQL 16 and Redis 7 containers
- ChromaDB 0.4.22 image pulled, with a startup compatibility issue noted below
- FastAPI development server on port 8000
- Vite development server on port 5173

The repository's `/health` endpoint returned HTTP 503 because its PostgreSQL and
Redis probes reported unhealthy even though both containers and the application
were running. The repository already tracks these health-check implementation
bugs in issues #154 and #155; they are unrelated to issue #89. The health route
reported ChromaDB as healthy, but the pinned `chromadb/chroma:0.4.22` container
later exited because its startup installed NumPy 2.2.6, which removed the
`np.float_` attribute used by that ChromaDB version. Despite these pre-existing
service issues, FastAPI completed startup and the frontend page loaded correctly.

### Next steps

- Update `docs/API.md` with accurate request documentation and examples.
- Review the rendered Markdown and compare every field with the source schemas.
- Record implementation decisions and validation results in this journal.
