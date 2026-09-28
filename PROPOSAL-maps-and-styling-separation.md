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

A new `rel: "map"` link — shaped like `web-map-links`' existing `rel=wmts`/`rel=xyz` pattern rather
than a new inline object, since a "map" is itself a fetchable/renderable resource (an actual endpoint
or preview), matching OGC API - Maps' resource-oriented model. It reuses the `render` cross-reference
attribute already established by `web-map-links` integration, rather than duplicating data-source or
styling info onto the link itself.

Raster case — data (assets, implicit via the `render` cross-reference) + style (a `renders` entry) +
map-specific constraints:

```jsonc
{
  "rel": "map",
  "href": "https://tiles.example.com/{z}/{x}/{y}.png",
  "type": "image/png",
  "title": "True color map",
  "render": "true-color",
  "tilematrixsets": { "WebMercatorQuad": [8, 22] }
}
```

Vector case — data + style (a `rel=stylesheet` link) + constraints:

```jsonc
{
  "rel": "map",
  "href": "https://tiles.example.com/vector/{z}/{x}/{y}.pbf",
  "type": "application/vnd.mapbox-vector-tile",
  "title": "Roads map",
  "stylesheet": "https://example.com/styles/roads.json",
  "minzoom": 0,
  "maxzoom": 14
}
```

What this would resolve:

- `render` stays scoped to pixel transformation (rescale, colormap, resampling, expression, `bidx`);
  `minmax_zoom`/`tilematrixsets` are deprecated there in favor of living on the `rel=map` link.
- @vincentsarago's precision need (TMS-scoped zoom) and @m-mohr's simplicity need (bare `minzoom`/`maxzoom`)
  can both be offered on the same link.
- Reuses an existing cross-reference pattern instead of inventing a new one.

**Open question**: whether a map link should offer `tilematrixsets` and bare `minzoom`/`maxzoom` 
side by side, or just one of them.

## What changes in `render`

- extension description updated to reflect limited scope to pixel transformation (rescale, colormap, resampling, expression, `bidx`)
- `minmax_zoom` and the `tilematrixsets` field proposed in PR #27 would be deprecated/removed from
  `render`'s own scope once this lands.
- PR #27 would be refocused (or closed) once the group agrees on where zoom/tiling constraints belong.

## Open questions / next steps

- New `stac-extensions/maps` repository
- How does the `rel=map` link's `tilematrixsets` relate to
  [tiled-assets](https://github.com/stac-extensions/tiled-assets)' existing `tiles:tile_matrix_set_links`
  — reuse, align, or intentionally keep separate?
- TMS-scoped zoom vs. bare `minzoom`/`maxzoom` vs. both, per the open question above.
