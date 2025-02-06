[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Classification via boundaries

Classification can be understood as dividing data into classes by defining boundaries.
	*Its the third way of doing it, the two others were [[11 - Nearest Neighbour Classifiers|k-NN]] and [[10 - Bayesian Classifiers|Bayesian classifiers]]*

These boundaries can be **linear** or **non-linear**, depending on the nature of the data.
![[boundaries.png|500]]

- **Linear classifiers** are simpler and computationally efficient but may struggle with complex datasets.
- **Non-linear classifiers** (e.g., polynomial or kernel-based methods) are better suited for complex data but are more computationally intensive.


****
## Linear Discriminant Analysis (LDA)

**Linear Discriminant Analysis (LDA)** is a linear classification method that separates classes by finding an optimal linear boundary. This boundary is defined by **weights** (`w`) and a **bias** (`b`).
- The bias (`b`) defines the minimum level needed to make a decision.
- The weights (`w`) determine the importance of each feature (a weight of 0 means the feature is irrelevant).


The biggest issue with this method is the **Binary Problem**: Linear classifiers work naturally for binary classes, but multi-class classification requires additional steps:
1. **One-vs-All (OvA):** Classify one class against all others. This approach requires modifying the training data and is only effective when the number of classes is small
2. **Master Classifier:** Resolves conflicts when a sample is assigned to multiple classes
    ![[ESIPE/Semestre 5 (Pavie)/Satelite Data Analysis/12 - Linear and polynomial classifiers/classifier.png|500]]


****
## Perceptron Learning Algorithm

The **Perceptron Learning Algorithm** is an iterative method for learning optimal weights (`w`, `b`) by minimising classification errors. The process is as follows:
1. Start with random weights.
2. Adjust weights based on errors using a **learning rate** (`η`).
	 _This is similar to a trial-and-error approach, where the algorithm keeps correcting itself until it achieves a good fit._

The main problem with this method is **redundant features**:
	If two attributes shares the same value, the algorithm will give them the same weight. They will both be considered as important, while one of them is clearly redundant and should be eliminated*


****
## Non-Linear Classifiers

Linear methods struggle with **complex data that cannot be separated by straight lines**.
Non-linear classifiers address this issue in two ways

### Polynomial Transformations

Use polynomial functions to model complex boundaries. In a nutshell, it allows to **obtain curves instead of simple straight lines**. 
	*It's a way (in ML) to make linear models capable of learning complex relationships.
	Instead of using `x1​,x2​,…,xn​` inputs, we add combinations of them `w0 + w1x1 + w2x2 + w3x1...`*

> [!success] Advantage
>- Can model complex relationships in the data

> [!fail] Disadvantages
> - Computationally intensive.
> - Prone to **overfitting**, where the model learns noise in the training data and fails to generalise to new data.

### Support Vector Machines (SVM)

**Support Vector Machines (SVM)** find an **optimal hyperplane** that maximizes the margin between classes. For non-linearly separable data, SVMs use the **kernel trick** to transform the data into a higher-dimensional space where it becomes linearly separable.

- **Kernel Trick:** This involves mapping the original data into a higher-dimensional space using functions like polynomial or Gaussian kernels.
    *For example, a 2D dataset can be made linearly separable in a 3D space using a polynomial kernel*

![[kerneltrick.png|550]]

> [!success] Advantages
>- Effective for high-dimensional datasets, such as hyperspectral imagery 
> - Robust to overfitting when the kernel and parameters are chosen correctly

> [!fail] Disadvantages
>- Computationally expensive for large datasets
>- Sensitive to noise and outliers  
>- Choosing the right kernel and tuning parameters can be challenging


****
## Summary

Linear classifiers like LDA and perceptrons are efficient for simple tasks but struggle with complex datasets. 
SVMs with kernel methods provide a robust solution for non-linear problems, making them particularly valuable for remote sensing applications where data is high-dimensional and highly variable.