[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Histogram and Modes

As explained [[03 - Histogram|here]], the histogram represents the tonal distribution in a digital image.

Usually, peaks in the histogram correspond to dominant features or classes (e.g., water, vegetation).
	*By applying a [[06 - Data Correction and Filtering#Thresholding|thresholding]] on this histogram, we can isolate features on the image (like a mask)*
Those peaks are called **modes**:
![[modes.png]]

In general, we use **Gaussian curves to model histogram modes**, describing data distributions with just two parameters (mean and variance).
	*This may introduce inaccuracies of real-world, as noise, variability, or spectral mixing affects the distribution...*

The **Median** is the `float` value that divides the data into two equal parts.
The **Skewness** measures asymmetry in the histogram (e.g., right-skewed radar data).
The **Kurtosis** indicates the sharpness or flatness of the histogram's peak, sensitive to outliers.


****
## Non-linear Transforms

Non-linear transforms convert spectral data into indexes tailored to specific features like vegetation, water, or urban areas.
