# google-mymaps-icons

An effort to reverse engineer "mapspro" icon sets to correctly render KML/KMZ files exported from [Google My Maps](https://www.google.com/maps/d/) in alternative apps.

## Original implementation

[This JS file on Google CDN](https://www.gstatic.com/mapspro/_/js/k=mapspro.mpid.en.htXxPIs22WQ.O/am=ACA/d=1/rs=ABjfnFWhAWDidD1z7xlmxZOQklbHy3456Q/m=mp_base,mp_edit) contains implementation of both icon sets, including mapping of icon IDs to icon names, and obfuscated code to generate URLs to customized "Newer" icons. While it might be possible to learn to use it as is and document the private API, i would much prefer to research and document how this code works to create language-agnostic resource files and clean maintainable implementations for different programming languages i use for my projects.

`YR` dictionary maps icon `id` to icon `name` and contains 1162 entries including icons from "Newer" and "Older" sets, but excludes "Shapes" category of "Older" set. Dictionary key is integer icon ID, and value is a string required to build URL to icon file on Google CDN. Sprite sheets of "Newer" and "Older" icon sets together contain 1161 icon sprites. The icon #1899 is a default red pin of Google Maps.

`ZR` dictionary contains 5 icons for "Shapes" category of "Older" icon set. This could be considered a separate icon set because it's rendered unlike both "Newer" and "Older" icons by Google My Maps. Unlike the rest of the "Older" icons, these can be color-tinted.

`MX` array contains additional metadata for 432 "Newer" icons grouped by category. Each group contains arrays of dictionaries with properties of icons from "Newer" set.

Color palettes are defined in following variables:
- `$R` dictionary contains alphanumeric human-readable keys for 147 colors. Purpose of this dictionary is not known.
- `eT` contains palette of 30 colors used for "Shapes" subset of "Older" icons
- `cT` contains palette of 30 colors used for tinting "Newer" icons
- `piA` contains palette of 44 colors that seems to be a superset of `cT`, purpose is not known

Google My Maps does not allow picking arbitrary color for placemark icon, and provides two distinct palettes for "Newer" and "Older shapes" icons, but picking a color only present in one palette and then switching to icon from a different set does not reset color of the placemark and exported KML file will have that placemark with styleUrl defining icon from one set and color from a palette for a different set. As far as i can see, color palettes are arbitrary limitation in UI, and not a format or data structure constraint. Icons that can be color tinted should be expected to be of any 32bit color.

Gradients are defined in variables `bT` and `Qia` and their purpose is not known.

To extract the data for use in alternative implementations, open a custom map for editing, then stringify arrays/dictionaries e.g. `JSON.stringify(YR)`.

## "Newer icons" set

- `styleUrl` referencing icon from "newer" icon set contains icon ID and hex value for color, e.g. `#icon-1522-0F9D58`
	- `styleUrl` belongs to "Newer icons" if color is specified _and_ icon ID can be resolved to file name in `YR` dictionary
	- `styleUrl` _likely_ belongs to "Newer icons" if file name is resolved to a value with "\_4x" suffix
	- `styleUrl` does _NOT_ belong to "Newer icons" if color is not specified _or_ `ZR` dictionary resolves icon ID to file name
- Icons can be color-tinted
- Icon image files are distributed in two formats:
	- as a [single sprite sheet](https://www.gstatic.com/mapspro/images/stock/extended-icons5.png) (24 columns x 16 full rows + 14 icons on 17th row = 398 icons)
		- `MX` structure contains values to extract specific icons from this spritesheet
			- `Je` key contains 0-based icon index
				- Icons are indexed left to right, top to bottom
			- Order of icons in array defines order in which icons are presented in "Choose an icon" picker of Google My Maps
			- there are more icon definitions in `MX` than sprites in the spritesheet
				- this is likely due to some icons being referenced by more than one group, e.g. `1601-horse_4x.png` is defined in both "Sports and Recreation" and "Animals" groups
	- icon-per-file with applied color customization, e.g. `https://mt.googleapis.com/vt/icon/name=icons/onion/SHARED-mymaps-container-bg_4x.png,icons/onion/SHARED-mymaps-container_4x.png,icons/onion/1592-heart_4x.png&highlight=ff000000,FF5252&scale=2.0`
		- Accepted values for `scale` parameter range from 0.2 to 4.0, but non-integer values result in badly aliased image, so stick with 1.0, 2.0, 3.0 and 4.0 values
- `Style` definition in exported KML files will always reference a default white pin icon URL, e.g. `https://www.gstatic.com/mapspro/images/stock/503-wht-blank_maps.png` - this value should be ignored/overridden

## "Older shapes" set

- `styleUrl` referencing icon from "older shapes" icon set contains icon ID and hex value for color, e.g. `#icon-961-F4EB37-nodesc`
	- `styleUrl` belongs to "older shapes" if color is specified _and_ icon ID can be resolved to file name in `ZR` dictionary
- Icons can be color-tinted
- Google My Maps doesn't seem to reference these icons in raster format from Google CDN, but rather generates in vector format on the fly
	- However, exported KML files contain `Style` definition with URL to untinted version of the icon image, e.g. `https://www.gstatic.com/mapspro/images/stock/961-wht-square-blank.png` and `IconStyle/color` value

## "Older icons" set

- `styleUrl` referencing icon from "older" icon set only contains icon ID, e.g. `#icon-1157`
	- `styleUrl` belongs to "Older icons" if color is not specified
	- `styleUrl` does _NOT_ belong to "Older icons" if color is specified
- Icons cannot be customized and their color is predefined by icon set
- Icon image files are distributed in two formats:
	- as a [single sprite sheet](https://www.gstatic.com/mapspro/images/stock/extended-icons3.png) (24 columns x 31 full rows + 19 icons on 32nd row = 763 icons) 
		- individual icons are addressed using `background-position` CSS property
		- the sprite sheet contains icons that are not listed in Placemark customization icon picker
		- there's alternative version of the spritesheet with "highlighted" versions of the icons: `http://www.gstatic.com/mapspro/images/stock/extended-icons-highlight3.png`
	- icon-per-file, e.g. `https://www.gstatic.com/mapspro/images/stock/1157-crisis-infestation-rodents.png`
		- KML files contain a full URL like this in `Style` definition, unlike icons from "Newer" set
