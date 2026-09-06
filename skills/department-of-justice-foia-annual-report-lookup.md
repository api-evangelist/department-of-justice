---
name: foia-annual-report-lookup
description: Retrieve a federal agency's annual FOIA report from the National FOIA Portal, either as JSON:API data or as the NIEM-conforming XML exchange.
api: National FOIA Portal JSON:API
base_url: https://api.foia.gov/api
operations:
  - GET /agency
  - GET /agency/{entity}
  - GET /annual_foia_report/fiscal_years
  - GET /annual_foia_report
  - GET /annual-report-xml/{agency}/{year}
generated: '2026-09-06'
method: generated
source: >-
  Grounded in openapi/department-of-justice-foia-api-swagger.json (verbatim first-party contract
  from github.com/usdoj/foia.gov) and https://www.foia.gov/developer/. Every path above is
  declared in that contract.
---

# Look up an agency's annual FOIA report

Every federal agency files an annual FOIA report. DOJ's Office of Information Policy collects them
and serves them two ways from the same host: as JSON:API resources, and as the NIEM XML exchange
that agencies actually file.

## Before you start

You need a free api.data.gov key. Sign up at <https://www.foia.gov/developer/#api-key-signup>.
Send it as `X-API-Key`. Without it every call returns
`403 {"error":{"code":"API_KEY_MISSING"}}` — the body is the gateway's, not JSON:API's, so do not
try to parse a 403 as a JSON:API error document.

Your key is limited to 1,000 requests per hour across all api.data.gov agencies. Read
`X-RateLimit-Remaining` on each response rather than counting your own calls.

## Steps

1. **Find the agency.** `GET /agency` lists the agency taxonomy. Narrow it with JSON:API sparse
   fields so you are not pulling every attribute:
   `GET /agency?fields[agency]=name,abbreviation`.
   You want the `abbreviation` (for example `DOJ`) and the resource `id` (a UUID).

2. **Find which years exist.** `GET /annual_foia_report/fiscal_years` returns the fiscal years that
   have reports. Do not assume the current year is present — reports lag the fiscal year they
   cover.

3. **Choose your representation.**
   - For structured data you can filter and join, `GET /annual_foia_report` returns JSON:API
     documents. Be warned: the `node--annual_foia_report_data` definition carries 281 attributes
     and 35 relationship members, one per statutory report section (`field_foia_requests_va`,
     `field_proc_req_viic1`, `field_admin_app_via` …). Always request `fields[]` and `include`
     explicitly or you will pull megabytes per agency-year.
   - For the filed exchange itself, `GET /annual-report-xml/{agency}/{year}` — for example
     `/annual-report-xml/DOJ/2021`. `{agency}` is the abbreviation from step 1, not the UUID.
     The response is XML conforming to the FOIA Annual Report XML schema, a NIEM IEPD. If you
     already have a NIEM toolchain, this is the path that needs no bespoke mapping.

4. **Page anything that returns a list.** The portal uses JSON:API `page[limit]` and
   `page[offset]` and returns `links.next`. Follow `links.next` until it is absent.

## Rules

- Everything here is a `GET`. There is no write operation on this API, so there is nothing to make
  idempotent and nothing to reverse. Retrying a failed call is always safe.
- The contract declares `info.version` as the literal string "Versioning not supported". There is
  no version to pin and no deprecation header to watch. Treat endpoint stability as unguaranteed
  and fail loudly on an unexpected shape rather than coercing it.
- Errors: `400` syntax error, `403` missing or rejected key, `404` not found. No `5xx` is declared.
  None of these are RFC 9457 problem documents. See
  `errors/department-of-justice-problem-types.yml`.
