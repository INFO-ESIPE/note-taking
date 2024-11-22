[[Cloud Computing]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## Multilabel Classification

Our problem is that, in real scenarios, elements aren't labeled to one and only one class. They belong to several.
	*k-NN does not work well here, as the average number of class in practice examples is often too high for this classifier to output a decent result*

In multilabel classification, examples can belong to multiple classes simultaneously, unlike traditional single-label classification.
This introduces challenges to overcome:
- **Computational cost:** Training many classifiers can be resource-intensive.
- **Imbalanced classes:** Some labels may have significantly fewer examples.
- **Mutual dependence:** Classes may influence each other, complicating predictions.

One can achieve multilabel classification through several techniques.

### Binary Relevance

The most common approach. We simply **create a separate binary classifier for each class**.
	*If there are 3 classes (A, B, C), train one classifier each for "is A?", "is B?", and "is C?".*
![[binary relevance.png]]

This technique, however, fails to overcome the challenges we defined above. 

### Classifier Chains

Here, classifiers are sequentially linked, with **each classifier using predictions from the previous one**. This is called a **Hard Decision**, opposite to a **Soft Decision** (probabilities).
	*From left to right. The user is responsible of the ordering*
![[chain.png]]

This works well if the classes follows a logical sequence, but this is far from always being the case. Furthermore, the biggest drawback of this technique is that it **propagates errors**
	*First classifiers are not always correct. Since the next classes depends on those, they will also contain mistake*

### Stacking

We set up a two-layer structure where predictions from the first layer feed into the second.
	*Upper classes takes the original attribute vector as input, while the second layer is based on the first layer's output*
![[stacking.png]]

This solution is more flexible (any top-layer class can influence any lower-layer class), but on the other hand, this also sometimes forces unnecessary dependencies between classes.


****
## Semi-supervised Learning

While traditional classifiers use labeled data only (for their training), semi-supervised learning offers an **hybrid approach combining labeled and unlabeled data** to improve classification performance.
	*Opposite of supervised learning that relies solely on labeled samples for the training. *

This technique avoid collecting a lot of labeled data (which is costly and time-consuming).

it follows those steps:
1. Start with a small labeled dataset and a large unlabeled one
2. Use the labeled data to train a supervised classifier
3. Predict labels for the unlabeled data and retrain iteratively

This works well, but still comes with a few issues:
- Requires careful selection of examples for labeling (e.g., [[13 - Multilabel & Semisupervised Classification#Active Learning|Active Learning]])
- Assumes that unlabeled data follows the same distribution as labeled data

### Active Learning

Active learning is a strategy where **the model queries for labels on the most uncertain examples from the unlabeled set**, refining its decision boundary efficiently.
![[learning.png]]

A few strategies can be used to achieve this:
- **Random Sampling (RS)**: Pick examples at random
- **Breaking Ties (BT)**: Choose examples with the smallest probability gap between the top two predicted classes
- **Marginal Sampling (MS)**: Focus on examples close to the decision boundary
