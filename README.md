# 3D-Map-using-Bathymetry-Sri-Lanka
A brief tutorial using Bathymetry data for 3D Map Generation.
![Demo1](https://github.com/Hwoabam/3D-Map-using-Bathymetry-Sri-Lanka/blob/master/Media/Animation/SL_bathy.gif)

Numerous packages which required for the 3D map generation which are installed and loaded into the program. These packages avail the functions such as Plotting of points, shading, conversion of GIS data, validating GDAL operations,etc. 
```{r}
library(rayshader)
library(magick)
library(sp)
library(raster)
library(rgdal)
```
The Bathymetry data for is downloaded from National Oceanic Atmosphere Administrations(NOAA) website-"ngdc.noaa.gov.in". The downloaded tif data is converted to a matrix. The result of the plot is an elevation intensity profile of the tile.
```{r fig1, fig.height = 15, fig.width = 10, align= "center"}
windowsize=c(12,8)

pb_elevation = raster("D:/Assam maps/New Folder/bengal/south india and sri lanka.TIF")
height_shade(raster_to_matrix(pb_elevation)) %>%
  plot_map()
```
![Bathymetry Heatmap](https://github.com/Hwoabam/3D-Map-using-Bathymetry-Sri-Lanka/blob/master/Media/Plots/Bathy_data.png)

The coordinate reference of the elevation data matched to the imagery data
```{r}

crs(pb_elevation)
```

The desired area whose map is to be plot is cropped in a rectangular shape by defining the co-ordinates of the bottom left and top right, diagonally opposite points. The longitude and latitude of the two points are found using "Google Maps" and then defined.
```{r}
bot_lefty=79.114216
bot_leftx=5.159407
top_righty=82.279764
top_rightx=10.211739
distl2l= top_righty-bot_lefty  
scl=1-(100/(distl2l*111))
bottom_left = c(bot_lefty, bot_leftx)
top_right   = c(top_righty, top_rightx)
extent_latlong = SpatialPoints(rbind(bottom_left, top_right), proj4string=sp::CRS("+proj=longlat +ellps=WGS84 +datum=WGS84"))
e = extent(extent_latlong)
e
scl

```
the elevation data is converted into Base R matrix
```{r fig4, fig.height = 15, fig.width = 10, align= "center"}
elevation_cropped = crop(pb_elevation, e)

pbel_matrix = raster_to_matrix(elevation_cropped)

```

The geographical plot with 3d visualization with shadow detailing using ambient_shade and ray_shade. Setting up of the Parameters for water rendering and sea bed visibility. Rendering of Compass to the west of the plot and scalebar to the south
```{r fig7, fig.height=12, fig.width=9, align=}
PBshadow = ray_shade(pbel_matrix, zscale = 50, lambert = FALSE)
PBamb = ambient_shade(pbel_matrix, zscale = 50)
pbel_matrix %>%
  sphere_shade(texture = "imhof1") %>%
  add_water(detect_water(pbel_matrix), color = "imhof1") %>%
  add_shadow(PBshadow, 0.7) %>%
    add_shadow(PBamb, 0) %>%
  plot_3d( pbel_matrix, zscale = 100, fov = 0, theta = -45, phi = 45, windowsize = c(1100,900),zoom = 0.75,
            water = TRUE, waterdepth = 0, wateralpha = 0.5, watercolor = "lightblue",
            waterlinecolor = "white", waterlinealpha = 0.5)
render_scalebar(limits=c(0,50,100),label_unit = "km",position = "S", y=50,scale_length = c(scl,1))
render_compass(position = "W" )
render_snapshot(title_text = "Sri Lanka geographical|Bathymetry",title_bar_color = "#1f5214", title_color = "white", title_bar_alpha = 1)
```
![Bathymetry Heatmap](https://github.com/Hwoabam/3D-Map-using-Bathymetry-Sri-Lanka/blob/master/Media/Snapshots/SnapSL.png)

A 24 second video is generated of the animation of the rotation of the plot using ffmpeg function at framerate of 60fps.
```{r}
angles= seq(0,360,length.out = 1441)[-1]
for(i in 1:1440) {
render_camera(theta=-45+angles[i])
  render_snapshot(filename = sprintf("SL_geo%i.png", i), 
                  title_text = "Sri Lanka Geographical| Bathymetry",
                  title_bar_color = "#1f5214", title_color = "white", title_bar_alpha = 1)
}
rgl::rgl.close()
system("ffmpeg -framerate 60 -i SL_geo%d.png -vcodec libx264 -an SRI_LANKA_geographical_1.mp4 ")
```
