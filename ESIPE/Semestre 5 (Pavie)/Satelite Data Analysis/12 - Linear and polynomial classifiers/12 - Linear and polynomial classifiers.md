[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Classification via Boundaries

Classification can be understood as dividing data into classes by defining boundaries.
	*Its the third way of doing it, the two others were [[11 - Nearest Neighbour Classifiers|k-NN]] and [[10 - Bayesian Classifiers|Bayesian classifiers]]*

These boundaries can be **linear** or **non-linear**, depending on the nature of the data. 
![[boundaries.png|500]]

Linear classifiers are simpler and computationally efficient but may not work well for complex datasets, leading to the need for polynomial or kernel-based methods.


****
## Linear Discriminant Analysis (LDA)

With LDA, we apply a linear function to separate classes by finding the optimal boundary.
	*We just put a straight line in between clusters to separate them by classes (left part of the image above)*
The boundary is defined by **weights** (`w`) and a **bias** (`b`).
	*The bias defines the minimum level needed to get a decision, while the weight of a feature defines it's importance (0 being irrelevant)*

The biggest issue with this method is the **Binary Problem**: Linear classifiers work naturally for binary classes, but multi-class classification requires additional steps:
- **One against All**: Classify one class against all others.
	*This requires to change our training data, and it only works if the number of class is small*
- **Master Classifier**: Resolves conflicts when a sample belongs to multiple classes
![[ESIPE/Semestre 5 (Pavie)/Satelite Data Analysis/12 - Linear and polynomial classifiers/classifier.png|550]]


****
## Perceptron Learning Algorithm

Another linear solution is to use this **iterative algorithm** to learn optimal weights (`w`, `b`) by minimising classification errors.
The process is simple:
1. Starts with random weights.
2. Adjusts weights based on errors using a **learning rate** (`h`).
*Like die and retry in games, kind of. You just keep correcting until the result seems to fit*

The main problem with this method is **redundant features**.
	*If two attributes shares the same value, the algorithm will give them the same weight. They will both be considered as important, while one of them is clearly redundant and should be eliminated*


****
## Non-Linear Classifiers

Linear methods struggle with **complex data that cannot be separated by straight lines**.
Non-linear classifiers address this issue in two ways

### Polynomial Transformations

Use polynomial functions to model complex boundaries. In a nutshell, it allows to **obtain curves instead of simple straight lines**. 
	*It's a way (in ML) to make linear models capable of learning complex relationships.
	Instead of using `x1​,x2​,…,xn​` inputs, we add combinations of them `w0 + w1x1 + w2x2 + w3x1...`*

It's accurate, but it is computationally intensive and introduces **Overfitting**.
	*The model becomes too flexible, learning not only the general pattern but also noise present in the training data. It struggles to generalise to new, unseen data*


### Support Vector Machines (SVM)

This method finds an **optimal hyperplane** that maximises the margin between classes.
	*We make "non-linearly separable datasets" separable by increasing the dimensional space*

The **kernel trick** is how we do this. We can, for instance, take a 2D dataset and make it linearly separable in a 3D space using a polynomial or Gaussian kernel.
![[kerneltrick.png|550]]

**Pros:** Works well on high-dimensional datasets (like hyperspectral imagery)
**Cons:** Not effective on huge datasets, very sensitive to noise, and generating the kernel might be tricky in specific contexts (easier said than done)


****
## Summary

Linear classifiers like LDA and perceptrons are efficient for simple tasks but struggle with complex datasets. 
SVMs with kernel methods provide a robust solution for non-linear problems, making them particularly valuable for remote sensing applications where data is high-dimensional and highly variable.