---
name: Query BLM's national and state ArcGIS services directly
description: >-
  Work against BLM's own ArcGIS Server estate at gis.blm.gov — 13 REST instances, 785 services,
  all anonymous — to enumerate services, read layer schemas and run spatial or attribute
  queries. Covers the HTTP-200-on-error trap that breaks naive clients.
api: openapi/bureau-of-land-management-arcgis-service-inventory.json
base: https://gis.blm.gov
operations: []
---

# Query BLM's national and state ArcGIS services directly

BLM publishes **no OpenAPI** for this surface. The contract is the ArcGIS REST resource
documents themselves (`?f=json`) plus, for 42 of the 46 national services, an OGC WMS 1.3.0
`GetCapabilities`. Seven of those are saved verbatim in `openapi/` in this repo.

## The instances

One national instance and twelve state instances, all ArcGIS Server 11.5, all
`isTokenBasedSecurity: false`:

```
https://gis.blm.gov/arcgis/rest/services      national     46 services
https://gis.blm.gov/akarcgis/rest/services    Alaska       59
https://gis.blm.gov/azarcgis/rest/services    Arizona      24
https://gis.blm.gov/caarcgis/rest/services    California   40
https://gis.blm.gov/coarcgis/rest/services    Colorado    100
https://gis.blm.gov/esarcgis/rest/services    Eastern States 2
https://gis.blm.gov/idarcgis/rest/services    Idaho       146
https://gis.blm.gov/mtarcgis/rest/services    Montana      23
https://gis.blm.gov/nvarcgis/rest/services    Nevada       50
https://gis.blm.gov/nmarcgis/rest/services    New Mexico   41
https://gis.blm.gov/orarcgis/rest/services    Oregon       85
https://gis.blm.gov/utarcgis/rest/services    Utah         51
https://gis.blm.gov/wyarcgis/rest/services    Wyoming     118
```

The full inventory, folder by folder, is in
`openapi/bureau-of-land-management-arcgis-service-inventory.json`.

## Steps

1. **Always send `f=json`.** The default representation is HTML. Without `f=json` you get a web
   page, not data.
2. List folders: `GET /<instance>/rest/services?f=json` → `folders[]`.
3. List services in a folder: `GET /<instance>/rest/services/{folder}?f=json` → `services[]`
   with `name` and `type`.
4. Describe a service: `GET /<instance>/rest/services/{folder}/{name}/{type}?f=json` →
   `layers[]`, `capabilities`, `supportedExtensions`.
5. Describe a layer: append `/{layerIndex}?f=json` → `fields[]`, `geometryType`,
   `maxRecordCount`, `extent`, `spatialReference`.
6. Query: `/{layerIndex}/query?where=1%3D1&outFields=*&f=geojson`. Add `returnCountOnly=true`
   first to size the result before you pull it.

## THE TRAP: failure arrives as HTTP 200

`gis.blm.gov` returns **200** for "Service not found", "Layer not found" and query failures. The
real outcome is in the body:

```json
{"error": {"code": 404, "message": "Service not found", "details": []}}
```

Branch on the presence of a top-level `error` key, never on the status line. Same for WMS: a
failure is an XML body whose root element is `ServiceExceptionReport`, also served with 200.

## OGC WMS

For any service whose `supportedExtensions` contains `WMSServer`:

```
https://gis.blm.gov/arcgis/services/{folder}/{name}/MapServer/WMSServer?service=WMS&request=GetCapabilities
```

Note the path segment is `/arcgis/services/`, **not** `/arcgis/rest/services/`. Only the
requests listed in the capabilities document are accepted; anything else returns
`RequestNotAllowed`.

No WFS is published on this surface — 0 of 46 national services advertise `WFSServer`.

## Naming

`BLM_Natl_*` national, `BLM_<ST>_*` state, `BOC_*` boundary reference, `*_Cached` pre-rendered
tiles. Service and folder names are case-sensitive.
