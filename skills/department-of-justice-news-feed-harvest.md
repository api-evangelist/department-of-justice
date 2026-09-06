---
name: doj-news-feed-harvest
description: Harvest DOJ press releases and blog entries from the DOJ News API, with correct paging, field selection and epoch-second date filtering.
api: DOJ News API
base_url: https://www.justice.gov/api/v1
operations:
  - GET /api/v1/press_releases.json
  - GET /api/v1/press_releases/[uuid].json
  - GET /api/v1/blog_entries.json
  - GET /api/v1/blog_entries/[uuid].json
generated: '2026-09-06'
method: generated
source: >-
  Grounded in the resource list, arguments and worked examples published at
  https://www.justice.gov/developer/api-documentation/api_v1, and confirmed against live
  anonymous responses on 2026-09-06. This API publishes no machine-readable contract; the four
  resources above are the four the documentation declares.
---

# Harvest the DOJ news feed

The Office of Public Affairs publishes press releases and blog entries as JSON. No key, no signup.

## The four resources

| Call | Returns |
|---|---|
| `GET /api/v1/press_releases.json` | list of press releases |
| `GET /api/v1/press_releases/[uuid].json` | one press release |
| `GET /api/v1/blog_entries.json` | list of blog entries |
| `GET /api/v1/blog_entries/[uuid].json` | one blog entry |

Note what is **not** there: there is no `speeches.json`. The DOJ Developer Resources page still
describes speeches as part of this API, but the reference page does not list the resource and
`GET /api/v1/speeches.json` returned `404` on 2026-09-06. Do not build against it.

## Steps

1. **Select fields.** Omitting `fields` returns every field, and `body` alone can run to several
   kilobytes per record. Ask for what you need:
   `?fields=title,url,uuid,date,component`. The unique identifier field is `uuid`.

2. **Page.** `page` is **zero-based**. `pagesize` defaults to 20 and is hard-capped at 50 — a
   larger request is silently clamped to 50 and still returns `200`, so never infer "no more
   results" from getting fewer rows than you asked for. Read
   `metadata.resultset.count` for the true total and walk `page` until you have consumed it.

3. **Sort.** `sort` takes a field name (`changed`, `created`, `date`) and `direction` takes `ASC`
   or `DESC`. For an incremental harvest, sort by `created` descending and stop when you reach a
   record you already hold.

4. **Filter.** Filters go through a bracketed array: `?parameters[title]="Chicago"`,
   `?parameters[date]=1231243200`. Dates are **Unix epoch seconds**, not ISO strings.

5. **Unwrap the envelope.** Every response is
   `{"metadata":{"responseInfo":{"status":200},"resultset":{"count","pagesize","page"},"executionTime":…},"results":[…]}`.
   `metadata.responseInfo.status` repeats the HTTP status inside the body; trust the HTTP status.

## Rules

- Stay at or below **4 requests per second**. The documentation warns that above that you "will
  experience degraded performance and may be blocked entirely". There is no `Retry-After`, no
  `X-RateLimit-*` header and no documented status code on exhaustion — you get no runtime signal,
  so pace yourself deliberately.
- A wrong path under `/api/v1/` returns a 56 KB Drupal **HTML** page with a `404` status, not a
  JSON error. Branch on `Content-Type`, not on the body.
- Read-only. Nothing to make idempotent, nothing to reverse; retries are always safe.
