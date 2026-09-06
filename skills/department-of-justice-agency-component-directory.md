---
name: agency-component-directory
description: Build a directory of the federal agency components that receive FOIA requests, including each component's submission address, FOIA officers, public liaisons and request form.
api: National FOIA Portal JSON:API
base_url: https://api.foia.gov/api
operations:
  - GET /agency_components
  - GET /agency_components/{entity}
  - GET /agency_components/{entity}/request_form
  - GET /agency
generated: '2026-09-06'
method: generated
source: >-
  Grounded in openapi/department-of-justice-foia-api-swagger.json and the worked curl examples
  published at https://www.foia.gov/developer/.
---

# Build the FOIA agency component directory

An "agency component" is the office inside a federal agency that actually fulfils a FOIA request —
the Office of Information Policy inside DOJ, for instance. The portal publishes the whole
government-wide directory.

## Steps

1. **List the components, sparsely.** The `node--agency_component` definition carries 29
   attributes and 7 relationship members. Ask for what you need:

   ```
   GET /agency_components?fields[agency_component]=title,abbreviation,description,email,submission_address
   ```

   Attributes the contract declares as selectable include `title`, `status`, `abbreviation`,
   `moderation_state`, `description`, `email` and `submission_address`.

2. **Join to the parent agency in one call.** JSON:API `include` saves you an N+1:

   ```
   GET /agency_components?include=agency&fields[agency]=name,abbreviation&fields[agency_component]=title,abbreviation,agency
   ```

   The parent agency arrives in the top-level `included` array; resolve it through
   `data[].relationships.agency.data.id`.

3. **Pull one component in full** with `GET /agency_components/{entity}` where `{entity}` is the
   component UUID. The FOIA.gov developer page publishes the Office of Information Policy's UUID
   as its worked example.

4. **Fetch the submission form** with `GET /agency_components/{entity}/request_form`. Components
   differ in what they require, so read the form rather than assuming a common shape.

5. **Follow the people relationships** — `foia_officers`, `public_liaisons`, `service_centers`,
   `paper_receiver` — when you need a contact rather than an address. Each is a JSON:API resource
   linkage; the contract does not state cardinality, so handle both a single object and an array.

## Rules

- `403` means the key is missing or rejected at the gateway, before the portal ever sees the
  request. `404` on `/agency_components/{entity}` means the component UUID is wrong.
- Read-only. Nothing here submits a FOIA request; the public contract is the directory side only.
- Respect the 1,000-requests-per-hour key limit. Prefer one `include` call over many follow-ups.
