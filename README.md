---
title: "Rgee: Hello world"
---

## 1) Load libraries
```r
library(rgee)
library(sf)
library(cptcity)
library(mapview)
library(leaflet)
```

Initialize earthengine
```r
ee_Initialize() 
```

## 2) Map visualization with ee_map
Load color palete 
```r
cpt_pal = cpt(pal = "mpl_inferno")
```
Load SRTM satellite image and create a visualization on leaflet.
```r
image <- ee$Image('CGIAR/SRTM90_V4')
ee_map(image,
       vizparams = list(min = 0, max = 5000, palette= cpt_pal),
       zoom_start = 2,
       objname = 'SRTM90_V4')
```

## 3) Sf map
Loading a shapefile file with sf library and color palete
```r
nc = st_read(system.file("shape/nc.shp", package="sf"))
cpt_pal <- cpt(pal = "wkp_schwarzwald_wiki_schwarzwald_cont")
nc_ee <- nc %>%
  st_transform(4326) %>%
  sf_as_ee(check_ring_dir = TRUE)
```
Clipping a image with a sf object
```r
clip_image <- image$clip(nc_ee)
```
Visualizing the clip image with the st object
```r
mapview(nc, alpha.regions = 0, legend = FALSE) +
  ee_map(clip_image,
         vizparams = list(min = 0, max = 1000, palette= cpt_pal),
         zoom_start = 6,
         objname = 'SRTM',
         quiet = TRUE)
```

## 4) Extract values from Earth Engine to sf
```r
nc_dem  <- ee_extract(clip_image,nc_ee,id = "FIPSNO") %>%
  `names<-`(c("FIPSNO",'SRTM_DEM')) %>%
  merge(nc, .)
plot(nc_dem['SRTM_DEM'])
```
