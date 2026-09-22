# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Add `bands`, a name-based alternative to `bidx` that references a `name` from the asset's `eo:bands`,
  `raster:bands`, or the STAC 1.1+ common `bands` construct, resolving the ambiguity over whether a band
  reference must correspond to STAC band metadata. Structured as one array per position in `assets`, so a
  band selection can be tied unambiguously to a specific asset when there is more than one
  ([#18](https://github.com/stac-extensions/render/issues/18))
- Add a Planet example (`examples/item-planet.json`), demonstrating `bands` on a genuine single multi-band
  asset (PlanetScope's 8-band analytic product), using STAC 1.1.0's common `bands` construct

### Deprecated

- `bidx` is deprecated in favor of `bands`, and is now documented as a 1-based index (matching the
  GDAL/rio-tiler/titiler convention) with no required correspondence to any STAC band metadata
  ([#18](https://github.com/stac-extensions/render/issues/18))

### Fixed

- Document that `expression` accepts `string`, `object`, or `array`, matching the schema ([#8](https://github.com/stac-extensions/render/issues/8))

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

[Unreleased]: <https://github.com/stac-extensions/render/compare/v2.0.0...HEAD>
[2.0.0]: <https://github.com/stac-extensions/render/compare/v2.0.0...v1.0.0>
[1.0.0]: <https://github.com/stac-extensions/render/compare/v1.0.0...HEAD>
