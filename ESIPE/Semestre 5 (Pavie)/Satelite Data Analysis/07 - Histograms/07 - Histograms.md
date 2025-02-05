[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Histogram and Modes

As explained [[03 - Histogram|here]], the histogram represents the tonal distribution in a digital image, showing the frequency of pixel intensity value.
Peaks in the histogram often correspond to dominant features or classes in the image, such as water, vegetation, or urban areas.
	*By applying a [[06 - Data Correction and Filtering#Thresholding|thresholding]] on this histogram, we can isolate features on the image (like a mask)*

Those peaks are known as **"modes"**:
![[modes.png|400]]

**Gaussian curves** are often used to model these histogram modes. A Gaussian distribution is described by two parameters: 
- The **mean** (average value) 
- The **variance** (spread of the data)
However, this modelling can introduce inaccuracies due to real-world factors such as noise, variability, and spectral mixing, which can distort the distribution.

The **Median** is the `float` value that divides the data into two equal parts.
The **Skewness** measures asymmetry in the histogram. For example, radar data is often right-skewed, meaning it has a long tail on the right side.
The **Kurtosis** indicates the sharpness or flatness of the histogram's peak. High Kurtosis indicates a sharp peak, while low Kurtosis indicates a flatter distribution. Kurtosis is sensitive to outliers.


****
## Non-linear Transforms

Non-linear transforms are used to convert spectral data into indexes that are tailored to highlight specific features, such as vegetation, water, or urban areas. 
	*So we enhance the interpretability of our data*

> [!example]- Examples
> - Vegetation Indexes: For example, the Normalised Difference Vegetation Index (NDVI) is a non-linear transform that uses the difference between near-infrared and red reflectance to highlight vegetation.
> - Water Indexes: Similarly, the Normalised Difference Water Index (NDWI) uses green and near-infrared bands to highlight water bodies.
> - Urban Indexes: Indexes like the Normalised Difference Built-up Index (NDBI) use shortwave infrared and near-infrared bands to highlight urban areas.
> These transforms are non-linear because they involve ratios or differences between bands, rather than simple linear combinations.
