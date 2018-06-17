# antanym-demo
Map demonstrations of the [antanym R package](https://github.com/AustralianAntarcticDataCentre/antanym).


- A [simple leaflet app](leaflet.html) using Mercator projection and clustered markers for place names.


Source code:

```{r eval = FALSE}
library(antanym)
library(leaflet)
g <- an_read()

## find single name per feature, preferring United Kingdom
##  names where available, and only rows with valid locations
g <- an_preferred(g, origin = "United Kingdom")
g <- g[!is.na(g$longitude) & !is.na(g$latitude), ]

## replace NAs with empty strings in narrative
g$narrative[is.na(g$narrative)] <- ""

## formatted popup HTML
popup <- sprintf("<h1>%s</h1><p><strong>Country of origin:</strong> %s<br />
  <strong>Longitude:</strong> %g<br /><strong>Latitude:</strong> %g<br />
  <a href=\"https://data.aad.gov.au/aadc/gaz/scar/display_name.cfm?gaz_id=%d\">
    Link to SCAR gazetteer</a></p>", g$place_name, g$origin,
    g$longitude, g$latitude, g$gaz_id)

m <- leaflet() %>%
  addProviderTiles("Esri.WorldImagery") %>%
  addMarkers(lng = g$longitude, lat = g$latitude, group = "placenames",
    clusterOptions = markerClusterOptions(), popup = popup,
    label = g$place_name)
```



- Leaflet using [polar stereographic projection](leafletps.html).


Source code (note that the leaflet package here must be the rstudio version; use `devtools::install_github("rstudio/leaflet")`):

```{r eval = FALSE}
startZoom <- 1

crsAntartica <-  leafletCRS(
  crsClass = 'L.Proj.CRS',
  code = 'EPSG:3031',
  proj4def = '+proj=stere +lat_0=-90 +lat_ts=-71 +lon_0=0 +k=1 +x_0=0 +y_0=0 +ellps=WGS84 +datum=WGS84 +units=m +no_defs',
  resolutions = c(8192, 4096, 2048, 1024, 512, 256),
  origin = c(-4194304, 4194304),
  bounds =  list( c(-4194304, -4194304), c(4194304, 4194304) )
)

mps <- leaflet(options = leafletOptions(crs = crsAntartica, minZoom = 0, worldCopyJump = FALSE)) %>%
    setView(0, -90, startZoom) %>%
    addCircleMarkers(lng = g$longitude, lat = g$latitude, group = "placenames",
                     popup = popup, label = g$place_name,
                     fillOpacity = 0.5, radius = 8, stroke = FALSE, color = "#000",
					 labelOptions = labelOptions(textOnly = FALSE)) %>%
    addWMSTiles(baseUrl = "https://maps.environments.aq/mapcache/antarc/?",
                layers = "antarc_ramp_bath_shade_mask",
                options = WMSTileOptions(format = "image/png", transparent = TRUE),
                attribution = "Background imagery courtesy <a href='http://www.environments.aq/'>environments.aq</a>") %>%
    addGraticule()
```
