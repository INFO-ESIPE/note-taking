[[Computer Vision]]
****
**Table of Contents**
```table-of-contents
```

****
## Convolution and basic things around it

Convolution is a **linear operation** applied to an image to extract features or modify it, such as smoothing, sharpening, or detecting edges. It involves applying a **kernel (filter)** to the image, where each pixel is replaced by a weighted combination of its neighbours.

**Edge detection** allows to **Identify sudden changes** (discontinuities) in an image.
	*rapid change in the image intensity*
![[intensity.png]]

**Segmentation** decomposes the image in segments (components) **based on a given homogeneity criteria**. 
	*(chromatic, morphologic, motion, depth ...)*

Segmentation often relies on edge detection to work (as it's a good way to segment things and detect area changes...).


Segmentation is a **binarisation process** as it allows to detect an individual object (foreground) by separating it from the rest (background). We then obtain **Binary images**. 
	*Some applications are by nature binary: black and white printing, blueprints for mechanical parts and architecture, bio-imagery...
	But sometimes it's more complex as it contains grey levels (fades) instead of sudden changes: area illumination, shadowing, noise on a camera...*


### Applications of Convolution

1. **Image Smoothing**: [[02 - Shape Extraction#Noise|Reducing noise]] or preparing for further analysis.
2. **Feature Extraction**: [[02 - Shape Extraction#Border search|Detecting edges, corners, or patterns]].
3. **Enhancement**: Improving image quality by sharpening or adjusting contrast.

By understanding convolutions and filters, we can pre-process images effectively for tasks like object recognition, segmentation, and texture analysis.


****
## Thresholding

A simple way to apply our binarisation process is to use a **Bimodal distribution** A.K.A. thresholding. 
The thresholding is **applied on the [[03 - Histogram#Histogram|histogram]]** by dividing the pixels into two groups: **those below the threshold** (background) and **those above it** (foreground).
	*Can be done manually or via dedicated methods like Otsu's algorithm*

More details [[03 - Histogram#Automatic thresholding|here]]


****
## Filters

We call the **Kernel** a **small matrix used in image processing filters**. It defines weights applied to a pixel and its neighbours during convolution to produce effects like blurring, sharpening, or edge detection. 
	*The matrix is usually of a 3x3 or 5x5 size*


### Box Filter

Simple blur-like filter that **averages the pixel values in a neighbourhood**.
> The goal is to remove sharp features, which results in a smoothing effect.

![[boxfilter.png]]

Kernel:
```
1 1 1
1 1 1
1 1 1
```


### Gaussian Filter

blur-like filter that **weighs pixel contributions by their distance**, giving more weight to closer pixels.
> The goal is to smooth the image while preserving **transitions better than a box filter**.

![[gaussianfilter.png]]

Kernel example:
```
0.003 0.013 0.022 0.013 0.003
0.013 0.059 0.097 0.059 0.013
0.022 0.097 0.159 0.097 0.022
0.013 0.059 0.097 0.059 0.013
0.003 0.013 0.022 0.013 0.003
```


### Sharpening Filter

Filter that **highlights edges and differences with local averages**.

![[sharpeningfilter.png]]


### Edge Detection Filter

Detect abrupt changes in intensity. More information [[02 - Shape Extraction#Border search|on the next chapter]], as it relies in Sobel operator.

We get two kernels, one for the vertical edges (1), and one for horizontal edges (2):
![[vertical.png]]
![[horizontal.png]]


****
## Border search

As mentioned above, basic way to retrieve border (edge detection) is via discontinuity of an image
	*Gap between grey level of two pixels/areas, depth...*

**Methods:**
1. **Sobel and Prewitt operators** (gradient-based), approximate gradients to find edge direction and magnitude.
![[sobel-example.png]]
	- The gradient is calculated as:
![[sobel.png]]

2. **John Canny operator**, in a few steps:
	1. Smooth image with a Gaussian filter.
	2. Compute gradient magnitude and direction.
	3. Apply non-maximum suppression to thin edges (reduces them to single-pixel width).
	4. Use hysteresis thresholding to finalise edges.
		*We define two thresholds: `low` and `high`. We use the `high` one to start the edge curve, and the `low` threshold to continue them*


****
## Difference of Gaussians (DoG)

DoG is a technique for edge detection that **highlights areas in an image where intensity changes significantly**. It's commonly used in feature extraction and object recognition as **the produced contours are closed**.

It works like this:
1. **Apply Gaussian Blur**:
    - Smooth the image using two Gaussian filters with different standard deviations (`σ1`​ and `σ2`​).
    - `σ1 < σ2`​, so one filter smooths less than the other.

2. **Subtract the Blurred Images**:
    - Subtract the image blurred with `σ2` from the image blurred with `σ1`​: 
	    `DoG(x, y) = Gσ1(x, y) - Gσ2(x, y)`
    - The result emphasises edges where intensity changes rapidly.

3. **Threshold the Result**:
    - Apply a threshold to the DoG output to highlight significant edges while suppressing noise.


This works fine, but the presence of **noise** can significantly alter the render. We need a way to handle it...


****
## Noise

There are various types of noises. We will visualise each on the following image:
![[noise-original.png]]

- **Salt and pepper noise**: Random occurrences of black and white pixels
![[noise-saltpepper.png]]

- **Impulse noise**: Random occurrences of white pixels (so it's like the one above but without the black pixels)
![[noise-impulse.png]]

- **Gaussian noise**: Variations in intensity drawn from a Gaussian normal distribution
![[noise-gaussian.png]]

- **Uniform noise**: Constant probability density in a given range
![[noise-uniform.png]]
	*Slides are shit, I had to take another picture lol, but you get the idea*

### How to handle noise

- **Average Filter**:
    - Reduces noise by replacing each pixel with the **mean** of its neighbours.
    - Doesn't work well on some noises (e.g., Salt and Pepper)

Given the following 3x3 neighbourhood:
```
3 6 8
3 4 2
5 8 3
```

The pixel at the centre will take the following value:
`(3+6+8+3+4+2+5+8+3)/9 = 4.67`


- **Median Filter**:
    - Replaces a pixel with the **median** of its neighbours.
    - **Advantage**: Removes salt-and-pepper noise while preserving edges.
![[median.png]]

Given the following 3x3 neighbourhood:
```
3 6 8
3 4 2
5 8 3
```

The correspondent values are:
![[median-res.png]]


- **Nagao-Matsuyama Filter**:
    - Chooses the region with the least variance to calculate the new pixel value.
    - **Use case**: Smooths noise while preserving edges, effective for complex scenes.
    - Costly in terms of computation workload, but offers a clean render

Computation works as it follows (for a pixel):
1. We make **nine sub-groups** (in red) for the pixel, each one handles the neighbourhood differently
![[nagao.png]]

2. Select the sub-group with the least variance
3. Calculate the mean of this group, and assign it's result to the centre pixel (the one we wanted to compute)
