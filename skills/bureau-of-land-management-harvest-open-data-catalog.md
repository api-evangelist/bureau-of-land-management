---
name: Harvest BLM's federal open data catalog
description: >-
  Pull BLM's complete machine-readable dataset inventory in DCAT-US 1.1 (the federal
  Project Open Data format), DCAT-AP 2.1.1 or GeoRSS, and detect what changed since last time.
api: openapi/bureau-of-land-management-gbp-hub-search-openapi.json
base: https://gbp-blm-egis.hub.arcgis.com
operations:
  - OgcItemAggregationController_getAggregations_api/search/v1
  - OgcItemController_getSiteCollectionItems_api/search/v1
---

# Harvest BLM's federal open data catalog

For a bulk harvest, do **not** page the OGC items endpoint. BLM serves the whole inventory as
one document in the format US federal agencies are required to publish.

## The catalog

```
GET https://gbp-blm-egis.hub.arcgis.com/data.json
```

4.8 MB, HTTP 200, `application/json`. It declares its own conformance:

```json
"@context":  "https://project-open-data.cio.gov/v1.1/schema/catalog.jsonld",
"@type":     "dcat:Catalog",
"conformsTo":"https://project-open-data.cio.gov/v1.1/schema"
```

803 datasets as of 2026-09-05. Every entry carries `publisher.name: "Bureau of Land
Management"`, a `landingPage`, a `contactPoint`, `keyword[]`, and — the part that makes this
useful on a schedule — `issued` and `modified` timestamps.

The same document is also served at `/api/feed/dcat-us/1.1.json`.

## Other serializations

- `GET /api/feed/dcat-ap/2.1.1.json` — DCAT-AP 2.1.1 JSON-LD (4.3 MB), for European portal
  tooling.
- `GET /api/feed/rss/2.0` — RSS 2.0 with `georss` and `gml` namespaces.

## Detecting change

BLM publishes no API changelog, no version history and no `Sunset`/`Deprecation` headers.
`dataset[].modified` is the **only** machine-readable change signal on this provider. Store the
maximum `modified` you have seen and diff on the next pull.

Be clear about what this does and does not tell you: it tracks the **data**. It will not warn
you that the contract changed.

## Sizing a slice before you fetch it

`OgcItemAggregationController_getAggregations_api/search/v1` —
`GET /api/search/v1/collections/dataset/aggregations` — returns facet counts without returning
records. Use it to decide whether a filter is worth running.
