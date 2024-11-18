[[Satellite Data Analysis]]
18/11/2024
****
**Table of Contents**
```table-of-contents
```

****
## Motivation

Useful to convert raster imagery to vector map
Basic image analysis that indicates information

We give partial data as **training data** in order for it to perform the image analysis by itself on the yet untreated part of the data


****
## Semantic Segmentation

*(we call the input data "x")*

The vegetation indicator (NVDI) is obtained by comparing the Red layer and the infrared one
	***Spectral features**, as we obtain this by manipulating the spectral information (not the patterns of the image)*

Analysing patterns present in the data refers to **textural feature**

So, the first thing we do with ML is to transform our input data in our vector feature function (applied on each pixel).


Each element of interest is represented by a class `C`. Elements in a class `C` is called `L`
	*`Ci` = `L1` -> trees
	`Ci` = `L2` -> roofs*

Our output returns a vector (array) of classes.
	*If our task is Image Classification/Categorisation (like google reverse search for images), the output is usually one single class, as we are only interested in identifying one central element in the image (recognising a fish, a cat, a monument).
	This is mostly used in Computer Vision*

If we operate a pixel-level classification, each pixel is assignated a grey-level value that defines to which class it belongs
	*Pixels with a value of 255 belongs to class C1, those of value 0 belongs to CN ...*
So, pixels knows the class they belong to (label). We apply a **pseudocolour** later on to render it properly...

We can have the following classes:
- **Semantic segmentation**: assign each pixel to a class. We don't get any object out of it, just assigned pixels
	*Very useful in remote sensing... This is the task we are interested in for this class*
- **Object detection**: drawing a rectangular shape around each object in the image
	*Used for security systems like CCTV*
- **Instance segmentation**: One step further than object detection, as it tries to identify which pixels inside the rectangle does represent the object itself
	*Used in Computer Vision to isolate precise elements on an image and manipulate them*
- **Panoptic segmentation**: Represent background (and futile) details in pixels (semantic segmentation), and elements of importance as objects (instance segmentation)
	*To distinguish what is important enough to be an object and what isn't, an algorithm (or data samples) are needed. If the segmentation is operated in a parallel behaviour (one thread does the semantic segmentation of futile data, the other thread does the instance segmentation to isolate objects), they might introduce incoherence (details can overlap)*


Based on the vector feature function we applied on a pixel, we obtain a point (x) we can place on a **feature space** of any kind
	*For instance, a 3-dimensional space where x, y and z axis belongs respectively to R, G and B layers.*

By placing all pixels in the feature space, we obtain clusters (usually, pixels are grouped by in a few coordinates).
	*We can obtain classes this way btw, by cluster*


****
## Training

Training is about finding a correct separation in between our classes' elements in the feature space
	*If a cluster is on top of the R axis, and another near the G one, we can split them apart easily*

This is the way we obtain the labels (`Lx`), we can't classify them beforehand as this is pretty much the classification phase.

There are several ways of operating this training correctly.


### Probabilistic methods

The classification is made on the basis of probabilities (or any similar concept).
We can either go with:
- **Generative methods**
- **Discriminative methods**

We consider both `x` features and `C` random variables. We determine distribution of `x` and `C` via a **probability density function** p(x, C), which will do it's job **via training data**. 


The MAP Criterion represents the **probability for classification errors**, which we can determine by analysing values colliding on the graph.
	*If we have several density functions, their Label output (classification) can overlap and show incoherence (values are assigned in two Labels)*

The best way to reduce risk of mismatch in Label assignation is to set a **decision boundary** that is placed at the centre of the collision zone of two probability density functions curves


### Non-probabilistic methods

We predict the class labels without modelling any probability (discriminative classifiers).
We need to find the optical decision boundary (same concept as above), but on the training data.



