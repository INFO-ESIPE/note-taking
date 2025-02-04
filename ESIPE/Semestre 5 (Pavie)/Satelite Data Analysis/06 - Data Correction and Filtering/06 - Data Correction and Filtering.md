[[Satellite Data Analysis]]
22/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Pre-processing

The **pre-processing** chain is here to correct the **raw dataset** — which might contains artifacts and errors — before it becomes effectively usable.
	*This class will focus on the steps we go through while processing the data. It is enough to just be aware of those, no need to fully know how to process by yourself*

We apply two types of corrections :
- **Radiometric corrections**: Have a better estimation of the reflectance we captured. 
	1. The first step is to go back from our digital numbers to **at-sensor radiance** (the EM measurement)
	2. We then attempt to convert out at-sensor values to **at-surface radiance** by applying known effects of the atmosphere on our data (based on the conditions at the time and place we captured them, if it was cloudy, foggy...)
	3. To correctly compute the reflectance, we have to know the amount of energy coming from the sun. This data is impossible to compute with our data, we have to use another sensor (**sun photometer**) which will be placed nearby the position where we captured our data.
		*If the sensor was active at the time we captured our data, it is fine. Otherwise, we capture it at a later time (in good conditions) and apply the atmosphere and meteorological conditions to it (to get a pretty good approximation of it).
		
	The **orientation of the sensor (topographic effect)** also impacts the data we retrieve. It is better if the orientation of both our data and the photometer sensor matches, but this is not always the case.*

- **Geometric corrections**: Fix the **geometric distortions** (mostly due to perspective view, atmospheric effects, vertical projection) on the data, to make it match the surface of the earth — and ultimately, make it match other sets of data with no incompatibilities.
	***Foreshortening** is this effect where the more our data is away from the centre of the captured area, the more the angle becomes different. Note that this effect disappears if we retrieve data from afar, while it is clearly visible when our sensor is close above the target.
	A **Shadowing** effect can also happen if a part of the target hides other parts (for instance, a tall skyscraper hiding smaller entities behind it).*

 If we do possess a 3D Model of the object we have a geometric distortion issue with, we can apply **orthorectification** to obtain a better geometric representation (a render of the object like if our sensor was right on top of it). We will obtain an **ortophoto** which will, however, contains missing data on the parts we re-projected via our model.
![[orthorectification.png|500]]


****
## Thresholding

We can apply a **threshold** to our render. We will place it on the histogram. 
Any value that is bellow this threshold will be ignored from our new output.
	*The logic is similar to a mask*

The most generic method used to automatically define this threshold is **Otsu's Criterion**. It optimises the division of the image between the values/measures that interests us (e.g. water, lakes) and the ones that we want to filter out (everything else, e.g. ground, sand ...).
	*In general, those values are represented on two Gaussian distributions. The threshold will then be placed in between those two, at the most suitable position.*

More details [[02 - Shape Extraction#Thresholding|here]] and [[03 - Histogram#Automatic thresholding|here (Otsu)]], even if it applies to another field.


****
## Pixel-wise Maths

### Differencing

**Image differencing** allows us to display differences between two — or more — images in pseudocolour.
	*If the value is kept on the two images, it's value is 0 (ignored), otherwise it is 1.
	As we can see, it is boolean. This logic allows us to even obtain a **change mask***


### Image Multiplication

In general, **image multiplication** is applied between an image and a mask (not in between two actual images).
	*When a position of the mask is 0, the value from the image at this position appears. It is masked otherwise. The point is to highlight certain areas of the image.
	We can obtain those mask with thresholding (on bright areas, on shadows ...)*


### Image Division

Image division is used to remove undesired illumination on a photo
	*If we take two photos of the same area at different times, the luminance level might be very different, which will affect the rendering*
The logic is to use **ratioing**. Let's say we want to get a render of the vegetation (and ignore water, buildings, soil...). We make a division with the Red Layer and the Infrared Layer (as they are the ones which provides the highest contrast between vegetation and other material).


****
## Filters

Low-pass filter keeps the low spatial frequency, but cuts high spatial frequency (abrupt changes between two colours, e.g. black pixel next to a white one)
	*We cut out some details, and obtain a render that appears more blurry*
![[lowpass.png|599]]

High-pass filter is the opposite. It only keeps the details of the image.
	*Everything will appear grey, except the abrupt changes (details) that appears white.
	That is basically `1 - Low-pass filter`*	
![[highpass.png|500]]
