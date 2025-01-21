**Table of Contents**
```table-of-contents
```

****
## Concept

Mathematical morphology on greyscale images extends the principles used for binary images to handle **greyscale intensities**. A greyscale image is treated as a **3D landscape**, where the intensity (brightness) of each pixel determines the height above a surface.
- **Foreground (F)**: The brighter regions of the image.
- **Background**: The darker regions.
- **Umbra**: The area beneath the "landscape" when viewed as a 3D solid.

### Umbra Properties

The **umbra** of a greyscale function `f(x, y)` is represented as: `U[f] = { (x, y, z) ∣ z ≤ f(x,y)}`

Key properties:
1. $$T[A]⊆A⊆U[A]⊆E^n$$
2. For any function `f : R² → R`, it’s possible to define an equivalent function `g : R³ → {0,1}`, linking greyscale morphology to binary morphology in 3D.

> [!info]
> Try to see it like a sheet casting a shadow in our space.
> ![[umbra.png|450]]


***
## Morphological Operations for Greyscale Images

### Dilation
*See [[05 - Mathematical Morphology#Dilation (Minkowski sum)|this chapter]] first*

Brightens the image by expanding light areas and reducing dark regions.
Defined as: `f ⊕ k = T(U[f] ⊕ U[k])`
    *Computationally equivalent to convolution.*

![[greyscale-dilation.png|450]]


### Erosion
*See [[05 - Mathematical Morphology#Erosion (Minkowski difference)|this chapter]] first*

Darkens the image by shrinking bright areas.  
Defined as: `f ⊖ k = T(U[f] ⊖ U[k])`
	*Computationally equivalent to **convolution** as well.*

![[greyscale-erosion.png|450]]


### Opening and Closing
*See [[05 - Mathematical Morphology#Opening|this chapter]] first*

**Opening**: Smooths the image by eliminating bright details smaller than the structuring element.
    - Rolls a spherical structuring element under the image surface.
    - Removes small bright curvatures.

![[greyscale-opening.png|450]]
![[greyscale-opening2.png|450]]

**Closing**: Fills in dark regions by smoothing valleys smaller than the structuring element.
    - Rolls a spherical structuring element over the image surface.
    - Removes small dark curvatures.

![[greyscale-closing.png|450]]
![[greyscale-closing2.png|450]]
***
Combining **opening followed by closing** for noise removal and smoothing.  
	*Applied to real-world scenarios like astronomical images.*


***
## Advanced Morphological Transformations

### Morphological Gradient

Enhances edges by calculating the difference between dilation and erosion: `Gradient=f ⊕ k − f ⊖ k`
	*Produces a "derivative-like" effect*

![[morph-gradient.png|450]]


### Top-Hat & Bottom-Hat Transformations

**Top-Hat Transformation**: Highlights bright regions smaller than the structuring element: $$That(f)=f−opening(f)$$
**Bottom-Hat Transformation**: Highlights dark regions smaller than the structuring element: $$Bhat(f)=closing(f)−f$$
![[tophat.png|450]]


***
## Granulometry

Granulometry uses **opening operations** of varying sizes to analyse particle distributions in an image.  
Steps:
1. Perform openings of increasing size.
2. Compute the sum (surface area) of pixel values after each operation.
3. Construct a histogram of particle size distribution.

