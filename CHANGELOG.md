# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Add `minmax_resolution`, expressing the visible resolution range in `gsd` units (meters/pixel) instead of
  ambiguous zoom levels, and explain the zoom-to-resolution calculation for implementations that still need
  zoom levels ([#16](https://github.com/stac-extensions/render/issues/16))

### Deprecated

- `minmax_zoom` is deprecated in favor of `minmax_resolution`, since "zoom level" depends on a mapping library's
  tile size convention (e.g. 256px vs 512px) and is not portable across libraries ([#16](https://github.com/stac-extensions/render/issues/16))

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
