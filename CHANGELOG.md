# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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

[Unreleased]: <https://github.com/stac-extensions/render/compare/v2.0.0...HEAD>
[2.0.0]: <https://github.com/stac-extensions/render/compare/v2.0.0...v1.0.0>
[1.0.0]: <https://github.com/stac-extensions/render/compare/v1.0.0...HEAD>
