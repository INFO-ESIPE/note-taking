[[Computer Vision]]
***
**Table of Contents**
```table-of-contents
```

****
### What is the Hough Transform?

**Feature extraction technique to detect shapes** that can be parameterised mathematically.
It works by **mapping points in the image space** to a parameter space and **identifying accumulations of points** in the parameter space, which represent potential shapes in the image

### Hough Line vs. Hough Circle

For detecting lines with the Hough Transform, two parameters (ρ, θ) are required:
- ρ: Distance of the line from the origin.
- θ: Angle of the line relative to the origin.
However, for detecting circles, three parameters are needed (x,y,r):
- x: x-coordinate of the circle's center.
- y: y-coordinate of the circle's center.
- r: Radius of the circle

### How Hough Circle Detection Works

- If the radius (r) is fixed or given, the algorithm searches for the center of the circle (x,y) in a 2D parameter space (the space of potential circle centres).
	This means that the algorithm is essentially looking for points in a 2D plane that could be the centres of circles with the specified radius.

- If the radius (r) is unknown, the parameter space becomes 3-dimensional (3D) because the algorithm now needs to search for (x,y,r):
	The mapping role in this 3D space forms a cone because the radius adds a new dimension to the parameter space.

### Erosion

Erosion reduces object size, removing details.

**Definition**: Erosion is a fundamental operation in morphological image processing. It was initially designed for binary images and later extended to greyscale images and complete lattices.
**Purpose**: It uses a structuring element to probe and reduce shapes in the input image. The operation shrinks the boundaries of objects in the image by eroding (removing) pixels from the edges.
**Result**: Erosion minimises the size of foreground objects in an image, often useful for separating connected components.
**Applications**:
- Reducing noise in binary images.
- Removing small details while preserving the structure of larger objects.

### Dilation

Dilation increases object size, filling gaps.

**Definition**: Dilation is another basic operation in mathematical morphology. It was initially developed for binary images and later extended to greyscale images and complete lattices.
**Purpose**: It uses a structuring element to probe and expand shapes in the input image. The operation adds pixels to the edges of objects, effectively "growing" them.
**Result**: Tends to brighten images by reducing dark regions and filling gaps in object boundaries.
**Applications**:
- Filling small holes and gaps.
- Connecting disjoint objects in an image.
- Smoothing object boundaries.

Together with erosion, they form the basis of other morphological operations like opening (erosion followed by
dilation) and closing (dilation followed by erosion).

### Opening

Removes small foreground objects/noise.
Obtained by doing **Erosion → Dilation**

**Definition**: Opening is a **morphological operation that smooths object contours** in an image, removing small objects or noise while preserving the shape and size of larger structures.
**How It Works**:
1. Erosion is applied first, which removes pixels from the edges of objects.
2. Dilation follows, which restores the size of the objects but without the small noise that was removed during erosion.
**Purpose**: Removes small objects/noise from the foreground (e.g., white pixels) in a binary image. Separates connected components.
**Applications**:
- Cleaning noise while retaining the main structure.
- Preparing an image for segmentation

### Closing

Fills small background holes or gaps in the foreground.
Obtained by doing **Dilation → Erosion**

**Definition**: Closing is a morphological operation that fills small holes or gaps in the foreground and connects broken edges or lines.
**How It Works**:
1. Dilation is applied first, which expands the boundaries of objects.
2. Erosion follows, which reduces the size of the dilated objects back to their original size while keeping the gaps filled.
**Purpose**: Closes small holes or gaps in the foreground. Connects fragmented or disjoint structures.
**Applications**:
- Filling small gaps in objects or lines.
- Smoothing contours while keeping larger holes intact.

### Line Detection (Hough Transform)

**Concept**: Each edge point in an image maps to a cosine curve in the Hough space (ρ, θ). When all edge points are mapped, a lot of cosine curves are generated in the Hough space.
**Key Insight**: If multiple edge points belong to the same line in the image, their cosine curves will intersect at a specific (ρ,θ) in the Hough space.
**Hough Transform Algorithm**: Detects lines by identifying (ρ,θ)pairs with intersections exceeding a certain threshold.
These intersections correspond to the strongest lines in the image.
**Purpose**: This process helps detect straight lines in noisy images where direct edge-based methods fail.

