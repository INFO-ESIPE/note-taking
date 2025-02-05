[[Satellite Data Analysis]]
22/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Pre-processing

The **pre-processing** chain is here to correct the **raw dataset**—which might contains artifacts and errors—before it becomes effectively usable for analysis.
	*This class will focus on the steps we go through while processing the data. It is enough to just be aware of those, no need to fully know how to process by yourself*

We apply two types of corrections:
1. **Radiometric Corrections:** Provides a better estimation of the reflectance captured by the sensor
    - **Step 1:** Convert digital numbers to **at-sensor radiance**, which represents the original electromagnetic (EM) measurement
    - **Step 2:** Convert at-sensor radiance to **at-surface radiance** by accounting for atmospheric effects (e.g., clouds, fog) that were present during data capture
    - **Step 3:** To accurately compute reflectance, we need to know the amount of solar energy at the time of capture. This is measured using a **sun photometer**, placed near the capture location.
        *If the sun photometer was active during data capture, its measurements can be used directly. Otherwise, measurements taken later under similar conditions can be adjusted to approximate the solar energy at the time of capture.*
        Keep in mind that the orientation of the sensor can also affect the data, its a **topographic effect**. Ideally, the orientation of both the sensor and the sun photometer should match, but this is not always possible.
        
2. **Geometric Corrections:** These corrections address **geometric distortions** caused by perspective view, atmospheric effects, and vertical projection. The goal is to align the data with the Earth's surface and ensure compatibility with other datasets.
    - **Foreshortening:** This effect occurs when objects farther from the center of the captured area appear at different angles. It is more pronounced when the sensor is close to the target.
    - **Shadowing:** This occurs when one part of the target obscures another, such as a tall building hiding smaller objects behind it.

If a 3D model of the object is available, **orthorectification** can be applied to correct geometric distortions. This process produces an **orthophoto**, which represents the object as if the sensor were directly above it. 
	*However, orthophotos may contain missing data in areas that were re-projected using the model.*

![[orthorectification.png|500]]


****
## Thresholding

**Thresholding** is a technique used to isolate specific features in an image by applying a threshold to the histogram. Values below the threshold are ignored, effectively creating a mask.

The most common method for automatically determining the threshold is **Otsu's Criterion**, which optimises the separation between two classes of values (e.g., water and land) by placing the threshold between two Gaussian distributions.
	*More details [[02 - Shape Extraction#Thresholding|here]] and [[03 - Histogram#Automatic thresholding|here (Otsu)]], even if it applies to another field.*


****
## Pixel-wise Maths

### Differencing

**Image differencing** highlights differences between two or more images using pseudocolour.
If a value is the same in both images, it is set to 0 (ignored). If it differs, it is set to 1. This boolean logic can be used to create a **change mask**.

### Image Multiplication

**Image multiplication** is typically applied between an image and a mask (rather than between two images).    
When a position in the mask is 0, the corresponding value in the image is masked. This technique is used to highlight specific areas of the image, often derived from thresholding.

### Image Division

**Image division** is used to normalise illumination differences between images captured at different times.    
For example, to highlight vegetation, the Red and Infrared bands can be divided (**ratioing**), as these bands provide the highest contrast between vegetation and other materials.


****
## Filters

### Low-pass Filter

A **low-pass filter** retains low spatial frequencies (gradual changes) while cutting high spatial frequencies (abrupt changes, such as a black pixel next to a white one).
	*We cut out some details, and obtain a render that appears more blurry*

![[lowpass.png|500]]

### High-pass filter

A **high-pass filter** does the opposite: It retains high spatial frequencies (details) while cutting low spatial frequencies.
	*Everything will appear grey, except the abrupt changes (details) that appears white.
	That is pretty much a sharpening filter*	

![[highpass.png|500]]
