# Issue #89 Contribution Journal

## 2026-07-16 — Issue selection and repository orientation

### Completed

- Selected PathReview issue #89, a Tier 1 documentation issue.
- Commented on the GitHub issue to claim the work.
- Registered issue #89 in the AI201 cohort issue ledger.
- Confirmed that the fork is hosted at `lec9243/pathreview`.
- Added the official repository as the `upstream` remote.
- Created the contribution branch `docs/89-api-request-schemas`.
- Read `docs/CONTRIBUTING.md` and confirmed the required branch and commit
  conventions.
- Inspected the existing API reference, profile/review routes, and Pydantic
  schemas.

### Reproduction Notes

`docs/API.md` names `POST /profiles` and `POST /reviews`, but neither entry tells
the reader how to construct a request. The source shows that the profile endpoint
uses multipart form fields and an optional file upload, while the review endpoint
uses a JSON body containing a required profile UUID. These details and examples
are missing from the documentation.

### Environment Check

Commands used:

```text
python3 --version
node --version
npm --version
docker --version
docker compose version
make --version
```

Observed environment:

- Python 3.10.12 (project setup requires Python 3.11 or newer)
- Node.js 22.22.2
- npm 10.9.7
- GNU Make 4.3
- Docker and Docker Compose are not installed in the current environment

Because Python and Docker prerequisites are not currently satisfied, the full
`make setup` workflow has not been run yet. This does not block reproducing or
planning the documentation-only issue, but the setup limitation must be resolved
before running the complete project checks required for the final PR.

### Next Steps

- Update `docs/API.md` with accurate request field documentation and examples.
- Review the rendered Markdown for clarity and consistency.
- Re-run the environment setup and required repository checks in an environment
  with Python 3.11+ and Docker.
- Record implementation decisions and test results here.
