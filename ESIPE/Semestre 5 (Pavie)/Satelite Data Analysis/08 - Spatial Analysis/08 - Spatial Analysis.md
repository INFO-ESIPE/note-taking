[[Satellite Data Analysis]]
29/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Spatial Analysis?

Focuses on understanding spatial relationships and patterns in remotely sensed data. A key concept in spatial analysis is **spatial continuity**, which refers to the likelihood that the value of a pixel is influenced by the values of its neighbouring pixels.
- **Missing Values:** Missing values in spatial data can be estimated via [[08 - Spatial Analysis#Kriging|kriging]]
- **Data Continuity:** The continuity of data is often measured using [[08 - Spatial Analysis#Covariance Functions|covariance]], which quantifies how pixel values (Digital Numbers, or DNs) are related across a given distance `h`
- **Visualising Relationships:** Relationships between pixels at varying distances can be visualised via [[08 - Spatial Analysis#Scattergrams|h-Scattergrams]]


****
## Statistical Tools for Spatial Analysis

### Covariance Functions

**Covariance Functions** quantify the relationship between pixels separated by a distance `h`. They indicate how pixel values co-vary spatially.

![[covariance.png|300]]

### Semivariogram

The **Semivariogram** measures the loss of correlation as the distance between pixels increases. 
	*It is closely related to the covariance function*

![[semivariogram.png|400]]

Three main components for a Semivariogram:
- **Nugget:** Represents variability at very small distances or due to measurement errors.
- **Sill:** The maximum variability, which corresponds to the variance of the dataset.
- **Range:** The distance at which the correlation drops to the sill, indicating how far spatial influence extends.

![[semivariogram-ill.png|450]]
> [!info]
> Here, the **nugget is the missing part at the beginning** (data not captured). The **sill is observable at the rightmost part** of the graph, and the **range is what's in between beginning and sill**.

> [!example]- More talkative example
>![[urban.png|500]]
>![[urban-result.png|500]]


****
## Scattergrams

A **scattergram** (or scatter plot) is used to visualise the homogeneity of pixel values in an image.
- If dense (few points, mostly focused on the same position), the values are highly correlated.
- If spread out, the values are uncorrelated.

Scattergrams allow us to compute spatial information for specific portions of an image, helping to recognise patterns such as rectangular fields, building patterns, or lakes.

![[scattergram.png|400]]


****
## Co-occurrence Matrices (CM)

### Concept

A **Co-occurrence Matrix (CM)** is a 2D matrix that shows the joint probabilities of pixel pairs separated by a distance `h`.

The **Gray-Level Co-occurrence Matrix (GLCM)** is a practical implementation of co-occurrence matrices. It tabulates how often specific combinations of pixel values appear within a sliding window, known as a **kernel**.

### Textures

So, the GLCM quantifies how often pairs of pixel values occur at specific spatial relationships in an image, capturing patterns of intensity variation. These patterns represent **textures**, which describe surface properties like smoothness, roughness, or regularity. Textures are useful for tasks such as land cover classification or material identification.

Around 14 textures are available, here are the major ones:
- **Contrast:** Measures the intensity variation between pixels; higher values indicate sharper changes
- **Correlation:** Assesses the linear relationship between pixel pairs
- **Energy:** Also known as Angular Second Moment; measures the uniformity of the texture, with higher values indicating smoother textures
- **Homogeneity:** Evaluates the closeness of pixel values to the diagonal of the GLCM, indicating smoother textures

![[textures.png|450]]


****
## Kriging

**Kriging** is an interpolation technique used to predict the value of a variable at unobserved points based on surrounding observed data. It is particularly useful for **filling in missing data and reconstructing bad values** in remote sensing images.

Kriging takes into account the spatial continuity of the data, making it a powerful tool for spatial analysis.

![[kriging.png|700]]
