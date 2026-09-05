---
name: 51offer-gpa-calculation
description: Convert a Chinese transcript into the GPA scales UK/US/Australian universities ask for, using 51offer's public GPA calculator API.
api: 51offer Horizon Site API
base_url: https://www.51offer.com
operations:
  - queryRefundConstants
  - queryGpaBai
  - queryGpaLevel
  - queryGpaWu
generated: '2026-09-05'
method: generated
source: openapi/51offer-horizon-site-openapi.yml (converted from https://www.51offer.com/api-docs)
---

# GPA calculation

51offer's GPA calculator is the one part of this API that answers **anonymously** — verified
2026-09-05, `GET https://www.51offer.com/ngGpaCalc/constants` returned HTTP 200 and
`application/json`. Everything below is grounded in operations that exist in the contract; no
operation, parameter or algorithm is invented here.

## 1. Read the algorithm dictionary first

    GET /ngGpaCalc/constants          # operationId: queryRefundConstants

Returns the envelope with `data.gpaCalc[]` — the algorithms 51offer supports, each with a `code`
and a Chinese `value`. Observed codes include `arithmetic` (算术平均分), `weighting` (加权平均分),
`wes` (WES algorithm), `standard` (standard 4.0) and `improvement4` / `improvement4_2` (improved
4.0 variants). **Do not hard-code this list** — read it, because the dictionary is the provider's
and can change under you.

Note the operationId: it is `queryRefundConstants`, not `queryGpaConstants`. That is what 51offer
published, and calling anything else will 404.

## 2. Pick the endpoint that matches the source grading scale

| Source transcript scale | Operation | Path |
|---|---|---|
| Percentage (百分制) | `queryGpaBai` | `POST /ngGpaCalc/gpabai` |
| Letter / band (等级制) | `queryGpaLevel` | `POST /ngGpaCalc/gpalevel` |
| Five-point (五分制) | `queryGpaWu` | `POST /ngGpaCalc/gpawu` |

All three take a JSON **array** of `GpaCalc` items as the request body (the Swagger 1.2 source
names the body parameter `gpaCalcList` and flags it `allowMultiple`), and all three return
`HttpResult«Map«string,object»»`.

## 3. Read the envelope, not the HTTP status

Every response is HTTP 200. The outcome is in the body:

```json
{ "message": "成功", "code": 200, "data": { }, "header": null, "page": null,
  "apiInfo": { "operationTime": "2026-09-06 04:55:09", "version": null, "hostName": null } }
```

Branch on `code`, never on the HTTP status. A `code` other than 200 is the failure signal, and
`message` is Chinese prose intended for a human.

## Rules that apply to every call

- **No idempotency.** There is no `Idempotency-Key` and no request key anywhere in this API
  (`conventions/51offer-conventions.yml`). GPA calculation is a pure read, so a retry is safe here —
  that is a property of this operation, not of the API.
- **No rate limits are published or signalled** (`rate-limits/51offer-rate-limits.yml`): no
  `RateLimit-*` and no `Retry-After` on any observed response. Back off on your own budget rather
  than waiting for a signal that does not exist.
- **No error-code reference exists.** Log `code` and `message` verbatim; there is nothing to map
  them against.
