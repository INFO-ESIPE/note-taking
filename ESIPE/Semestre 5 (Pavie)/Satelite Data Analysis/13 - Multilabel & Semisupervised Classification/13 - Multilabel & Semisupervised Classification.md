[[Cloud Computing]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Multilabel Classification

### Concept

Our problem is that, in real scenarios, elements aren't labeled to one and only one class. They belong to several.
	*k-NN does not work well here, as the average number of class in practice examples is often too high for this classifier to output a decent result*

In multilabel classification, examples can belong to multiple classes simultaneously, unlike traditional single-label classification.
This introduces challenges to overcome:
- **Computational cost:** Training multiple classifiers can be resource-intensive
- **Imbalanced classes:** Some labels may have significantly fewer examples, making them harder to classify
- **Mutual dependence:** Classes may influence each other, complicating predictions

Several techniques can be used to achieve multilabel classification.

### Binary Relevance

The most common approach. We simply **create a separate binary classifier for each class**.
	*If there are 3 classes (A, B, C), train one classifier each for "is A?", "is B?", and "is C?".*
![[binary relevance.png|500]]

> [!caution] Limitations
> - This approach does not account for dependencies between classes
> - It can be computationally expensive if there are many classes
> - It struggles with imbalanced datasets

### Classifier Chains

Here, classifiers are sequentially linked, with **each classifier using predictions from the previous one**. This is called a **Hard Decision**, opposed to a **Soft Decision** (probabilities).
	*The order of classifiers is determined by the user, and each classifier in the chain depends on the predictions of the previous classifiers*
![[chain.png|300]]

> [!success] Advantages
> - Works well if the classes follow a logical sequence.
> - Can capture dependencies between classes.
>   

> [!fail] Disadvantages
>- **Errors can propagate through the chain**, as each classifier depends on the predictions of the previous ones.
>  (First classifiers are not always correct. Since the next classes depends on those, they will also contain mistake)
>- The performance heavily depends on the order of the classifiers.

### Stacking

We use a two-layer structure where predictions from the first layer feed into the second layer.
Upper classes takes the original attribute vector as input, while the second layer is based on the first layer's output*

![[stacking.png|300]]

> [!success] Advantages
> - More flexible, as any top-layer class can influence any lower-layer class
> - Can capture complex dependencies between classes

> [!fail] Disadvantages
> - May introduce unnecessary dependencies between classes.
> - Can be **computationally expensive**.


****
## Semi-supervised Learning

### Concept

Traditional classifiers rely solely on labeled data for training. **Semi-supervised learning** offers a hybrid approach that **combines labeled and unlabeled data** to improve classification performance.
	*Opposite of supervised learning that relies solely on labeled samples for the training.*

This technique avoids the need for a large labeled dataset, which can be costly and time-consuming to create.

**Steps**:
1. Start with a small labeled dataset and a large unlabeled dataset
2. Use the labeled data to train a supervised classifier
3. Predict labels for the unlabeled data and retrain the classifier iteratively

![[semisupervised.png|700]]

> [!success] Advantages
>- Reduces the need for labeled data
>- Can improve classification performance by leveraging unlabeled data
  
> [!caution] Challenges
>- Requires careful selection of examples for labeling (e.g., using [[13 - Multilabel & Semisupervised Classification#Active Learning|Active Learning]])
>- Assumes that the unlabeled data follows the same distribution as the labeled data

### Active Learning

Active learning is a strategy where **the model queries for labels on the most uncertain examples from the unlabeled set**, refining its decision boundary efficiently.

![[learning.png|500]]

A few strategies can be used to achieve this:
- **Random Sampling (RS)**: Pick examples at random
- **Breaking Ties (BT)**: Choose examples with the smallest probability gap between the top two predicted classes
- **Marginal Sampling (MS)**: Focus on examples close to the decision boundary

> [!success] Advantages
>- Reduces the amount of labeled data needed 
>- Improves model performance by focusing on the most informative examples

> [!caution] Challenges
>- Requires an oracle (e.g., a human annotator) to provide labels for the queried examples  
>- The performance depends on the quality of the query strategy
