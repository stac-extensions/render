# Proposal: separate "map" concerns from `render`

Status: **discussion draft**.

## Context

`render` didn't start from "how do I make a map." It started from wanting to describe a *composite*
derived from existing assets ([composite](https://github.com/stac-extensions/composite)), which
overlapped confusingly with [virtual-assets](https://github.com/stac-extensions/virtual-assets)'
cross-reference/repositioning model. What we settled on was narrowing `render`'s scope specifically
to *rendering*: how to combine assets (or bands within assets) with a small set of usual display
parameters (rescale, colormap, resampling, expression, band selection).

[PR #27](https://github.com/stac-extensions/render/pull/27) (adding `minmax_resolution`/
`tilematrixsets`, fixing [#16](https://github.com/stac-extensions/render/issues/16)) surfaced that
`minmax_zoom` and its proposed replacements don't fit cleanly into that scope:

- A bare zoom integer is ambiguous across mapping libraries (different default tile sizes).
- A bare resolution number is meaningless without a CRS/bbox to interpret it against.
- A bare TileMatrixSet id is resolvable without GDAL/PROJ for well-known ids (OGC publishes static
  JSON for these), and a custom CRS can be self-hosted and referenced by URI instead of needing a
  central registry.

That last point reframed the debate: `minmax_zoom`/`tilematrixsets` are web-mapping/tiling concepts,
not properties of *how pixel values are transformed*, which is what `render` was actually scoped to.
The same `render` object can inform a GIS desktop template (QGIS layer styling, a print layout) where
"zoom level" isn't a meaningful concept at all — so tiling concerns don't belong inside `render` even
though they're clearly still needed *somewhere*.

That question moved to a maintainer discussion, which converged on a three-way structural split:

- **Data Sources** — `assets`, [web-map-links](https://github.com/stac-extensions/web-map-links)
- **Styling** — `render` (raster), a `rel=stylesheet` link (vector + external raster styles, proposed
  in [#21](https://github.com/stac-extensions/render/issues/21) /
  [PR #28](https://github.com/stac-extensions/render/pull/28), based on OGC API - Styles)
- **Maps** — composes a data source with an optional style into a viewable map, carrying map-specific
  constraints such as zoom/tile-matrix-set limits

This isn't a novel split: **OGC API - Maps** (draft:
[20-058](https://docs.ogc.org/DRAFTS/20-058.html), overview:
[ogcapi-workshop.ogc.org](https://ogcapi-workshop.ogc.org/api-deep-dive/maps/)) already models exactly
this — `/collections/{id}/map` (data + default style) and
`/collections/{id}/styles/{styleId}/map` (data + a specific style) are both "map" resources, distinct
from **OGC API - Styles**' ([20-009](https://docs.ogc.org/DRAFTS/20-009.html)) independent `/styles`
catalog. [tiled-assets](https://github.com/stac-extensions/tiled-assets) (Proposal maturity) is the
closest existing STAC prior art for TMS-scoped zoom limits and should be referenced/aligned with here,
not duplicated. No `stac-extensions/maps` (or similarly scoped) extension currently exists.

## Proposed mechanism

**web-map-links** — where TMS/zoom information belongs, since it's a property of a specific *service*:

- Existing rel types (`xyz`, `wmts`, `wms`, `pmtiles`, `3d-tiles`, `tilejson`) are unchanged. `render`
  as a link attribute isn't one of them either — it's defined by `render`'s own integration with
  web-map-links, not by web-map-links itself.
- There's already real prior art for per-TMS zoom constraints inside web-map-links today, closer than
  `tiled-assets`: WMTS's REST encoding lets a `uriTemplate` variable like `{TileMatrix}` carry a JSON
  Schema constraint (minimum/maximum) via `variables`, and the built-in `{TileMatrixSet}`/`{TileMatrix}`
  placeholders are already tile-matrix-set aware without any custom field. Extending that same
  `variables` mechanism to OGC API – Tiles (and to `xyz`, which today has no way to constrain `{z}` at
  all) may be a smaller change than a new field, and worth checking before adding one.
- `maps` and `tiles` are added for OGC API – Maps and OGC API – Tiles, since both are formally distinct
  protocols from what `xyz`/`wmts` already cover.

Illustrative shape, field names TBD — using the existing `variables`/JSON-Schema-constraint mechanism
rather than a new field:

```jsonc
"links": [
  {
    "rel": "tiles",
    "type": "image/png",
    "uriTemplate": "https://tiles.example.com/tiles/{tileMatrixSetId}/{z}/{x}/{y}.png",
    "render": "true-color",
    "variables": {
      "tileMatrixSetId": { "const": "WebMercatorQuad" },
      "z": { "type": "integer", "minimum": 8, "maximum": 22 }
    }
  }
]
```

**maps** — a composition document:

- A projection
- A default extent
- An ordered list of layers, bottom to top
- Each layer has exactly one data source (an asset, a web-map-link, or a `render` reference) and
  optionally one style (a `rel=stylesheet` link or a `render` reference)

A single data source with a single style and its own zoom limits is just a map with one layer — the
composition also covers what a single link structurally can't: a COG rendered client-side with no tile
server at all, GeoParquet paired with a stylesheet, several layers shown together (see
[#22](https://github.com/stac-extensions/render/issues/22)), or a basemap layer underneath the data.
`source` is only needed when there's no `style.render` to imply it — a `render` entry already has its
own `assets`, so restating them on the layer would be redundant (and could disagree with it). Illustrative
shape, field names TBD:

```jsonc
"maps": {
  "default": {
    "projection": "EPSG:3857",
    "extent": [-122.52, 37.70, -122.35, 37.83],
    "layers": [
      { "source": { "web_map_link": "basemap" } },
      { "style": { "render": "true-color" } },
      { "source": { "web_map_link": "roads-tiles" }, "style": { "stylesheet": "https://example.com/styles/roads.json" } }
    ]
  }
}
```

**render** — carries no visibility/zoom/resolution field at all, including for the client-side-rendering
case. That case is already covered above: a COG rendered client-side with no tile server is just a
`maps` document with one layer and no explicit TMS, so a dedicated escape hatch on `render` would
duplicate what `maps` already does, and reopen the exact scope creep this split exists to close.
Keeping `render` at zero map-adjacent fields is a stronger, simpler boundary than a narrow exception
for "just this one case."

## What changes in `render`

- Extension description updated to reflect the narrower scope: pixel transformation only (rescale,
  colormap, resampling, expression, `bidx`).
- `minmax_zoom` and the `tilematrixsets` field proposed in PR #27 are removed from `render`'s scope
  entirely — TMS/zoom for tiled services moves to `web-map-links`, and the client-side/no-tile-server
  case moves to `maps` (see above). `render` keeps no map-adjacent field of any kind.
- PR #27 would be closed or refocused once the group agrees on this.

## Open questions / next steps

- Detailed **maps v1** and **web-map-links v2** proposals, written up before any schema here.
- New `stac-extensions/maps` repository: scope, ownership.
- How do web-map-links' per-TMS zoom ranges relate to
  [tiled-assets](https://github.com/stac-extensions/tiled-assets)' existing `tiles:tile_matrix_set_links`
  — reuse, align, or intentionally keep separate?
