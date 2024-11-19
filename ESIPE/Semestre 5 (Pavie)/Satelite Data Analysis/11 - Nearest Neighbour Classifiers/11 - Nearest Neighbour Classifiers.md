[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Refresher on classification with Proximity

Classification assigns data points (pixels in an image) to classes based on their similarity to known examples. 
Proximity is used as the measure of similarity, making **distance** a central concept in classifiers like **k-NN**.

But we still have a problem: What if clusters are too close to each other ?
![[distance.png]]
	*If a pixel ends up in the red area, does it belongs more into the left cluster, or the one on the right ?*


****
## k-Nearest Neighbours (k-NN)

k-NN is a simple yet powerful **classifier leveraging proximity for classification**. Its performance depends heavily on preprocessing (feature scaling, training set optimisation) and careful handling of dimensionality.

The algorithm assigns a point to the most common class among its **k nearest neighbors**.
**Why k and not 1 neighbour ?** Well, a single neighbour (1-NN) may product misclassifications due to noise, so using multiple neighbors increases robustness.

Distance metrics, such as **Euclidean distance** or **Hamming distance**, define how "close" two points are.


The most performing distance measure is to use **Weighted k-NN**, which **weights neighbours based on their distance**
	*closer neighbors have more influence...*


****
## Challenges

The first challenge is to **get rid of the various irrelevant features**
	*Some attributes may not contribute to the classification but distort distance measurements...*
**Solution**: Feature selection or dimensionality reduction (e.g., Principal Component Analysis).

Another issue is **scale-related**. Features with larger ranges dominate distance calculations.
**Solution**: Normalise features to a common range (0–1).

The last concern is related to the **Curse of Dimensionality (or "Hughes Phenomenon")**:
Increasing the number of features improves classification accuracy initially but eventually reduces it as noise increases and training samples become sparse.
![[hugues.png]]

**Solution**: Well, just avoid excessive features
	*At least without the adequate training data that goes with it*


****
## Improve k-NN

The first step towards improving k-NN is to simply **improve our training set**.
	*Mostly by removing redundant (points that doesn't add new information) and misleading (points that belong to one class but are near other classes) points*
![[redundancy.png]]
