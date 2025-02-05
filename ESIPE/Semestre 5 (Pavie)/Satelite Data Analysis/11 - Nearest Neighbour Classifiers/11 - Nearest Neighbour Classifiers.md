[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Refresher on classification with Proximity

Classification involves assigning data points (e.g., pixels in an image) to predefined classes based on their similarity to known examples. Proximity, often measured by **distance**, is a central concept in classifiers like **k-Nearest Neighbours (k-NN)**.
But we still have a problem...

![[distance.png|300]]
> [!question]
> What if clusters are too close to each other?
> If a pixel ends up in the red area, does it belongs more into the left cluster, or the one on the right ?*


****
## k-Nearest Neighbours (k-NN)
*Supervised*

k-NN is a simple yet powerful **classifier leveraging proximity for classification**. Its performance depends heavily on preprocessing (e.g., feature scaling, training set optimisation) and careful handling of dimensionality.

**How it works**: The algorithm assigns a point to the most common class among its **k nearest neighbors**. 
It is a **voting system**: We retrieve all the near pixels and check the class we assigned them, and attribute the current value to the most common class in this neighbourhood.

> [!info] Why not just 1 neighbour? 
> Using a single neighbour (1-NN) can lead to misclassifications due to noise. Using multiple neighbors (k > 1) increases robustness.

Distance metrics, such as **euclidean distance** (for continuous data) or **hamming distance** (for categorical data), define how "close" two points are.

**Weighted k-NN** is a more advanced version of k-NN which assigns weights to neighbours based on their distance. Closer neighbors have more influence on the classification decision, improving accuracy.


****
## Challenges

The first challenge is to **get rid of the various irrelevant features**
	*Some attributes may not contribute to the classification but distort distance measurements...*
**Solution**: Feature selection or dimensionality reduction (e.g., Principal Component Analysis).

Another issue is **scale-related**. Features with larger ranges dominate distance calculations.
**Solution**: Normalise features to a common range (0–1).

The last concern is about the **curse of dimensionality (or "Hughes Phenomenon")**:
Increasing the number of features improves classification accuracy initially but eventually reduces it as noise increases and training samples become sparse.

![[hugues.png|450]]

**Solution**: Well, just avoid excessive features
	*At least without the adequate training data that goes with it*


****
## Improve k-NN

The first step towards improving k-NN is to simply **improve our training set**.
	*Mostly by removing points which are redundant (doesn't add new information) and misleading (belong to one class but are near other classes).
![[redundancy.png|550]]
