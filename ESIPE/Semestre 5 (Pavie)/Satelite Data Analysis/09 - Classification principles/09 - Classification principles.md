[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## What is classification?

It's about assigning each portion of the Earth's surface to a predefined **class** (e.g., land cover types like water, vegetation, or urban areas). 
The output is typically a **thematic map**, providing actionable insights from satellite data.

This is useful in Machine Learning in various situations:
- Involves optimisation techniques to minimise error metrics like "Overall Accuracy".
- Adequate training data is essential for supervised methods.

To obtain decent classification, it is mandatory to provide a **reliable training** to our algorithm.

Example of classification:
![[classification.png|800]]

*Note that (Remote sensing) classification overlaps with **semantic segmentation** in computer vision.*

### Key Concepts

**Land Cover (LC) & Land Use (LU)**:
-  **LC**: The physical materials covering the Earth's surface (e.g., grass, water).
-  **LU**: The human-defined usage of the land (e.g., agriculture, residential).

**Classification Unit**: The smallest portion of data assigned to a class, often a pixel or a group of pixels (superpixel).
The **Classifier** is what assigns LC/LU to each classification unit:

![[classifier.png|500]]

### Hard & Fuzzy classification

**Hard**: Each pixel belongs to one and only one class

![[hard.png|450]]

**Fuzzy**: Pixels can partially belong to multiple classes with assigned probabilities.

![[fuzzy.png|500]]


****
## Methods of Classification

**Supervised Classification** is a **knowledge-driven approach** where classes are predefined and the algorithm is trained on labeled data (Ground Truth).
1. Collect training data representative of each class
2. Use it to "train" the classifier to recognise these classes in the dataset
![[supervised.png|300]]

**Unsupervised Classification** is a **data-driven approach** where the algorithm groups pixels based on inherent similarities.
	*No a priori information, only user-set parameters–limited human input*
![[unsupervised.png|300]]


****
### Essential Elements of Classification

**Ground Truth (GT)**: Accurate, external data used for training and validating the classifier.
**Test Set**: A separate dataset to evaluate the classifier's accuracy.
**Sampling Strategies**: Methods for collecting representative data points; usually 50–100 samples per class.

![[sampling.png|300]]


****
## Evaluating Classification Results

We need to know how “reliable” the classification is (how well the classification matches the actual at-ground situation...)

### Confusion Matrix

We compare the classification with ground truth on a unit-by-unit basis (frequently means pixel-by-pixel).
Once this is done, we store it into a **confusion matrix**, which is a table comparing predicted classes against actual Ground Truth classes.

![[confusion matrix.png|550]]
> [!info]
> Here, diagonal values are the ones that were correctly determined (classifier found the correct class)*

Two types of errors can occur:
- **Omission Errors**: Missed pixels for a class (in red)
- **Commission Errors**: Incorrectly assigned pixels to a class (in blue)

We can calculate the overall accuracy like so:
Number of correct pixels: 1345 + 2315 + 678 = 4328
Total number of pixels : 4766
```
Accuracy = 4328 / 4766 = 0.9080990... 
         = 90.81%
```

> [!question]
> While this accuracy is good, there is a caveat: An accuracy greater than zero can be obtained even from random class assignment to each pixel.
> (Like when you answer randomly to a multiple choice questionnaire... some of them will be correct out of pure luck)
> 
> Can we fix this?

### Kappa Coefficient `𝑘`

Measures classification accuracy while accounting for chance agreement.
We can attempt to compute this with a simple formula:
![[kappa.png|150]]
	where `P0` is the probability of correct classification
	and `Pc` is the probability of correct classification attributable to pure chance

> [!caution]
> While the Kappa Coefficient is useful, it is not always reliable or appropriate for all applications. It assumes that the chance agreement is uniform across all classes, which may not always be the case.
