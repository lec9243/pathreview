# PathReview Contribution Journal

## Week 10 — Iteration & reflection

### Reviewer feedback

**Feedback received:** [ ] Yes  [x] No — still awaiting review

**Summary of feedback:**
None. PR #423 has been open since 2026-07-31 and as of 2026-08-06 has no review
comments, no submitted reviews, and no requested reviewers. Nothing has been
merged or closed. Per the Summer 2026 course note, reviewer feedback is not part
of this term, so this is the expected outcome rather than a stalled PR.

**How you responded:**
No response was required. I left the PR open and unchanged rather than pushing
speculative edits to a branch nobody had commented on. The one thing I would
raise if a maintainer does pick it up is already written into the PR
description: `api/routes/profiles.py` accepts `text/plain` resumes, but that
route's own docstring and its 422 message both say only "PDF or Markdown." I
documented the behavior the code enforces and flagged the mismatch rather than
silently picking one side, because deciding which of the two is wrong is the
maintainer's call, not mine.

---

### Reflection

**What was harder than you expected?**

Getting to the point where I could trust what I wrote was harder than writing it.
I picked a Tier 1 documentation issue and estimated 2–3 hours, which was roughly
right for the prose — and almost irrelevant to the actual cost. Week 7 setup ate
far more: WSL shipped Python 3.10.12 and the project needs 3.11, Docker wasn't
reachable from my execution environment at first, the pinned
`chromadb/chroma:0.4.22` image installs NumPy 2.2.6 at startup and dies because
that release removed `np.float_`, and `/health` returned 503 the whole time
against healthy Postgres and Redis containers because of health-check bugs the
repo already tracks in issues #154 and #155.

The genuinely hard part, though, was telling my breakage apart from the repo's.
`main` already fails 56 unit tests, 182 ruff checks, and 103 mypy checks in 26
files. On my own project a red test suite is a signal; here it's the baseline,
and it meant "did I break something" was unanswerable by looking at the output.
I had to record the failure sets before touching anything and diff the sets
afterward — the only difference being my three reproduction tests going red to
green. I also didn't expect the repo's own tooling to be a hazard: `make check`
runs `black .` in **write** mode, so the sanctioned pre-submit command would have
reformatted 52 files I never opened and buried a one-file docs change in churn.
I ran `black` on my file alone plus `make lint` and `make typecheck` in full.

**What did you learn about working in a large codebase?**

That there is no single source of truth, and finding out which artifact is
authoritative is part of the work. For the resume upload, the route's allow-list,
the route's docstring, and the 422 error message gave two different answers to
"what file types are accepted" — three places, two answers, all shipped. In my
own projects that question never comes up because I hold the answer in my head.
Here I had to decide that the enforced behavior wins and that the disagreement
itself is worth reporting.

The second lesson was that assumptions are expensive. I assumed `POST /auth/login`
took a JSON body; it uses FastAPI's `OAuth2PasswordRequestForm`, so it's
`application/x-www-form-urlencoded` with `username` and `password`. Had I written
the `curl` examples from that assumption they would have looked completely
reasonable and failed for every reader. The rule I ended up working under — copy
every field name and limit out of `api/routes/`, `api/schemas/`, don't write one
from memory — was in PLAN.md as a documentation-drift mitigation, but its real
value was catching me.

Third: what the reviewer sees is part of the deliverable. My working branch
carries 245 lines of JOURNAL.md and PLAN.md that mean nothing to this project.
Opening the PR from that branch would have forced a maintainer to mentally
subtract course homework from the diff. Cutting a separate branch from
`upstream/main` with only the three real commits wasn't required by anyone; it's
just the difference between a one-file diff and a three-file one.

**How did AI tools help — and where did they fall short?**

It was most useful for orientation and for mechanical drafting. Locating the
relevant routes and Pydantic schemas in an unfamiliar FastAPI project, getting
the Markdown field tables and the test scaffolding into shape, working out the
baseline-diffing approach for a repo that's already red — all of that went much
faster than it would have alone.

It fell short in exactly the place that mattered most: anything that required
knowing what *this* code actually does. Asked to document the resume upload, a
model produces a confident, idiomatic, entirely plausible answer — "PDF or
Markdown," matching the docstring — and it's wrong, because the allow-list has a
third entry. Same with the login flow: JSON is the obvious guess and the obvious
guess is incorrect. Neither error is detectable by reading the generated text;
both are obvious after ten seconds in the source file. The failure mode isn't
that AI didn't know, it's that not knowing and knowing look identical on output.
That's the reason I ran the `curl` examples against a real token instead of
trusting that they looked right, and it's why the PLAN.md mitigation was written
as "copy from source" rather than "check the output carefully."

**What would you do differently if you started over?**

Cut the clean PR branch on day one instead of at submission time. I only split
`docs/89-api-request-body-schemas` off `upstream/main` when I was about to open
the PR and realized what the diff looked like; keeping course artifacts out of
the contribution branch from the first commit would have made that a non-event
instead of a scramble, and it's why I now have two branches to keep straight.

I'd also record the test/lint/typecheck baseline during Week 7 setup rather than
Week 9. I needed those numbers under deadline pressure to prove a negative, and
capturing them while the repo was still untouched would have been thirty seconds
of work at a point when I had time.

Finally, I carried both Week 8 open questions — `text/plain` support and the
bearer-token flow — into Week 9 as blockers, when both were answerable by
reading two files. I flagged them as questions because they felt like decisions
that needed someone else's input; they were just things I hadn't looked up yet.
I'd be quicker to separate "needs a maintainer" from "needs me to open the file."

**What are you most proud of from this module?**

The regression tests, not the documentation. Issue #89 asked for prose, and prose
rots — the whole reason the gap existed is that `docs/API.md` drifted away from
routes that kept changing. So I extended the Week 8 reproduction file into eight
assertions that pin the specific things most likely to drift: the accepted MIME
types, the 255 and 500 character limits, the content type per endpoint, the
bearer requirement, and a worked example for each. The detail I'm happiest with
is scoping: `profile_id` appears in `docs/API.md` as a path parameter under
Profiles, so a naive assertion would pass on the wrong occurrence and silently
stop guarding anything. Scoping it to the Reviews section means the test fails
when the request body is what goes missing. If the docs drift again, something
goes red — which is more than the issue asked for and the part most likely to
still matter after the PR is forgotten.

---

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
