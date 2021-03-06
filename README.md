# **Mapsheet.js**

**Mapsheet.js** makes an interactive map out of a Google Spreadsheet. It depends on [Tabletop.js](http://github.com/jsoma/tabletop) and whatever map provider you're using. It is also dead simple.

*Optional:* Mapsheet supports [Handlebars](http://handlebarsjs.com) for info window templates.

**NOTE: Mapsheet is still under heavy development (if you can call it that).**

## So how do I do this?

First, you make a published Google Spreadsheet. You can find out how to do this in the [Tabletop docs](http://github.com/jsoma/tabletop). The only columns you actually need are *latitude* and *longitude*. We're also friendly enough to allow *lat*, *lng*, *long*, any probably anything else you can think of.

    var map = Mapsheet({
      key: "https://docs.google.com/spreadsheet/pub?hl=en_US&hl=en_US&key=0AmYzu_s7QHsmdDNZUzRlYldnWTZCLXdrMXlYQzVxSFE&output=html",
			element: "map"
		});

and as long as you have a `<div id="map"></div>` somewhere you'll be good to go.

(Of course you'll want to do this after the page has loaded, mind you, with `onload` or `document.ready` or whatnot.)

# Details 

## The Moving Parts

### Mapsheet itself

When you initialize Mapsheet you have plenty of options to pick through. `key` and `element` are the only two required parameters, though!

`key` is the URL to the *published* Google Spreadsheet you're dealing with.

`element` is the id of the element that's going to become the map. You can also pass in an element.

`fields` is an array of columns to display in the info window if you don't feel like using a template. Check the examples!

`popupContent` is a function that returns content for the info window popup. It's passed the Tabletop model for the given row. It overrides both `fields` and `popupTemplate`.

`popupTemplate` is the id of a [Handlebars](http://handlebarsjs.com) template that will be used for the info window that pops up when you click the marker on the map. You can also provider the compiled template instead.
	
`provider` is the provider of the map service. I detail the providers a bit more below, but your options are

		Mapsheet.Providers.Leaflet
		Mapsheet.Providers.Google
		Mapsheet.Providers.MapBox
		Mapsheet.Providers.MapQuest

`markerOptions` are passed through to the marker, and are used for things like specifying marker shadows and other very-specific-to-the-provider details.

`map` is the map, if you feel like rendering it without using a Mapsheet provider.

`titleColumn` is the column that you'd like to use as the title of the point (e.g., the name of the place). This is only important if using `fields`, or if you want hover effects for MapQuest or Google.

### Mapsheet.Marker

`.model` is the Tabletop model of the row the marker is associated with

`.longitude()` is the longitude, pulled from a column named `lng`, `long` or `longitude`

`.latitude()` is the latitude, pulled from a column named `lat` or `latitude`

`.title()` gets the title of the marker - this column is set through `titleColumn`

`.coords()` gives you the marker's `[ lat, lng ]`

`.get(name)` returns the value of the given column. Not that it matters to you, but: Multi-word and capitalized things, like 'Web URL' are fixed up in an attempt to successfully read from tabletop, i.e. converted into lowercase with spaces removed (`Web URL` turns into `weburl`).

`.content()` uses `popupContent`, `popupTemplate`, or `titleColumn` and `fields` to create content for the info window popup

`.isValid()` returns true if latitude() and longitude() both return valid floats

### Mapsheet.Provider.Whatever

Must respond to `.initialize(id)` and `.drawPoints(pointsArray)`.

Their constructor sets up the options, `initialize` draws the map, and `drawPoints` puts the points on it and auto-fits the zoom and bounds.

## Providers

[Google Maps](https://developers.google.com/maps/) as `Mapsheet.Providers.Google` (default)

[Leaflet](http://leafletjs.com) (using MapQuest tiles by default, but supports CloudMade, etc) as `Mapsheet.Providers.Leaflet` (be sure to include the CSS)

[MapBox](http://mapbox.com) as `Mapsheet.Providers.MapBox`

[MapQuest](http://developer.mapquest.com/web/products/open) as `Mapsheet.Providers.MapQuest`

# Customizing Your Map

## Styling info windows

All of the info boxes live inside of a `<div class='mapsheet-popup'></div>`, so feel free to apply styles to `.mapsheet-popup h3`, etc

## Customizing the map itself (roads, water, etc)

You'll need to pick a provider that allows map customization

[CloudMade](http://www.cloudmade.com) has a billion styles over at their [map style editor](http://maps.cloudmade.com/editor), you'll just have to check out the `cloudmade.html` example. You need an API key, but it ain't tough beyond that.

[MapBox](http://www.mapbox.com) could also work for you, especially if you have a lot of stuff going on.

## Custom Markers

I think it's *really stupid* that, generally speaking, providers make it so tough to have custom-colored markers. So I tried to make it easy!

Custom markers are supported for Google Maps, Leaflet, and MapBox (to varying degrees).

### Possibilities

**Custom color:** Add a `hex color` column in your spreadsheet to customize the color of each marker.

**Custom icon:** If you want to an specify icon for each point, add an `icon url` column in your spreadsheet. You'll probably want to make it the full path, i.e. `http://blahblah.com/images/icons/blah.png`

**Custom default icon:** If want every point to have the same marker, just not the default, you can pass a default icon URL through `markerOptions`, typically as `iconUrl`

**Custom icon + shadows:** Check the `examples/leaflet-with-marker-customization.html` example, it's complicated.

### Provider support

**Google Maps:** custom colors, custom icons, custom default icon

**MapBox:** custom icons

**Leaflet:** custom icons, custom icons + shadows, custom default icon

### General notes on using colored/custom icons

If you're unsure of what hex colors are, pop on over to [http://www.colorpicker.com](http://www.colorpicker.com) or [Wikipedia](http://en.wikipedia.org/wiki/Web_colors#X11_color_names) - they're just strings of numbers/letters like FF36A2 that end up as colors.*

When you customize your icons using Leaflet, you'll probably want to pass some `markerOptions` as well to make sure everything lines up nicely. `markerOptions: { iconSize: [32, 37], iconAnchor: [16, 37] }` works for the icons from the [Map Icons Collection](http://mapicons.nicolasmollet.com). [Read more on the Leaflet site](http://leafletjs.com/examples/custom-icons.html).

### Using other Leaflet tile providers

Leaflet defaults to serving tiles from MapQuest, because it's free, and doesn't require an API key. If you'd like to use [CloudMade](http://www.cloudmade.com) or something instead, you'll need to make some changes. You'll need to pass in a mapOptions along the lines of the following

		mapOptions: {
			subdomains: 'abc',
			tilePath: 'http://{s}.tile.cloudmade.com/{apikey}/{styleId}/256/{z}/{x}/{y}.png',
			styleId: 1930,
			apikey: 'Your CloudMade API Key',
			attribution: 'An appropriate credit line without MapQuest'
		}

And then you should be all set!

# Examples

There are a lot of examples in `/examples`, check them out!

If you're looking for the least complicated one, check out `fields.html`. It isn't too customizable but it gets the job done. `leaflet-with-marker-customization.html` is the zaniest of the bunch.

# Etc

## Credits

Hi, I'm [Jonathan Soma](http://twitter.com/dangerscarf). I run the [Brooklyn Brainery](http://brooklynbrainery.com), and make all sorts of nonsense.

## Todo

Fix up the docs to actually be nice and pleasant

Don't show popups if there's no content in the popup

Replace the sample map with some quality, dedicated sample maps

Callbacks callbacks everywhere

Notes about styling your infoboxes