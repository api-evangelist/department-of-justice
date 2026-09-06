---
name: bjs-crime-statistics
description: Pull Bureau of Justice Statistics NCVS victimization and NIBRS national estimate datasets from api.ojp.gov without silently truncating the result.
api: BJS NCVS and NIBRS National Estimates
base_url: https://api.ojp.gov/bjsdataset/v1
operations:
  - GET /bjsdataset/v1/{resource}.json
  - GET /bjsdataset/v1/{resource}.csv
generated: '2026-09-06'
method: generated
source: >-
  Grounded in the access instructions, worked example URLs and record-count warnings published at
  https://bjs.ojp.gov/national-crime-victimization-survey-ncvs-api and
  https://bjs.ojp.gov/national-incident-based-reporting-system-nibrs-national-estimates-api, and
  confirmed against live anonymous responses on 2026-09-06. BJS publishes no machine-readable
  contract for these datasets.
---

# Pull BJS crime statistics

BJS serves NCVS and NIBRS estimate datasets as Socrata-style resources. No key, no signup.

## The shape of a call

BJS documents the URL in three parts:

```
https://api.ojp.gov/bjsdataset/v1/  gcuy-rt5g.csv  ?$limit=200000
└────────── path ────────────────┘  └─ resource ─┘  └── filter ──┘
```

The **resource** is a four-four alphanumeric dataset id plus a format extension. Format is chosen
by extension — `.json` or `.csv` — not by an `Accept` header.

## Steps

1. **Get the resource id from the BJS page, not from a guess.** Each dataset's id is published as
   a link on its API page: the NCVS page lists the four NCVS Select datasets (personal
   population, personal victimization, household population, household property victimization) and
   the NIBRS page lists the incident-, offense- and victim-level estimate datasets. There is no
   index endpoint that will enumerate them for you.

2. **Always set `$limit`.** This is the one thing that will silently give you wrong answers. The
   default is **1,000 records**, and BJS warns in its own documentation that every NCVS dataset
   holds more than 1,000 rows *for any single year*. A query without `$limit` returns a `200` and
   a truncated dataset with no indication it was cut. BJS's own example uses `$limit=200000`;
   the per-dataset record counts are published as spreadsheets linked from each API page.

3. **Read the codebook before interpreting a column.** NCVS and NIBRS columns are coded
   (`ager`, `hincome1`, `race_ethnicity` …). BJS publishes person-level and household-level NCVS
   codebooks as PDFs and a NIBRS codebook with supplementary documentation. Do not infer a code's
   meaning from its name.

4. **Handle the bare array.** Unlike the DOJ News API, BJS returns a top-level JSON array with no
   envelope, no count and no paging links.

## Rules

- **The endpoints moved in July 2025.** Anything pointing at `bjs.ojp.gov/api/...` is dead —
  it returned `404` on 2026-09-06. The live path is `api.ojp.gov/bjsdataset/v1/`. There was no
  deprecation header, no versioned alias and no dated changelog; the only notice is a sentence on
  the documentation page. Re-read that page before trusting a stored base URL.
- `api.ojp.gov` is a WSO2 API Manager gateway. Its errors are **XML** Synapse faults
  (`<am:fault><am:code>404</am:code>…`), not JSON. A `HEAD` returns `405`.
- No published rate limit and no rate-limit headers. Be conservative.
- NCVS note: the 2024 statistics in the API come from the *legacy* instrument. BJS ran a
  split-sample redesign in 2024 and will publish redesigned-instrument estimates separately, so do
  not concatenate the two without reading the methodology note on the NCVS page.
