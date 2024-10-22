[[Satellite Data Analysis]]
22/10/2024
****

The **pre-processing** chain is here to correct the **raw dataset** — which might contains artifacts and errors — before it becomes effectively usable.
	*This class will focus on the steps we go through while processing the data. It is enough to just be aware of those, no need to fully know how to process by yourself*

We apply two types of corrections :
- **Radiometric corrections** : Have a better estimation of the reflectance we captured. 
	1. The first step is to go back from our digital numbers to **at-sensor radiance** (the EM measurement)
	2. We then attempt to convert out at-sensor values to **at-surface radiance** by applying known effects of the atmosphere on our data (based on the conditions at the time and place we captured them, if it was cloudy, foggy ...)
	3. To correctly compute the reflectance, we have to know the amount of energy coming from the sun. This data is impossible to compute with our data, we have to use another sensor (**sun photometer**) which will be placed nearby the position where we captured our data.
		*If the sensor was active at the time we captured our data, it is fine. Otherwise, we capture it at a later time (in good conditions) and apply the atmosphere and meteorological conditions to it (to get a pretty good approximation of it).
		
	   The **orientation of the sensor (topographic effect)** also impacts the data we retrieve. It is better if the orientation of both our data and the photometer sensor matches, but this is not always the case.*

- **Geometric corrections** : Fix the **geometric distortions** (mostly due to perspective view, atmospheric effects, vertical projection) on the data, to make it match the surface of the earth — and ultimately, make it match other sets of data with no incompatibilities.
	***Foreshortening** is this effect where the more our data is away from the centre of the captured area, the more the angle becomes different. Note that this effect disappears if we retrieve data from afar, while it is clearly visible when our sensor is close above the target.
	A **Shadowing** effect can also happen if a part of the target hides other parts (for instance, a tall skyscraper hiding smaller entities behind it).*
	
1. If we do possess a 3D Model of the object we have a geometric distortion issue with, we can apply **orthorectification** to obtain a better geometric representation (a render of the object like if our sensor was right on top of it). We will obtain an **ortophoto** which will, however, contains missing data on the parts we re-projected via our model.


****
