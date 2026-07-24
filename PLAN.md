## Solution plan

**Issue:** API reference doc is missing the `POST /profiles` request body schema —
https://github.com/ascherj/pathreview/issues/89 (Tier 1 / good first issue)

Working branch: `docs/89-api-request-schemas`

### Understand

**Root cause.** `docs/API.md` documents the API surface as one-line endpoint
summaries only. `POST /profiles` and `POST /reviews` list *what* the endpoint does
but never document the request body — the accepted fields, which are required, the
content type, or example requests. The information exists only in the FastAPI route
signatures and the Pydantic schemas, so a consumer must read source code (or open
the live Swagger UI) to call either endpoint correctly.

**Expected vs. actual.**
- *Expected:* For each write endpoint, the reference states the content type, lists
  each field with type / required / constraints, and shows a valid example request.
- *Actual:* `POST /profiles` → *"Create a profile with resume and GitHub username."*
  and `POST /reviews` → *"Request a new portfolio review for a profile."* — no
  fields, no content type, no examples.

This gap is pinned by the failing reproduction test
`tests/unit/test_api_doc_reproduction.py` (three assertions, all red on current
docs). The fix should turn it green.

### Map

Files I expect to **touch**:

- `docs/API.md` — add request-body documentation for the two POST endpoints.
- `tests/unit/test_api_doc_reproduction.py` — already added as the reproduction;
  it becomes the acceptance check and should pass after the docs change (no further
  edits expected unless assertions need tightening).

Files used as **source of truth** (read-only, not modified):

- `api/routes/profiles.py` — `create_profile_endpoint`: `github_username` / `portfolio_url`
  as `Form(default=None)`, `resume_file` as `File(default=None)`; enforces resume MIME
  in `{application/pdf, text/markdown, text/plain}`, else HTTP 422; requires auth.
- `api/schemas/profile.py` — `ProfileCreate`: `github_username` max_length 255,
  `portfolio_url` max_length 500, both optional.
- `api/routes/reviews.py` — `create_review_endpoint`: JSON body `data: ReviewCreate`;
  requires auth; returns a review with `status="pending"`.
- `api/schemas/review.py` — `ReviewCreate`: `profile_id: UUID` (required).

### Plan

1. **Document `POST /profiles`.** Add a request-body subsection: content type
   `multipart/form-data`; a field table (`github_username` — optional str, ≤255;
   `portfolio_url` — optional str, ≤500; `resume_file` — optional file, PDF/Markdown/
   plain-text only, 422 otherwise); note the endpoint requires a bearer token.
2. **Document `POST /reviews`.** Add a request-body subsection: content type
   `application/json`; field `profile_id` (UUID, required); note bearer auth and the
   immediate `status="pending"` response behavior.
3. **Add example requests.** One `curl` example per endpoint (multipart `-F` for
   profiles, `-d '{"profile_id": "..."}'` for reviews) using clearly illustrative
   placeholder values.
4. **Validate against source & the reproduction test.** Re-check every documented
   field/constraint against the routes and schemas, then run
   `pytest tests/unit/test_api_doc_reproduction.py` and confirm all three assertions
   pass. Eyeball the rendered Markdown.
5. **Run repo checks.** Run `make check` / relevant `make test-unit` where the
   environment allows, recording any pre-existing unrelated failures separately.

### Inputs & outputs

- **Input:** The current route signatures and Pydantic schemas (the authoritative
  field definitions) plus the existing `docs/API.md` style.
- **Output:** An updated `docs/API.md` with two new request-body subsections (content
  type, field tables, constraints, `curl` examples) and a green reproduction test.
  No application code, schema, or API behavior changes.

### Risks & unknowns

- **Documentation drift.** Fields could be described from memory and diverge from the
  code. *Mitigation:* copy field names/constraints directly from `api/schemas/profile.py`,
  `api/schemas/review.py`, and the `Form`/`File` declarations in the routes.
- **Content-type confusion.** Profiles is multipart, reviews is JSON; mixing them
  would mislead consumers. *Mitigation:* state the content type explicitly in each
  subsection and mirror it in the `curl` flags (`-F` vs `-d`).
- **Auth requirement.** Both endpoints call `get_current_user`; an example without a
  token would 401. *Mitigation:* show an `Authorization: Bearer <token>` header in
  each example. *Unknown:* exact token acquisition flow — verify against
  `POST /auth/login` before finalizing examples.
- **Resume MIME list.** The route also accepts `text/plain`, not just PDF/Markdown as
  the issue title implies. *Unknown:* whether to document `text/plain` as supported or
  treat it as incidental — will confirm intended behavior (route comment says
  "PDF or Markdown") and document what the code actually enforces.
- **Style match.** The reference is terse; a verbose addition could clash.
  *Mitigation:* keep tables compact and match existing heading levels.

### Edge cases

- `POST /profiles` with **no fields at all** — all three are optional, so document
  that an empty profile is technically valid.
- **Invalid resume type** (e.g. `.docx` / `image/png`) → HTTP 422 "Resume must be a
  PDF or Markdown file"; document the rejection path.
- **Corrupt PDF** that fails parsing → HTTP 422 "Failed to parse PDF resume".
- **Field length overflow** (`github_username` > 255 or `portfolio_url` > 500) →
  validation error; note the limits.
- `POST /reviews` with a **missing or malformed `profile_id`** → 422; and a
  well-formed but **non-existent `profile_id`** → downstream failure — document that
  `profile_id` must reference an existing profile owned by the caller.
- **Missing/expired auth token** on either endpoint → 401; reflected in examples.
