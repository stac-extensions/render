# Rendering Extension Specification

- **Title:** Rendering
- **Identifier:** <https://stac-extensions.github.io/render/v1.0.0/schema.json>
- **Field Name Prefix:** renders
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @emmanuelmathot @abarciauskas-bgse @smohiudd

This document explains the Rendering Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.

Rendering extension aims at providings consumers with the possible rendering of an item or a collection (e.g. on a online map)

- Examples:
  - [Landsat-8 example](examples/item-landsat8.json): Shows the basic usage of the extension in a landsat-8 STAC Item
  - [Sentinel-2 example](examples/item-sentinel2.json): Shows the basic usage of the extension in a Sentinel-2 STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [ ] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name | Type                                         | Description                                                                               |
| ---------- | -------------------------------------------- | ----------------------------------------------------------------------------------------- |
| renders    | Map<string, [Render Object](#render-object)> | **REQUIRED**. Dictionary of rendering objects that can be viewed, each with a unique key. |

### Render Object

| Field Name    | Type      | Description                                                                                                                                                              |
| ------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| assets        | \[string] | **REQUIRED**. Array of asset keys [referencing the assets](#assets-reference) that are used to make the rendering                                                        |
| title         | string    | Optional title of the rendering                                                                                                                                          |
| rescale       | \[float]  | 2 dimensions array of delimited Min,Max range per band. If not provided, the data will not be rescaled.                                                                  |
| nodata        | float, string     | Nodata value to use for the referenced assets.                                                                                                                           |
| colormap_name | string    | Color map identifier that must be applied for a raster band                                                                                                              |
| colormap      | object    | [Color map JSON definition](https://developmentseed.org/titiler/advanced/rendering/#custom-colormaps) that must be applied for a raster band                             |
| color_formula | string    | [Color formula](https://developmentseed.org/titiler/advanced/rendering/#color-formula) that must be applied for a raster band                                            |
| resampling    | string    | Resampling algorithm to apply to the referenced assets. See [GDAL resampling algorithm](https://gdal.org/programs/gdalwarp.html#cmdoption-gdalwarp-r) for some examples. |
| expression    | string    | Band arithmetic formula to apply to the referenced assets.                                                                                                               |
| minmax_zoom   | \[int]    | Zoom levels range applicable for the visualization                                                                                                                       |

The `render` object is open ended, so additional fields can be provided according to the needs of the rendering application.

## Assets reference

The `assets` field is a list of asset keys referencing the assets that are used to make the rendering.
The assets MUST be local assets defined in the same item.

> \[!NOTE]
> When it is intended to use assets from external items or specific bands in an asset,
> it is recommended to define a [virtual asset](https://github.com/stac-extensions/virtual-assets)
> and then reference its key in the `assets` field. See the [NDVI example](#normalized-difference-vegetation-index-ndvi-example).

## Positioning

The positioning of the source assets is defined by their position in the `assets` array.
Typically, in the case of the composition of a RGB image, the first pointer would be the red band,
the second the green band and the third the blue band.

```json
"assets": [ "red", "green", "blue" ]
```

## Rescaling

A rescaling of the values from the source asset(s) to the destination asset can be defined using the `vrt:rescale` field.
It is specified as a 2 dimensions array of delimited Min,Max range per band.

```json
"rescale": [
  [0, 10000], // band 1
  [0, 10000], // band 2
  [0, 10000]  // band 3
]
```

A prescaling can also be performed according to the `offset` and `scale` fields value of the
[raster](https://github.com/stac-extensions/raster) extension.

## Dynamic tile servers integration

The render objects are designed to be used by dynamic tile servers to produce RGB tiles from a STAC Item.
They are generic enough to be used by any dynamic tile server. In the following sections, some tilers integration are described.

### Titiler

[titiler](https://github.com/developmentseed/titiler) offers a native
[STAC reader](https://github.com/developmentseed/titiler/blob/main/docs/src/endpoints/stac.md).

The following table describes the titiler query parameters that could be used and the corresponding extension fields.

Either the client building titiler url can use the information in the virtual asset to build the query parameters
or the dynamic tile server could use the information in the virtual asset to build the query parameters
by simply specifying the `url` and `assets` query parameters.

| Query key       | field                                  | Description                                                                                                                         |
| --------------- | -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `url`           | `href` in item self link               | STAC Item URL                                                                                                                       |
| `assets`        | `assets`                               | Assets keys to use for the tile rendering defined in the `assets` field                                                             |
| `rescale`       | `rescale`                              | Delimited Min,Max bounds defined in `rescale` field.                                                                                |
| `expression`    | `expression`                           | Band arithmetic formula as defined in field `expression`                                                                            |
| `nodata`        | `nodata` or `nodata` `raster:bands`    | If not provided in `nodata` field, the nodata value can be read from the `nodata` field of the corresponding `raster:bands` object. |
| `unscale`       | `scale` and `offset` in `raster:bands` | Scale and Offset value defined in `scale` and `offset` fields of the corresponding `raster:bands` item                              |
| `colormap_name` | `colormap_name`                        | Color map name defined in `colormap` field of the `asset`                                                                           |
| `colormap`      | `colormap`                             | Color map JSON definition as defined in `colormap` object of the `asset` (overrides `colormap_name` if present )                    |
| `color_formula` | `color_formula`                        | Color formula as defined in `color_formula` field of the `asset`                                                                    |
| `resampling`    | `resampling`                           | Resampling method to use when reprojecting the raster.                                                                              |
| `bidx`    | `bidx`                           | Dataset band indexes                                                                            |

#### Shortwave Infra-red visual thermal signature example

From the [Sentinel-2 item](https://github.com/stac-extensions/virtual-assets/blob/main/examples/item-sentinel2.json):

```json
"renders":{
  "sir":
  {
    "title": "Shortwave Infra-red",
    "assets": [ "swir22", "nir2",  "red" ],
    "rescale": [[0,5000],[0,7000],[0,9000]],
    "resampling": "nearest"
  }
}
```

| Query key | value                                               | Example value                                                                                |
| --------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| url       | STAC Item URL                                       | `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json` |
| assets    | Assets keys defined in the `assets` fields          | `B12,B8A,B04`                                                                                |
| rescale   | Delimited Min,Max bounds defined in `rescale` field | `0,5000,0,7000,0,9000`                                                                       |

URL: [`https://api.cogeo.xyz/stac/crop/14.869,37.682,15.113,37.862/256x256.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json&assets=B12,B8A,B04&resampling_method=average&rescale=0,5000,0,7000,0,9000&return_mask=true`](https://api.cogeo.xyz/stac/crop/14.869,37.682,15.113,37.862/256x256.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-sentinel2.json&assets=B12,B8A,B04&resampling_method=average&rescale=0,5000,0,7000,0,9000&return_mask=true)

**Result**: Lava thermal signature of Mount Etna eruption (February 2021)

![etna](images/etna.png)

#### Normalized Difference Vegetation Index (NDVI) example

From the [Landsat-8 example](examples/item-landsat8.json) \[[article](https://www.usgs.gov/core-science-systems/nli/landsat/landsat-normalized-difference-vegetation-index?qt-science_support_page_related_con=0#qt-science_support_page_related_con)]:
This example uses the [virtual assets](https://github.com/stac-extensions/virtual-assets) to define the NDVI asset first because in this use case,
the NDVI asset could also be downloaded as a standalone asset.

```json
"assets":{
  "ndvi": 
  {
    "roles": [ "virtual", "data", "index" ],
    "type": "image/vnd.stac.geotiff; cloud-optimized=true",
    "href": "https://raw.githubusercontent.com/stac-extensions/render/main/examples/item-landsat8.json#/assets/ndvi",
    "vrt:hrefs": [
      { "key": "B04", "href": "https://raw.githubusercontent.com/stac-extensions/render/main/examples/item-landsat8.json#/assets/B04"}, 
      { "key": "B05", "href": "https://raw.githubusercontent.com/stac-extensions/render/main/examples/item-landsat8.json#/assets/B05"}],
    "title": "Normalized Difference Vegetation Index",
    "vrt:algorithm": "band_arithmetic",
    "vrt:algorithm_opts": {
      "expression": "(B05–B04)/(B05+B04)",
      "rescale": [[-1,1]]
    },
  }
},
"renders":{
  "ndvi":
  {
    "title": "Normalized Difference Vegetation Index",
    "assets": [ "ndvi" ],
    "resampling": "average",
    "colormap_name": "ylgn"
  }
}
```

If this case, the parameters to titiler must be extracted from both the virtual asset definition and the render object.

| Query key         | value                                                                            | Example value                                                                               |
| ----------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| url               | STAC Item URL                                                                    | `https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-landsat8.json` |
| expression        | Band math formula as defined in field `vrt:algorithm`                            | `(B5–B4)/(B5+B4)`                                                                           |
| rescale           | Delimited Min,Max bounds defined in `rescale` field                              | `-1,1`                                                                                      |
| colormap          | Color map JSON definition as defined in `colormap_name`                          | `ylgn`                                                                                      |
| resampling_method | Resampling method to use when reprojecting the raster as defined in `resampling` | `average`                                                                                   |

URL:

[`https://api.cogeo.xyz/stac/preview.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-landsat8.json&expression=(B5–B4)/(B5+B4)&max_size=512&width=512&resampling_method=average&rescale=-1,1&color_map=ylgn&return_mask=true`](https://api.cogeo.xyz/stac/preview.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-landsat8.json&expression=(B5–B4)/(B5+B4)&max_size=512&width=512&resampling_method=average&rescale=-1,1&color_map=ylgn&return_mask=true)

Result:  Landsat Surface Reflectance Normalized Difference Vegetation Index (NDVI) path 44 row 33.

![sacramento](https://api.cogeo.xyz/stac/preview.png?url=https://raw.githubusercontent.com/stac-extensions/raster/main/examples/item-landsat8.json&expression=(B5–B4)/(B5+B4)&max_size=512&width=512&resampling_method=average&rescale=-1,1&color_map=ylgn&return_mask=true)

Obviously, the same rendering can be applied to local source assets without using the virtual asset.

```json
"renders":{
  "ndvi":
  {
    "title": "Normalized Difference Vegetation Index",
    "assets": [ "B05", "B04" ],
    "resampling": "average",
    "colormap_name": "ylgn",
    "expression": "(B05–B04)/(B05+B04)",
    "rescale": [[-1,1]]
  }
}
```

## Links

It is highly suggested to have a web map link in the `links` section of the STAC Item as described in the
[Web Map Link extension](https://github.com/stac-extensions/web-map-links) to allow application to
find the tiling endpoint of the dynamic tile server.

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
