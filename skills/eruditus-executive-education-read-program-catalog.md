---
name: Read the Emeritus program catalog
description: Retrieve Emeritus programs, partner schools and landing page templates via the Programs API, and work within its unpublished response contract.
api: openapi/eruditus-executive-education-programs-openapi.yml
operations:
  - listPrograms
  - listSchools
  - listLandingPageTemplates
generated: '2026-08-04'
method: generated
source: https://emeritus-tech.github.io/emeritus-api-docs/api/v1/programs_api/programs.html
---

# Read the Emeritus program catalog

Use this to look up what Emeritus offers — the programs themselves, the partner universities
and business schools that award them, and the landing page templates used to market them.

## Authentication

The Programs API uses a **different header from the Leads API**. Send:

```
HTTP-EE-RESOURCES-API-KEY: <your token>
```

Do not reuse the Leads API's `LEAD_WEBHOOK_KEY` header here — it will return
`401 {"error": "This is not an authorized request"}`.

Tokens are issued manually by Emeritus (https://emeritus.org/connect-with-us/).

## Operations

All three are `GET`, all under `/api/v1/programs_api/`:

- `listPrograms` — `GET /api/v1/programs_api/programs`
- `listSchools` — `GET /api/v1/programs_api/schools`
- `listLandingPageTemplates` — `GET /api/v1/programs_api/landing_page_templates`

Base URL: `https://admissions.emeritus.org` (production) or `https://staging.emerituss.org`
(staging).

## Read this before you build against it

**Emeritus publishes no request parameters and no response schemas for these three
resources.** Their reference pages still carry the documentation template's placeholder
content — a single parameter row reading "param / string / This is a param description" and a
response body of `{"example":"response"}`.

Practical consequences:

- Do not assume query parameters for filtering, sorting or pagination. None are documented.
- Do not hard-code a response shape. Treat the body as untyped JSON, inspect it at runtime,
  and fail soft on missing fields.
- Do not assume the collections are paginated or that they are not. Handle both: check for a
  large flat array and for any envelope/cursor key you actually observe.
- Ask Emeritus for the real schemas before shipping anything that depends on their structure.

The endpoints themselves are real and live — each returns `401` (not `404`) without a key,
confirmed 2026-08-04.

## Linking to a lead

`course_code` is the field that ties an enrollment lead to a program (see the lead-submission
skills). Use `listPrograms` to resolve a program to its code before calling
`createGenericLead`, rather than hard-coding codes.

## Errors

`401 {"error": "This is not an authorized request"}` — the `HTTP-EE-RESOURCES-API-KEY` header
is missing or invalid. No other error responses are published for these operations.

No rate-limit policy or headers are published; back off conservatively.
