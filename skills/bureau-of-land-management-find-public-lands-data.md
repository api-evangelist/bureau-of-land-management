---
name: Find and download BLM public lands data
description: >-
  Search the Bureau of Land Management's geospatial catalog for a dataset (grazing allotments,
  surface management agency, PLSS, recreation sites, wild horse and burro herd areas) and get
  from a search result to actual features you can query. No credential required.
api: openapi/bureau-of-land-management-gbp-hub-search-openapi.json
base: https://gbp-blm-egis.hub.arcgis.com
operations:
  - CollectionController_getSiteCollections_api/search/v1
  - QueryableController_getSiteCollectionQueryables_api/search/v1
  - OgcItemController_getSiteCollectionItems_api/search/v1
  - OgcItemController_getSiteCollectionItemById_api/search/v1
  - GeoserviceController_featureLayers_api/search/v1
  - GeoserviceController_featureLayerQuery_api/search/v1
---

# Find and download BLM public lands data

BLM's catalog is an OGC API - Records service. It is anonymous — **do not look for an API key,
there isn't one**, and there is no plan, quota or sign-up.

## 1. Pick a collection

`CollectionController_getSiteCollections_api/search/v1` — `GET /api/search/v1/collections`

Three exist: `dataset` (the 803 data items), `document`, `site`. You almost always want `dataset`.

## 2. Learn what you may filter on

`QueryableController_getSiteCollectionQueryables_api/search/v1` —
`GET /api/search/v1/collections/dataset/queryables`

Do this before writing a `filter`. Guessing a field name costs you a 400.

## 3. Search

`OgcItemController_getSiteCollectionItems_api/search/v1` —
`GET /api/search/v1/collections/dataset/items`

Useful parameters: `q` (free text), `bbox` (CRS84 `minx,miny,maxx,maxy`), `filter` (CQL2),
`type`, `tags`, `sortBy`, `openData`, `limit`, `startindex`.

- `limit` is capped at **20000**. Exceeding it returns HTTP 400 with `message` as an **array**,
  not a string — handle both shapes.
- Page with `startindex`, not by raising `limit`.
- Response is GeoJSON: `features[]`, `numberMatched`, `numberReturned`.

If you only need counts, call `OgcItemAggregationController_getAggregations_api/search/v1`
instead — it returns facets without returning records, which is much cheaper for sizing a
question.

## 4. Read the record

`OgcItemController_getSiteCollectionItemById_api/search/v1` —
`GET /api/search/v1/collections/dataset/items/{itemId}`

`itemId` is a 32-character hex ArcGIS item id. A wrong one returns HTTP 404 with
`Cannot find item with recordId ... in collection Data`.

## 5. Get to the actual features

`GeoserviceController_featureLayers_api/search/v1` then
`GeoserviceController_featureLayerQuery_api/search/v1` —
`GET /api/search/v1/rest/services/collections/{collectionId}/geoservice/FeatureServer/layers`
then `.../FeatureServer/{layerIndex}/query`

Query parameters follow ArcGIS conventions: `where` (SQL-92; `1=1` for everything),
`outFields`, `returnGeometry`, `resultOffset`, `resultRecordCount`, `f=geojson`.

**Check `maxRecordCount` on the layer document before paging.** It varies per layer — 2000 on
the layers checked, but it is not a constant across BLM's services.

## Error rules that will bite you

- A malformed `where` returns `Unable to complete operation.` — the useful part is `details[]`.
- On the **gis.blm.gov** surfaces (see the other skill) failures come back as **HTTP 200** with
  an `error` object in the body. On this Hub surface the status code is honest.

## Licensing

BLM geospatial data is US federal public-domain public record. Credit BLM; see
https://www.doi.gov/disclaimer.
