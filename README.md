# google-mymaps-icons

An effort to reverse engineer "mapspro" icon sets to correctly render KML/KMZ files exported from [Google My Maps](https://www.google.com/maps/d/) in alternative apps.

To reproduce the issue:

- export a KML/KMZ file containing color-customzied icons, e.g. [this one](https://www.google.com/maps/d/viewer?mid=1JKBG5M29xZP_Gl6QU4tQIeUD6MI&ll=55.5069850610042%2C25.60861000000001&z=15)
- import it into any alternative app

As of September 2024, vast majority of alternative apps will not render icons as set in Google My Maps.

## Original implementation

When creating a map on Google My Maps and adding a marker (defined by "Placemark" tag in KML file) to it, there is an option to change the Placemark's style ("Style" (bucket fill icon) > "More Icons") by either picking from one of the two existing icon sets ("See older/newer icons") or uploading your own custom icon ("Custom icon" button).
[This JS file on Google CDN](https://www.gstatic.com/mapspro/_/js/k=mapspro.mpid.en.htXxPIs22WQ.O/am=ACA/d=1/rs=ABjfnFWhAWDidD1z7xlmxZOQklbHy3456Q/m=mp_base,mp_edit) contains the implementation of both icon sets, including a mapping of icon IDs to icon names, and obfuscated code to generate URLs to customized "Newer" icons. While it might be possible to learn to use it as-is and document the private API, I would much prefer to research and document how this code works to create language-agnostic resource files and clean maintainable implementations for different programming languages I use for my projects.

## Source data

### Icon mappings

- `YR` dictionary maps icon `id`s to icon `name`s and contains 1162 entries including icons from both the "Newer" and "Older" sets, but excludes the "Shapes" category of the "Older" set. Dictionary key is integer icon ID, and value is a string required to build a URL to the icon file on Google CDN. Sprite sheets of the "Newer" and "Older" icon sets together contain 1161 icon sprites. Icon #1899 is the default red pin of Google Maps.
- `ZR` dictionary contains 5 icons for the "Shapes" category of the "Older" icon set. This could be considered a separate icon set because it's rendered unlike both "Newer" and "Older" icons by Google My Maps. Unlike the rest of the "Older" icons, these can be color-tinted.
- `MX` array contains additional metadata for 432 "Newer" icons grouped by category. Each group contains a list of dictionaries with properties of icons from the "Newer" set.

### Color palettes

- `$R` dictionary contains alphanumeric human-readable keys for 147 colors. Purpose of this dictionary is not known.
- `eT` contains palette of 30 colors used for "Shapes" subset of "Older" icons
- `cT` contains palette of 30 colors used for tinting "Newer" icons
- `piA` contains palette of 44 colors that seems to be a superset of `cT`, purpose is not known
- `bT` and `Qia` contain color gradients and their purpose is not known.

The color can be changed for all of the "Newer" set icons, as well as the "Older" set "Shapes" icons, while the rest of the "Older" set colors are hardcoded. Picking a color for an icon whose color can be changed and then switching to an icon whose color cannot be changed does not reset the color of the Placemark and the exported KML file will have that Placemark with a `styleUrl` defining the icon from one set and a color from a palette for a different set. As far as I can see, color palettes are an arbitrary limitation in the UI, not a format or data structure constraint. Icons that can be color tinted should be expected to be of any 32bit color.

To extract the data for use in alternative implementations, open a custom map for editing, then stringify the arrays/dictionaries in Dev Tools, e.g. `JSON.stringify(YR)` (all of the aforementioned variables are global).

## Exported data

### "Newer icons" set

- `styleUrl` referencing an icon from this set contains icon ID and hex value for color, e.g. `#icon-1522-0F9D58`
  - `styleUrl` belongs to this set if color is specified _and_ icon ID can be resolved to file name in `YR` dictionary
  - `styleUrl` _likely_ belongs to this set if file name is resolved to a value with `_4x` suffix
  - `styleUrl` does _NOT_ belong to this set if color is _not_ specified _or_ `ZR` dictionary resolves icon ID to file name
- Icons can be color-tinted
- Icon image files are distributed in two formats:
  - as a [single sprite sheet](https://www.gstatic.com/mapspro/images/stock/extended-icons5.png) (24 columns x 16 full rows + 14 icons on 17th row = 398 icons)
    - `MX` structure contains values to extract specific icons from this spritesheet
      - `Je` key contains 0-based icon index
        - Icons are indexed left to right, top to bottom
      - Order of icons in array defines order in which icons are presented in "Choose an icon" picker of Google My Maps
      - there are more icon definitions in `MX` than sprites in the spritesheet (this is likely due to some icons being referenced by more than one group, e.g. `1601-horse_4x.png` is defined in both "Sports and Recreation" and "Animals" groups)
  - icon-per-file with applied color customization, e.g. `https://mt.googleapis.com/vt/icon/name=icons/onion/SHARED-mymaps-container-bg_4x.png,icons/onion/SHARED-mymaps-container_4x.png,icons/onion/1592-heart_4x.png&highlight=ff000000,FF5252&scale=2.0`
    - Accepted values for `scale` parameter range from `0.2` to `4.0`, but non-integer values result in badly aliased image, so stick with `1.0`, `2.0`, `3.0` and `4.0` values
- `Style` definition in exported KML files will always reference a default white pin icon URL, e.g. `https://www.gstatic.com/mapspro/images/stock/503-wht-blank_maps.png` - this value should be ignored/overridden

### "Older icons" set - "Shapes" category

- `styleUrl` referencing an icon from this subset contains icon ID and hex value for color, e.g. `#icon-961-F4EB37-nodesc`
  - `styleUrl` belongs to this subset if color is specified _and_ icon ID can be resolved to file name in `ZR` dictionary
- Icons can be color-tinted
- Google My Maps doesn't seem to reference these icons in raster format from Google CDN, but rather generates in vector format on the fly
  - However, exported KML files contain `Style` definition with URL to untinted version of the icon image, e.g. `https://www.gstatic.com/mapspro/images/stock/961-wht-square-blank.png` and `IconStyle/color` value

### "Older icons" set - the rest

- `styleUrl` referencing an icon from this subset only contains icon ID, e.g. `#icon-1157`
  - if `styleUrl` contains a color, the icon does _NOT_ belong to this subset
- Icons cannot be customized and their color is predefined by category
- Icon image files are distributed in two formats:
  - as a [single sprite sheet](https://www.gstatic.com/mapspro/images/stock/extended-icons3.png) (24 columns x 31 full rows + 19 icons on 32nd row = 763 icons)
    - individual icons are addressed using `background-position` CSS property
    - the sprite sheet contains icons that are not listed in Placemark customization icon picker
    - there's an alternative version of the spritesheet with ![highlighted versions of the icons](http://www.gstatic.com/mapspro/images/stock/extended-icons-highlight3.png)
  - icon-per-file, e.g. `https://www.gstatic.com/mapspro/images/stock/1157-crisis-infestation-rodents.png`
    - KML files contain a full URL like this in `Style` definition, unlike icons from "Newer" set
