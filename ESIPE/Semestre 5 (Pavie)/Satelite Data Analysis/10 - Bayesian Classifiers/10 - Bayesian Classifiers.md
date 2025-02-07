x²[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Refresher

Bayesian classifiers leverage statistical analysis to **assign pixels in remote sensing images to predefined classes based on probabilities**. This approach is particularly useful when dealing with uncertainty and overlapping classes.

For instance, applying a decision threshold on a histogram can help isolate classes:
![[ESIPE/Semestre 5 (Pavie)/Satelite Data Analysis/10 - Bayesian Classifiers/threshold.png|400]]
> [!info]
> Here, raw pixel values are classified using an automatic method

However, traditional classification methods that rely on simple thresholds (e.g., histogram valleys) can lead to errors, especially when training data is limited or thresholds are imprecise.

> [!question]
> How can we select the thresholds in a reliable way?


****
## Bayesian Approach

The Bayesian approach represents each pixel in a multidimensional space (e.g., a 3D plan for RGB bands) where:

The basic idea is to represent each pixel on multidimensional space (e.g., a [[S1 - Machine Learning for Remote Sensing Applications#Probabilistic methods|3D plan (MAP)]] for RGB bands). 
	*x = R, y = G, z = B*

Now that all our pixels are spatially placed depending on their colour, we can group them by **cluster**
	*Pixels that are close to each other probably belongs to the same class*

> [!caution]
> This works, but this method can face challenges when clusters overlap or are too close to each other
> 
> ![[collision.png|300]]

### Central Limit Theorem and Gaussian PDFs

To address overlapping clusters, we can use **Gaussian Probability Density Functions (PDFs)**, which are defined by its mean (centre) and variance (spread), simplifying the representation of classes.
- **PDF (Probability Density Function):** Describes the likelihood of different pixel values for a specific class.
- **Gaussian PDF:** A bell-shaped curve that models the distribution of pixel values for a class.

![[gaussian.png|450]]

By using Gaussian PDFs, we can define thresholds based on **probability rather than distance**, which helps handle overlapping classes more effectively.

### Bayes’ Theorem

It assigns a pixel to the class with the highest **posterior probability**:
![[bayes.png|200]]
where:
- **`p(DN∣C)`:** The probability of observing a pixel value (DN) given a class (C). This is the class PDF.
- **`p(C)`:** The prior probability of the class (relative abundance of the class in the dataset).
- **`p(DN)`:** The overall probability of observing the pixel value across all classes.

The pixel is assigned to the class that maximises the posterior probability.

### Maximum Likelihood

A third way of doing it. We pass the class means and covariances as input, and it becomes possible to compute the probability for a pixel to belong to each class.

Once everything is computed, we **assign a pixel to the class with the highest probability** value.

![[likelihood.png|300]]

This method is particularly effective when the classes are well-separated and follow Gaussian distributions.


****
## Multispectral image bands

All of the above works if we work on a **single-band data**. But what if we work on **several dependant bands**?
	*In this case, we call our data a "Multispectral image". It's bands are—in general—highly correlated due to the continuity in the spectral response (i.e., discontinuities are rare).*

![[multispectral.png|400]]

To handle this, we use **Principal Component Transform (PCT)**, which removes correlation between bands.

### PCT

As briefly explained right above, PCT is a linear spectral transform that converts correlated bands into a set of uncorrelated features called **Principal Components (PCs)**. These PCs are ordered by variance, with the first few capturing most of the information. So:
	**Higher-order PCs** often represent noise and can be discarded
	**Lower-order PCs** retain the most significant information

![[pct bands.png|400]]

> [!example]
> This phenomenon is also clearly illustrated on the bands' [[08 - Spatial Analysis#Scattergrams|scattergram]]:
>
> ![[scatterplots.png|450]]

### Alternative Distributions and Techniques

Sometimes, the Gaussian is not good enough to offer a decent result.
In such cases, we can use **Kernel Density Estimation (KDE)**, a non-parametric method that estimates data distributions without assuming Gaussianity.
- **Kernel Function:** A function (e.g., Gaussian, Epanechnikov) placed at each data point to create a smooth curve.
- **Bandwidth:** A parameter that controls the width of the kernel, affecting the smoothness of the resulting distribution.

![[functions.png|400]]

>[!info]
>KDE is particularly useful for modelling complex datasets, such as SAR (Synthetic Aperture Radar) data, where standard PDFs may not fit well.