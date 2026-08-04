---
name: Submit an Emeritus team (B2B) enrollment lead
description: Submit an enterprise/team enrollment inquiry to Emeritus, switching the lead into the B2B flow and handling its distinct 422 validation envelope.
api: openapi/eruditus-executive-education-leads-openapi.yml
operations:
  - createGenericLead
generated: '2026-08-04'
method: generated
source: https://emeritus-tech.github.io/emeritus-api-docs/api/v1/leads_api/generic_lead_b2b.html
---

# Submit an Emeritus team (B2B) enrollment lead

Use this when someone is enquiring on behalf of a company or team rather than for themselves.
It is the **same endpoint** as the B2C flow — `createGenericLead` — but three extra fields
switch Emeritus onto the B2B path, and the validation errors come back in a different shape.

## Switching into the B2B flow

Three fields are required on top of the B2C set:

- `inquiring_for` — must be the literal string `"team"`. This is the discriminator. Per the
  reference: "When this field value is 'team' we change the flow to use b2b ones. So 'team'
  should be always."
- `company` — the company name.
- `number_of_participans` — an enumerated band, **not** a free count:

  | Value | Means |
  |---|---|
  | `2` | 1-2 participants |
  | `5` | 3-5 participants |
  | `11` | 6-11 participants |
  | `12` | 12+ participants |

  Sending any other integer will fail validation with `"is not included in the list"`.

  Note the field name is spelled `number_of_participans` (missing the second `t`) in the
  request. Emeritus's own 422 error body reports it back as `number_of_participants`. Send the
  spelling from the request table.

## Steps

1. Assemble the B2C required fields — `course_code`, `agree`, `country`, `last_name`, `email`,
   `phone`, `utm_source` — plus the three B2B fields above.
2. Call `createGenericLead`: `POST /api/v1/generic_lead`, `Content-Type: application/json`,
   header `LEAD_WEBHOOK_KEY: <your token>`.
3. Success returns `{"lead": "<uuid>"}`. Store it.

## Error handling — the B2B envelope is different

The B2B flow returns **422**, not 400, and uses a nested envelope:

```json
{
  "title": "Could not save the lead",
  "errors": {
    "number_of_participants": ["required", "is not included in the list"]
  }
}
```

Parse `errors` as a map of field name to a list of messages. Published field keys include
`full_name`, `number_of_participants`, `name` and `company_name` — note these do not all match
the request field names, so map them back to your input by meaning, not by string equality.

`401 {"error": "This is not an authorized request"}` still means the `LEAD_WEBHOOK_KEY` header
is missing or invalid.

## Retry rules

Same as the B2C skill: **no idempotency contract exists**. A retried submission is expected to
create a second lead. Escalate ambiguous failures to a human instead of retrying.

## Test first

Run the flow against staging — `https://staging.emerituss.org/api/v1/generic_lead` — before
sending live traffic to `https://admissions.emeritus.org`.
