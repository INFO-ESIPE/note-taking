[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Refresher

Bayesian classifiers leverage statistical analysis to assign pixels in remote sensing images to predefined classes based on probabilities. 

For instance, applying some kind of "decision" threshold on our histogram to isolate classes in it:
![[ESIPE/Semestre 5 (Pavie)/Satelite Data Analysis/10 - Bayesian Classifiers/threshold.png]]
*Here, our values were row at first. We run an automatic method on it that classified them for us.*

But this is a simple thing to say...
Traditional classification uses thresholds (e.g., **histogram valleys**) to separate classes (like above). But limited training data or imprecise thresholds can lead to errors.

**How can we select the thresholds in a reliable way ?**


****
## Bayesian Approach

The basic idea is to represent each pixel of the image on a [[S1 - Machine Learning for Remote Sensing Applications#Probabilistic methods|3D plan (MAP)]]. 
	*x = R, y = G, z = B*
Now that all our pixels are spatially placed depending on their colour, we can group them by **cluster**
	*Pixels that are close to each other probably belongs to the same class*

This works, but what if a collision happens (clusters are too close to each other) ?
![[collision.png]]


### **Central Limit Theorem and Gaussian PDFs**

Reminder: 
- PDF is the Probability Density Function, describing how likely different values (e.g., pixel intensities) are for a specific class. It helps model the distribution of data.
- Gaussian is a common type of PDF shaped like a bell curve, defined by its mean (centre) and variance (spread). It's often used because many natural phenomena approximate this distribution.


First solution is a **Gaussian PDF**, which depends only on the mean and variance, simplifying class representation.
![[gaussian.png]]

We can apply this to define the thresholds, but it means we would define them by **probability** instead of **distance on the MAP (or equivalent)**...

### Bayes’ Theorem

The principle is the following: We assign **a pixel to the class with the highest posterior probability**.
![[bayes.png]]
where:
- `p(DN∣C)`: Class PDF.
- `p(C)`: Prior probability (relative abundance of the class).
- `p(DN)`: Overall data distribution.


### Maximum Likelihood

A third way of doing it. We pass the class means and covariances as input, and it becomes possible to compute the probability for a pixel to belong to each class.

Once everything is computed, we **assign a pixel to the class with the highest probability** value.
![[likelihood.png]]


****
## Multispectral image bands

All of the above works if we work on a **single-band data**. But what if we work on **several dependant bands** ?
	*In this case, we call our data a "Multispectral image". It's bands are — in general — highly correlated due to the continuity in the spectral response (i.e., discontinuities are rare).*
![[multispectral.png]]

The main solution to this is to use **Principal Component Transform (PCT)**, allowing us to **remove correlation**.

### PCT

As briefly explained right above, PCT is a linear spectral transform that **turns correlated bands** (features) **into a set of uncorrelated features** called Principal Components (PCs).

Those PCs are **ordered by variance**, with the first few capturing most information.
	*Higher-order PCs often represent noise and can be discarded (see band 7 and 8 bellow, and how we seem to lose information as each band goes by).*
![[pct bands.png]]

This phenomenon is also clearly illustrated on the bands' [[08 - Spatial Analysis#Scattergrams|scatterplots]].
![[scatterplots.png]]


### Alternative Distributions and Techniques

Sometimes, the Gaussian is not good enough to offer a decent result.

We can use a **Kernel Density Estimation**, which is a non-parametric method to estimate data distributions without assuming Gaussianity.
It works by placing a **kernel function** at each data point and summing them to create a smooth curve:
![[functions.png]]

The **bandwidth** parameter controls how wide each kernel is, affecting the smoothness of the resulting distribution. It’s useful for modelling complex datasets, such as SAR data, where standard PDFs may not fit well.