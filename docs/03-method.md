# Methods

## Study site

1. download the boundaries for protected areas from [OpenStreetMap](https://www.openstreetmap.org/#map=12/30.7526/76.8920)

* see documentation at https://wiki.openstreetmap.org/wiki/Map_features


```r
library(dplyr) # data manipulation and wrangling
library(osmdata) # osm data download
library(raster)
library(sf)
library(tmap)


x <- opq(bbox = c(76.81, 30.66, 77.01, 30.81)) %>% 
    add_osm_feature(key = "boundary", value = "protected_area", value_exact = FALSE) %>%
    osmdata_sf()
tm_shape(x$osm_polygons) + tm_borders()
# extract protected area boundaries from https://www.openstreetmap.org
#st_layers("D:/spatial-data/osm/swls.osm") # to check layers on the file
#sw_pas <- st_read("D:/spatial-data/osm/swls.osm", layer = "multipolygons") %>% 
  #filter(name %in% c("Sukhna Lake WLS", "Bir Shikargarh WLS", 
                     #"Morni Hills (Khol-Hi-Raitan) WLS")) %>%
  #dplyr::select(name, geometry) %>%
  #mutate(short_name = c("MH", "BS", "SL"))
#st_write(sw_pas, "./data/sw-pas.gpkg", overwrite = T) # write to local directory
#rm(sw_pas)

sw_pas <- st_read("./data/sw-pas.gpkg") # protected area boundary

# download, read and crop the SRTM elevation data
#sw1 <- raster("D:/spatial-data/dem/n30_e076_1arc_v3.tif")
#sw2 <- raster("D:/spatial-data/dem/n30_e077_1arc_v3.tif")
#sw <- merge(sw1, sw2) %>% crop(extent(76.81, 77.01, 30.66, 30.81))
#writeRaster(x = sw, filename = "./data/cropped-elevation.tif", overwrite = T)
#rm(sw1, sw2, sw)

swelev <- raster("./data/cropped-elevation.tif")

# prepare hillshade
#slope <- terrain(swelev, opt= "slope") # extract slope from elevation
#aspect <- terrain(swelev, opt= "aspect") # extract aspect from elevation
#demhs <- hillShade(slope = slope, aspect = aspect, angle = 45, direction = 315) # prepare hill shade
# 45 altitude angle and 315 azimuth angle
#writeRaster(x = demhs, filename = "./data/hs.tif", overwrite = T) # write to local directory
#rm(slope, aspect, demhs) # removed slope and aspect files

hs <- raster("./data/hs.tif") # read the hillshade file

# masked elevation for protected areas
elv_pas <- mask(swelev, mask = sw_pas)

# contour lines
#cl <- rasterToContour(swelev)

# map the files
tm_shape(hs) + 
  tm_raster(palette = "-Greys", n = 100, style = "cont", #alpha = 0.7, 
            legend.show = FALSE) +
  tm_graticules() +
  
  tm_shape(swelev) + tm_raster(palette = terrain.colors(7), alpha = 0.7,
                               n = 7,
                               #style = "quantile",
                               #style = "cont", 
                               #style = "order",
                               legend.show = T,
                               title = "Elevation (m)") +
  
  tm_shape(sw_pas) + tm_fill(col = "white", alpha = 0.3) + 
  tm_borders(col = "white", alpha = 0.8) +
  tm_text(text = "short_name") +
  tm_add_legend(type = "fill", labels = "Protected Areas", col = "white", 
                alpha = 0.3, border.col = "white", border.alpha = 0.8) +
  
  tm_compass(position = c(0.25, 0.03)) +
  tm_scale_bar(position = c(0.4, 0.01), breaks = c(0, 2.5, 5, 7.5, 10)) +
  
  tm_layout(frame.double.line = T, compass.type = "arrow", 
            legend.position = c(0.01, 0.01), legend.bg.color = "white", 
            legend.bg.alpha = 0.5)

#rm(cl, hs, bb)
```


## lets try 3d! something more realistic


```r
library(rayshader)
swelev <- raster("./data/cropped-elevation.tif")

elmat <- swelev %>% raster_to_matrix()

elmat %>% sphere_shade(texture = "desert", zscale = 6, colorintensity = 5) %>%
  #height_shade() %>% 
  add_shadow(lamb_shade(elmat, zscale = 6), 0) %>%
  plot_map()

ovl1 <- sphere_shade(elmat, texture = "desert", zscale = 6, colorintensity = 5)
elmat %>% height_shade() %>% 
  add_overlay(ovl1, alphalayer = 0.5) %>%
  plot_map()

#add_overlay(sphere_shade(elmat, texture = "desert", zscale = 4, colorintensity = 5), alphalayer = 0.5) %>%



elmat %>% 
  sphere_shade(texture = "desert") %>%
  add_water(detect_water(elmat), color = "desert") %>%
  add_shadow(ray_shade(elmat), 0.5) %>%
  add_shadow(ambient_shade(elmat), 0) %>%
  #plot_map()
  plot_3d(elmat, zscale = 10, fov = 0, theta = 135, zoom = 0.75, phi = 45, windowsize = c(1000, 800))
```


