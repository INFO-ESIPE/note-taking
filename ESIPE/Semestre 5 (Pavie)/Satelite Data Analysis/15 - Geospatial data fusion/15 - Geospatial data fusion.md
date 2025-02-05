[[Satellite Data Analysis]]
12/12/2024
==Didn't understand this class lol, I suggest you don't rely on those notes for this lecture==
****
**Table of Contents**
```table-of-contents
```

****
## Concept

Merging data coming from various sources in order to obtain greater quality.

An easy and common situation is when our data sources covers the same area with the same resolutions (source 1: reflectance from optical image, source 2: backscattering from radar image).
	*We obtain a 6-bands images: R, G, B, IR (source 1) and VV, VH (source 2)*
We must, however, mind how we re-evaluate the values of each channel (RGB/IR layers will be represented on one byte per pixel, while backscattering-related layers will have a different representation).
> This is what we call **Raw data fusion** A.K.A. **Fusion of gridded collections of measurements** (images shares the same dimension, the same "grid").

Another situation is to fusion a cadastre (a GIS/Polygon representation) with a raster image (radar image for instance).
	*This is mostly used to discover new blocks/areas to put in the—deprecated—cadastre.*
Difference with previous case is that the samples doesn't share the same properties (polygon and raster have different representations, as explained [[05 - Data Formats and Visualisation#Raster|here]]).
> This situation is what we call **Fusion of remote sensing image data and semantic data** ("semantic" because the raster images comes with labels).

We can go further in the aforementioned data fusion method by using something like a carbon cycle, where each type of data (atmosphere, plants ...) is represented by a specific type data format (as they have a specific sensor).
> This is **model-based fusion**, which is the most meaningful way of **combining data of different types**.


****
## Raw data fusion

By "raw" data we mean "pre-processed" data. We (or the data provider) need to apply corrections to the data we want to fusion.
The fusion step is relatively straightforward since all sources represents the same area with the same dimensions. We simply merge them on one another.

The problem this method describes is image sharpening (we obtain better characteristics and details by combining our samples).
It's a simple issue, but it has a lot of methods that can solve it. The main one is **Component Substitution (CS)**.

### Component Substitution
*IHS = Intensity, Hue, Saturation*

We begin with an RGB image, which means every pixel can be represented on a 3D cube (x=g, y=r, z=b) model.
	*So, the colour space is represented by the **greyline** which goes from black (0, 0, 0) to white (255, 255, 255). If our image is monochromatic, all of its pixels will be placed on this line.*

We select a level of intensity on the greyline (between black and white). We then select a pixel with generic colour (that isn't on the greyline), and obtain an intersection point that returns an hexagonal plane. This intersection point corresponds to the greyscale equivalent of the original colour.
	*We are doing the same thing as old CRT TVs when they had to forward a black and white signal for coloured programs (to ensure backward compatibility with people which still couldn't afford a new TV set). We transmit intensity, hue and saturation instead of raw RGB colour: its an HLS-like model like described [[01 - Colours#Models|here]].*
> By doing this, we obtain the high resolution image. We merge the (high quality) intensity component with the (lower quality) hue and saturation components, and obtain our high quality RGB fused image.

![[parking.png]]
> We see the parking slots on the panchromatic image. When we merge it to the two other components, we obtain our **Pansharpened** (high quality) image:
![[parking-final.png]] 
> Of course, this method of combination works well if you manipulate RGB images (3 bands). If you have an RGB+Infrared image, it doesn't work well. We can see a direct consequence of this phenomenon above where the infrared (used to represent the material, notice where the vegetation/concrete is) is way less vivid.

A way to overcome this issue is to substitute the principal component with the panchromatic band.