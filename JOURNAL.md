# PathReview Contribution Journal

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
