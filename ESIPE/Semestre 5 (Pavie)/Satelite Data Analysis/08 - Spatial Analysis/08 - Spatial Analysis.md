[[Satellite Data Analysis]]
29/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Spatial Analysis ?

Focuses on understanding spatial relationships and patterns in remotely sensed data.
**Spatial Continuity** refers to the likelihood of data at a location depends on the values in its neighbourhood.

We estimate **missing values** via [[08 - Spatial Analysis#Kriging|kriging]].
We measure **data continuity** via covariance
	*Measures how pixel values (DNs) are related across a given distance `h` in the image*
We visualise **relationships between pixels at varying distances** with [[08 - Spatial Analysis#Scattergrams|h-Scattergrams]].


****
## Statistical Tools for Spatial Analysis

### Covariance Functions

**Covariance Functions** quantifies the relationship between pixels separated by distance `h`.
	*Indicates how values co-vary spatially*

![[covariance.png|300]]


### Semivariogram

**Semivariogram** measures the loss of correlation as the distance between pixels increases.
	*It is usually tightly related to covariance...*
![[semivariogram.png|400]]

A Semivariogram is composed of:
- **Nugget**: Variability at small distances or due to measurement errors/miss.
- **Sill**: Maximum variability, corresponding to dataset variance.
- **Range**: Distance where correlation drops to the sill, showing how far spatial influence extends (minimum `h` where sill is reached).
![[semivariogram-ill.png|450]]

Here, the **nugget is the missing part at the beginning** (data not captured). The **sill is observable at the rightmost part** of the graph, and the **range is what's in between beginning and sill**. 


Here is a more talkative situation:
![[urban.png|500]]
![[urban-result.png|500]]


****
## Scattergrams

The scattergram indicates how homogeneous the values are

If the scattergram is dense (few points, mostly focused on a same position on the scattergram), the values are correlated. They are uncorrelated otherwise.
	*This allows us to compute spatial information on portions of an image. We can recognise patterns in an area (rectangular fields, buildings pattern, lakes ...)*
![[scattergram.png|400]]


****
## Co-occurrence Matrices (CM)

A 2D matrix showing joint probabilities of pixel pairs separated by distance `h`.

The **Gray-Level Co-occurrence Matrix (GLCM)** is the practical implementation of Co-occurrence matrices.
It tabulates how often specific pixel value combinations appear in a sliding window.
	*This window is called "kernel"*


### Textures

So, a (GL)CM quantifies how often pairs of pixel values occur at specific spatial relationships in an image, capturing patterns of intensity variation. These patterns represent **textures**, describing surface properties like smoothness, roughness, or regularity, useful for tasks like land cover classification or material identification.

Around 14 textures are available, here are the major ones:
- **Contrast**: Measures intensity variation; higher values indicate sharper changes.
- **Correlation**: Assesses the linear relationship between pixel pairs.
- **Energy**: Also called Angular Second Moment; measures uniformity, with higher values for smoother textures.
- **Homogeneity**: Evaluates closeness of pixel values to the diagonal of the GLCM, indicating smoother textures.

![[textures.png|450]]


****
## Kriging

It's an Interpolation Technique that **predicts the value of a variable at unobserved points based on surrounding observed data**.

It's main purpose is to fill missing data — and reconstruct bad values — in remote sensing images.

![[kriging.png|500]]