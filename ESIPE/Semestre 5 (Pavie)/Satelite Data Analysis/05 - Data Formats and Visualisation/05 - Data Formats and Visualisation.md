[[Satellite Data Analysis]]
17/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Raster

Raster images are similar to [[02 - (Digital) Images#Representation|bitmaps]] really, it is just the opposite of [[02 - (Digital) Images#Vector Graphics|vector data that is computed mathematically (object-oriented)]]. 
	*Raster is just a grid (2D Array) where each cell (e.g., pixel) contains a value (e.g., colour, elevation...)*

### Bands

In remote sensing, **bands** refer to different ranges of wavelengths captured by the sensor. 
Typically, bands are used to display images in their normal colours. For example, a 3-band image usually corresponds to the Red, Green, and Blue (RGB) bands, which combine to create a true-colour image:

![[bands.png]]

However, bands can be manipulated for specific purposes. For instance, the Red band in an RGB image can be replaced with an Infrared band to highlight vegetation or other features:

![[bandcolour.png|550]]


We call **pseudocolour** a representation that applies a custom palette of colours on a greyscale image in order to highlight details (this is especially used in maps or weather forecast presentations about temperature, winds ...)
![[pseudoclour.png|700]]


****
## DEM

As briefly explained [[03 - Earth Observation Instruments#LIDAR|here]], the Digital Elevation Model (DEM) **represent Earth's surface on 3D**. It is possible to superimpose images of the surface on this model in order to obtain a realistic render.
	The z axis represents the height, based on a reference level called **geoid**

![[dem.png|500]]

The DEM is actually a combination of two models: 
- The **Digital Surface Model (DSM)**, features on the ground like buildings, trees...
- The **Digital Terrain Model (DTM)**, which only represents the bare ground surface (without anything on top of it)

![[dsm-dtm.png|400]]

### TIN

A **Triangulated Irregular Network (TIN)** is an alternative way to represent a DEM. Instead of using a regular grid, a TIN uses a series of non-overlapping triangles calculated from a set of points.

![[tin.png|500]]

**Advantages of TIN:**
    - Provides more detailed information.
    - Outlines natural features more accurately.
    - Adapts better to the terrain's relief, making it more suitable for representing complex landscapes.
    
**Disadvantages of TIN:**
    - More complex to set up compared to a regular grid.
    - Requires more computational resources.


****
## Vector

Mathematical approach to representing geographic features. It is composed of points, lines, and polygons (both open and closed).
	*Widely used for maps, where precision and scalability are important*


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
> [!info]- Details on some categories
> **Spatial Resolution:** Raster data can represent continuous data (e.g., elevation, temperature) more effectively, while vector data is better for discrete features (e.g., roads, boundaries).
> **Ease of Combining with Remote Sensing Imagery:** Raster data is easier to combine with remote sensing imagery, as it is already in a grid format.
