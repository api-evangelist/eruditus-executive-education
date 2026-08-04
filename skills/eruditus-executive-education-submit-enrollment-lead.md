---
name: Submit an Emeritus enrollment lead
description: Submit a B2C (individual learner) enrollment inquiry to Emeritus for a specific program, and handle the documented failure modes.
api: openapi/eruditus-executive-education-leads-openapi.yml
operations:
  - createGenericLead
generated: '2026-08-04'
method: generated
source: https://emeritus-tech.github.io/emeritus-api-docs/api/v1/leads_api/generic_lead.html
---

# Submit an Emeritus enrollment lead (B2C)

Use this when an individual working professional wants information about, or wants to enroll
in, a specific Emeritus program.

## Before you start

- You need a partner token issued by Emeritus. It is **not** self-service — Emeritus issues it
  on request via https://emeritus.org/connect-with-us/.
- You need the program's `course_code` (for example `KLG-DMS`). It is how the lead is attached
  to a program. Do not use `batch_name` — the reference marks it DEPRECATED.
- Test against staging first: `https://staging.emerituss.org`. Production is
  `https://admissions.emeritus.org`.

## Steps

1. **Collect the required fields.** `createGenericLead` requires all of: `course_code`,
   `agree` (`1` = consent given, `0` = not given), `country` (2-letter ISO code), `last_name`,
   `email` (must be a valid address), `phone`, and `utm_source` (the vendor name in one word).
   Optional: `first_name`, `work_experience`, `job_title`, `utm_content`, `utm_placement`.

2. **Do not submit without consent.** `agree` is required and semantically meaningful — send
   `1` only when the person has actually agreed. Never default it.

3. **Call `createGenericLead`.** `POST /api/v1/generic_lead` with
   `Content-Type: application/json` and the header `LEAD_WEBHOOK_KEY: <your token>`.
   The token goes in that custom header, not in `Authorization`.

4. **Read the 200 body.** Success returns `{"lead": "<uuid>"}`. Store that identifier — it is
   the only handle Emeritus gives you for the submission.

## Error handling

Emeritus returns two different error envelopes; branch on status, not on shape.

| Status | Body | What to do |
|---|---|---|
| `401` | `{"error": "This is not an authorized request"}` | The `LEAD_WEBHOOK_KEY` header is missing or wrong. Do not retry with the same key. |
| `400` | `{"error": "Missing batch"}` | The `course_code` did not resolve to a program. Re-check the code with Emeritus. |
| `400` | `{"error": "Missing custom form"}` | The program has no lead form configured. This is a provider-side configuration issue — escalate, do not retry. |
| `400` | `{"error": "Email required\nEmail must be an email"}` | Validate the email address before resending. |

## Retry rules — read this

**There is no idempotency contract.** Emeritus documents no idempotency key, and the endpoint
carries no retry-safety guarantee (see `conventions/eruditus-executive-education-conventions.yml`).
A blind retry after a timeout or a 5xx is expected to create a **second lead** for the same
person. On an ambiguous failure, escalate to a human rather than retrying automatically.

No rate-limit policy or `RateLimit-*` / `Retry-After` headers are published, so back off
conservatively on your own schedule.

## Handling personal data

Every field on this operation is personal data — name, email, phone, employer role, country.
Send only what the person supplied, only with their consent, and do not log the payload in the
clear. Emeritus's privacy notice is at https://emeritus.org/privacy-notice/.
