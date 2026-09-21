# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- **Breaking:** `bidx` is now an array of arrays, with one inner array per position in `assets`, so a band
  selection can be tied unambiguously to a specific asset when there is more than one. The previous flat array
  of numbers (e.g. `"bidx": [1]`) is no longer valid; migrate it to `"bidx": [[1]]` for a single asset, or to
  one inner array per asset (use `[]` for an asset that needs no sub-selection)
  ([#18](https://github.com/stac-extensions/render/issues/18))
- `bidx` index selectors are now documented and validated as 1-based integers (`minimum: 1`), matching the
  GDAL/rio-tiler/titiler convention ([#18](https://github.com/stac-extensions/render/issues/18))
- **BREAKING** (to be released as a new major version): `colormap_name` MUST now be a standard
  [matplotlib colormap](https://matplotlib.org/stable/users/explain/colors/colormaps.html) name (e.g. `viridis`,
  `YlGn`), to give the field a well-known, renderer-agnostic naming convention instead of an arbitrary free-form
  string ([#14](https://github.com/stac-extensions/render/issues/14))

### Added

- Extend `bidx` to accept a band name (matching the `name` of a Band Object declared on the asset via
  `eo:bands`, `raster:bands`, or the STAC 1.1+ common `bands` construct) as an alternative to a 1-based index
  ([#18](https://github.com/stac-extensions/render/issues/18))
- Add a Planet example (`examples/item-planet.json`), demonstrating `bidx` name-based selectors on a genuine
  single multi-band asset (PlanetScope's 8-band analytic product), using STAC 1.1.0's common `bands` construct

## [2.1.0] - 2026-09-22

### Added

- Adds `asset_as_band` property in `render` object ([#12](https://github.com/stac-extensions/render/issues/12))

### Fixed

- Document that `expression` accepts `string`, `object`, or `array`, matching the schema ([#8](https://github.com/stac-extensions/render/issues/8))
- Clarify that `nodata` overrides any nodata value already defined on the referenced assets for this specific
  render, rather than duplicating it ([#15](https://github.com/stac-extensions/render/issues/15))
- Update broken titiler `colormap`/`color_formula` doc links and replace the dead `api.cogeo.xyz` demo host.
  Rework the NDVI example around the still-live Sentinel-2 item, since the Landsat-8 example's source imagery
  was removed from the public `landsat-pds` bucket ([#13](https://github.com/stac-extensions/render/issues/13))

## [2.0.0] - 2024-11-19

### Added

- Adds render cross reference in links ([#2](https://github.com/stac-extensions/render/issues/2))

### Changed

- Place 'renders' object in Item properties [#4](https://github.com/stac-extensions/render/issues/4)

### Deprecated

### Removed

### Fixed

## [1.0.0] - 2023-12-19

Initial release

[Unreleased]: <https://github.com/stac-extensions/render/compare/v2.1.0...HEAD>
[2.1.0]: <https://github.com/stac-extensions/render/compare/v2.0.0...v2.1.0>
[2.0.0]: <https://github.com/stac-extensions/render/compare/v2.0.0...v1.0.0>
[1.0.0]: <https://github.com/stac-extensions/render/compare/v1.0.0...HEAD>
