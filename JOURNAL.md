# PathReview Contribution Journal

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

**Setup confirmation:** [ ] App runs locally at http://localhost:5173

Setup is not yet confirmed. The current environment has Python 3.10.12 instead
of the required Python 3.11+ and does not have Docker or Docker Compose, so
`make setup` and `make run` cannot be completed here yet.

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

Environment commands checked:

```text
python3 --version
node --version
npm --version
docker --version
docker compose version
make --version
```

Observed environment:

- Python 3.10.12
- Node.js 22.22.2
- npm 10.9.7
- GNU Make 4.3
- Docker and Docker Compose are not installed

### Next steps

- Complete project setup in an environment with Python 3.11+ and Docker, then
  update the setup confirmation above.
- Update `docs/API.md` with accurate request documentation and examples.
- Review the rendered Markdown and compare every field with the source schemas.
- Record implementation decisions and validation results in this journal.