### Canny Edge Detector (CED)

**Steps**:
1. Filter the image using the derivative of a Gaussian to reduce noise and smooth the image.
2. Compute the magnitude and orientation of gradients to identify edges.
3. Apply Non-Maximum Suppression:
	*Thin wide ridges (multiple pixels) into single-pixel edges for precision.*
4. Linking and Thresholding (Hysteresis):
	Use two thresholds:
		▪ High threshold to identify strong edges.
		▪ Low threshold to connect weak edges that are adjacent to strong edges.
**Additional Notes**:
- Used for processing binary images (black and white).
- Helps retain critical edges while removing noise.
- Convolution (e.g., Gaussian filter) is applied for smoothing, and other filters can be used to refine results.

### 3/9 Operator

**Definition**: An operator that analyses relative grey-level intensities to detect edges.
**Usage**: Used for edge detection, particularly in low-contrast regions.
**Features**: Sensitive to noise, requiring careful threshold selection.

### Compass Operator

**Definition**: An edge detection operator that calculates gradients in multiple directions.
**Usage**: Used for detecting edges in specific orientations.
**Features**: Provides more directional information than other gradient-based operators.

### Kirsh’s Operator

**Definition**: An edge detection operator that computes gradients in a 3x3 neighbourhood in all directions.
**Usage**: Used for detecting edges with strong directional sensitivity.
**Features**: Computationally intensive but effective for highlighting directional edges


### Filters
#### Box Filter

**Definition**: A linear filter that replaces each pixel in an image with the average of its neighbourhood pixels.
**Usage**: Used for image smoothing by removing high-frequency noise.
**Features**: Simple and fast but can blur edges and reduce image details.

#### Gaussian Filter

**Definition**: A smoothing filter that applies a Gaussian function to reduce high-frequency noise and smooth images.
**Usage**: Frequently used in pre-processing steps for edge detection and image de-noising.
**Features**: Preserves edges better than a box filter due to the weighted average approach.

#### Sharpening Filter

**Definition**: A filter designed to enhance image edges by accentuating the intensity differences.
**Usage**: Used in applications requiring clearer edges and object boundaries.
**Features**: Increases the sharpness of images but can amplify noise.

#### Sobel Filter

**Definition**: A discrete differentiation operator that calculates gradients in the horizontal and vertical directions.
**Usage**: Used for edge detection to find contours in images.
**Features**: Robust to noise, but not ideal for detecting fine edges.

#### Prewitt Filter

**Definition**: A simpler alternative to the Sobel filter, calculating gradients with equal weights.
**Usage**: Edge detection, particularly in scenarios where computational simplicity is preferred.
**Features**: Produces similar results to the Sobel filter but is less sensitive to noise.

#### Laplacian Filter

**Definition**: A second-order derivative operator that highlights regions of rapid intensity change.
**Usage**: Used for edge detection and identifying regions of high spatial frequency.
**Features**: Does not consider edge direction and is often used with a smoothing step

#### Median Filter

**Definition**: A non-linear filter that replaces each pixel with the median value of its neighbourhood.
**Usage**: Effective in removing salt-and-pepper noise.
**Features**: Preserves edges while reducing noise.

#### Average Value Filter

**Definition**: A simple filter that replaces each pixel with the average value of its neighbourhood.
**Usage**: Used for image smoothing and noise reduction.
**Features**: Blurs fine details and edges

#### Nagao-Matsuyama Filter

**Definition**: A filter that selects the mean value of the neighbourhood with the least variance. It creates 9 subgroups in the neighbourhood and keeps the subgroup with the smallest variance. 
**Usage**: Used for reducing noise while preserving edges and textures.
**Features**: Computationally expensive but effective for complex scenes.

#### Difference of Gaussians (DoG)

**Definition**: A filter obtained by subtracting two Gaussian-blurred versions of an image.
**Usage**: Used for edge detection and feature enhancement.
**Features**: Efficient and captures edges across different scales.

#### Weighted Moving Average Filter

**Definition**: A smoothing filter that uses weighted averages of pixel neighbourhoods.
**Usage**: Used for noise reduction while maintaining more detail than a simple average filter.
**Features**: More flexible as weights can be customised for specific purposes.

