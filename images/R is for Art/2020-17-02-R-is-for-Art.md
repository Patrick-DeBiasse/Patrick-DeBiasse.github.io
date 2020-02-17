---
title: "R is for Art"
author: "Patrick DeBiasse"
date: "2020-17-02"
output: 
  html_document:
    keep_md: true
---

I'd guess a common source of anxiety when flying is anticipating your immediate neighbors - the screeching child, the sniffler who's "pretty sure it's just allergies", the broad and sharp-elbowed. Once seated 3 inches from one (or a few) of these, no amount of stroopwafel and headphone volume can really salvage things.  

That said, now and then this forced proximity can be great. Case in point: this past November,  what was a 4 hour flight from Newark to Denver turned into a 4ish hour chat spanning best neighborhoods in Philly, new cancer treatments, food truck entrepreneurship, and (the inspiration for this work) metallic topographical maps. My new friend's name was Jim Abraham, and among other things, he hand-sculpts metallic topographical maps. Here is Nantucket Island: 

<center>

![**Nantucket Island, by Jim Abraham. View more of his work** [here.](https://www.abrahamartistry.com/)](C:\Users\Pat\Desktop\Patrick-DeBiasse.github.io\images\R is for Art\1_Nantucket Island_metalic.png)

</center>

About a month after meeting Jim, I stumbled upon Rayshader - an R package that allows you to create detailed 3D renderings of topography using publicly available elevation data. 

I immediately thought of his work, and emailed him to learn more about how he was currently getting topography for his regions of interest. Turns out he had grown accustomed to manually crawling through the terrain on Google Maps - a slow and painful process.

I gladly offered to generate topography using Rayshader to relieve some of this arduous map-crawling. Below is Nantucket Island again, rendered with elevation data and a satellite image overlay: 

<center>

![**Nantucket Island, Rendered with Elevation Data.**](C:\Users\Pat\Desktop\Patrick-DeBiasse.github.io\images\R is for Art\2_Nantucket Island_rendered.png)

</center>

Admittedly, flat regions like Nantucket Island don't lend themselves to exciting 3D renderings (there's just not a lot of the 3rd dimension going on). But for more mountainous regions, things can look much more interesting, while also being more  helpful for Jim. 

To demonstrate this, I generated 3D topography for one of Salt Lake City's more popular hikes - the Living Room Trail. With about a 1,000 feet of elevation gain, there's a lot more to see (for reference, the highest elevation on Nantucket is 30 feet).  

See below for the code needed to generate the rendering! 

**Install packages**:

```r
library(rayshader) #3D plotting. 
library(geoviz) #Elevation data downloader.
library(av) #For making videos (only needed if you want to make things like rotating GIFs). 
```

**Set up the region**: 

```r
#Lat and long for Living Room Trail, SLC.
lat <- 40.766632 
long <- -111.811704

#Size of surrounding area you want to visualize.
square_km <- 0.8
```

**Choose the plotting resolution**:

```r
max_tiles <- 20 
#Tiles are little  pieces of the overall map that get stitched together.
#Requesting more tiles results in a higher resolution (but slower to generate) image.
```

**Get elevation data**:

```r
#Get Digital Elevation Model from 'mapzen' via 'Amazon Public Datasets'.
dem <- mapzen_dem(lat, long, square_km, max_tiles = max_tiles)

#elmat stands for elevation matrix.
elmat = matrix(
  raster::extract(dem, raster::extent(dem), method = 'bilinear'),
  nrow = ncol(dem),
  ncol = nrow(dem)
)
```
Some background on elevation data:
Thanks to NASA's Space Shuttle Radar Topography Mission, elevation data for most of the earth's surface is publicly available with spatial (x-y) resolution of 30 meters and height (z) resolution of about 15 meters. You can explore and manually download this data [here](https://www.usgs.gov/earthexplorer-0/).

**Add a satellite image overlay**:

```r
#To get a mapbox key, go here https://docs.mapbox.com/help/glossary/access-token/.
mapbox_key <- "pk.eyJ1IjoicGF0cmlja2RlYiIsImEiOiJjazVtcmltcWIxMnJmM21wbDZkcHlzMzEwIn0.sAIvHarJXAc6VHgomtK2yQ"

#Rayshader has built-in terrain aesthetics you can choose from, but satellite images are more realistic:
overlay_image <-
  slippy_overlay(
    dem,
    image_source = "mapbox",
    image_type = "satellite",
    png_opacity = 0.6,
    api_key = mapbox_key
  )
```


**Render the Rayshader scene**:

```r
elmat %>%
  sphere_shade(texture = "desert") %>%
  add_water(detect_water(elmat), color = "imhof4") %>%
  add_shadow(ray_shade(elmat, zscale = 3), 0.5) %>%
  add_shadow(ambient_shade(elmat), 0) %>%
  add_overlay(overlay_image) %>%
  plot_3d(elmat, zscale = raster_zscale(dem), fov = 0, theta = 0, zoom = 0.75, phi = 55, windowsize = c(1000, 800))
Sys.sleep(0.2)

render_movie('SLC_Living_Room_Trail')
```

**Final output:**

<center>

![**3D Topography of Living Room Trail and Surrounding Area.**](C:\Users\Pat\Desktop\Patrick-DeBiasse.github.io\images\R is for Art\3_livingroomtrail_onerotation_reduced.gif) 

</center>

**Closing Thoughts**:

3D visualizations of topography have quite a few promising applications - helping Jim spend more time with his pieces and less time scrolling through 2D terrain is one fairly niche use-case (nonetheless one I'm excited about supporting).

For something that might appeal to a broader audience, I think replacing standard 2D topographical maps with 3D renderings would be a boon for hikers. I for one have botched more than a few hikes not interpreting standard topogrpahical maps correctly - this is the nature of making 2D abstractions of what is inherently 3D information. See below for what the Living Room Trail looks like, represented with standard topography:

<center>

![**Living Room Trail Topography (from AllTrails.com).**](C:\Users\Pat\Desktop\Patrick-DeBiasse.github.io\images\R is for Art\4_living room trail_topography.PNG)

</center>

If AllTrails (a popular hiking app) replaced their standard topographical maps with 3D renderings of this sort, users would benefit from a much clearer idea of where they are located on the hike and the nature of what's down the trail. This would encourage more confident exploration and likely cut down on wrong turns. 

If you have any thoughts or questions on the above, it'd be great to hear from you: [patrick.debiasse@gmail.com](patrick.debiasse@gmail.com).

**Acknowledgements**:

* Jim Abraham for the collaboration, view his work [here](https://www.abrahamartistry.com/)

* Tyler Morgan-Wall for creating [Rayshader](https://www.rayshader.com/)

* Neil Charles for creating [Geoviz](https://cran.r-project.org/web/packages/geoviz/index.html) 