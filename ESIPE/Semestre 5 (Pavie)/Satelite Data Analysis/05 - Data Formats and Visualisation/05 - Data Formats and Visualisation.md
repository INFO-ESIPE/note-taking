[[Satellite Data Analysis]]
17/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Raster

Raster images are similar to [[02 - (Digital) Images#Representation|bitmaps]] really, it is just the opposite of [[02 - (Digital) Images#Vector Graphics|vector data that is computed mathematically (object-oriented)]]. 
	*Raster is just a grid (2D Array) where each cell contains a value.*


### Bands

In general, we use bands to simply display an image with its normal colours:
![[bands.png]]

But it's up to us. For instance, a 3-bands image expect RGB for normal colours. We can decide that the R band will be replaced by the Infrared band, etc.
![[bandcolour.png]]


We call **pseudocolour** a representation that applies a custom palette of colours on a greyscale representation in order to highlight details (this is especially used in maps or weather forecast presentations about temperature, winds ...)
![[pseudoclour.png]]


****
## DEM

As briefly explained [[03 - Earth Observation Instruments#LIDAR|here]], the Digital Elevation Model (DEM) represent Earth's surface on 3D. It is possible to superimpose images of the surface on this model in order to obtain a realistic render.
	*The z axis represents the height, based on a reference level called **geoid***
![[dem.png]]

The DEM is actually a combination of the **Digital Surface Model (DSM)** (features on the ground like buildings, trees ...) and the **Digital Terrain Model (DTM)** (only represents the terrain)
![[dsm-dtm.png]]


### TIN

We can also represent the DEM on a **Triangulated Irregular Network (TIN)** instead of a regular grid like we did above.
This TIN will be composed of non-overlapping faces (triangles) calculated from a series of points:
![[tin.png]]

This is more complex to set-up, but it offers way more information, outlines natural features, and **adapts to the relief** (which is good, for a DEM that is supposed to incarnate the relief...).


****
## Vector

Mathematical approach. Essentially composed of points, lines and open/closed polygons.
	*This is used for maps*


****
## Pros & Cons

|                                                   | Raster Data | Vector Data |
| ------------------------------------------------- | ----------- | ----------- |
| **Storage Size**                                  | -           | +           |
| **Spatial Resolution**                            | +           | -           |
| **Attribute Complexity**                          | -           | +           |
| **Scalability**                                   | -           | +           |
| **Data Representation (Continuous)**              | +           | -           |
| **Data Representation (Discrete)**                | -           | +           |
| **Analysis Performance**                          | +           | -           |
| **Accuracy**                                      | -           | +           |
| **Flexibility in Representing Shapes**            | -           | +           |
| **Ease of Combining with Remote Sensing Imagery** | +           | -           |
