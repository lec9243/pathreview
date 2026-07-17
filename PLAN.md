# Issue #89 Solution Plan

## Issue

- GitHub issue: https://github.com/ascherj/pathreview/issues/89
- Title: API reference doc is missing the `POST /profiles` request body schema
- Tier: Tier 1 / good first issue
- Working branch: `docs/89-api-request-schemas`

## Problem Summary

The PathReview API reference lists the `POST /profiles` and `POST /reviews`
endpoints but does not document their request bodies. API consumers therefore
cannot tell which fields each endpoint accepts, which fields are required, or
what valid requests look like without reading the application source or opening
the generated Swagger documentation.

## Reproduction

1. Open `docs/API.md`.
2. Find the Profiles and Reviews sections.
3. Observe that `POST /profiles` and `POST /reviews` are listed only as one-line
   endpoint summaries.
4. Compare the documentation with `api/routes/profiles.py`,
   `api/schemas/profile.py`, `api/routes/reviews.py`, and
   `api/schemas/review.py`.
5. Confirm that the accepted request fields, content types, constraints, and
   example values are absent from the API reference.

## Proposed Changes

Update `docs/API.md` with request documentation for both endpoints:

### `POST /profiles`

- Explain that the endpoint accepts `multipart/form-data`.
- Document the optional `github_username`, `portfolio_url`, and `resume_file`
  fields.
- Include relevant length and file-type constraints from the route and schema.
- Add a representative `curl` example that shows text fields and a resume file.

### `POST /reviews`

- Explain that the endpoint accepts an `application/json` body.
- Document the required `profile_id` UUID field.
- Add representative JSON and `curl` examples.

The new sections will follow the concise Markdown style already used in the API
reference and will not change application behavior.

## Files in Scope

- `docs/API.md`

The route and schema files will be used as sources of truth but will not be
modified.

## Validation Plan

1. Compare every documented field and content type against the FastAPI route
   signatures and Pydantic schemas.
2. Check that all Markdown headings, tables, JSON, and shell examples render
   correctly.
3. Search `docs/API.md` to confirm that both POST endpoints have request-body
   documentation and example values.
4. Run the repository's required `make check` and `make test-unit` commands once
   the required Python and Docker environment is available, recording any
   pre-existing failures separately.

## Risks and Mitigations

- **Documentation drift:** Use the current route implementations and schemas as
  the source of truth rather than guessing field behavior.
- **Incorrect content type:** Clearly distinguish the multipart profile request
  from the JSON review request.
- **Misleading example IDs:** Use visibly illustrative UUID values and describe
  them as examples.

## Out of Scope

- Changing API behavior or validation rules.
- Adding or modifying response schemas.
- Redesigning the rest of the API reference.
- Adding request schemas for unrelated endpoints.

