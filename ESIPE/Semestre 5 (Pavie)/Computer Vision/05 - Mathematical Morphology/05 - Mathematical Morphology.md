**Table of Contents**
```table-of-contents
```

****
## Concept

French concept which purpose is to analyse and process geometric structures in images.
At its core, it uses **set theory**, **logic operators**, and **geometric principles** to extract meaningful structures from images.


***
## Basic Operations

Major ones:
- **Translation**: Move a set `A` by a vector `t`. $$At={c∈E^n∣c=a+t,a∈A}$$
- **Reflection**: Reflect a set `A` about the origin. $$Ar={c∣c=−a,a∈A}$$
- **Complement**: Invert the elements of `A`. $$Ac=E^n−A$$

***
## Morphological Operations

### Dilation (Minkowski sum)

Dilation expands the shape of an object. The formula is: $$A⊕B={c∈E^n∣c=a+b,a∈A,b∈B}$$
- `B`: Structuring element (e.g., a small shape like a circle or square).
- **Properties**:
    - Translation invariance.
    - Associativity.
    - `A ⊕ B = B ⊕ A`

> [!example]-
> - `A` = Original set.
> - `B` = Structuring element.
> - `A ⊕ B = A` = Enlarged set.
>
> ![[dilation.png|450]]

### Erosion (Minkowski difference)

Erosion shrinks the shape of an object. The formula is: $$A⊖B={c∈E^n∣c+b∈A,∀b∈B}$$
- Removes boundary pixels based on the structuring element.

> [!example]-
> ![[erosion.png|450]]
> ![[erosion-multi.png|450]]

### Duality Theorem between them

Dilation and erosion are opposites (a.k.a. "duals") of each other when combined with the concept of complements.

> [!tip] Here's a simpler way to think about it:
>1. **Dilation** makes shapes **grow** by expanding them outward.
>2. **Erosion** makes shapes **shrink** by reducing them inward.
>3. The theorem says that:
  >  If you dilate a shape and then take its complement, it's the same as taking the complement of the shape first, flipping it (reflecting the structuring element), and then eroding it.


***
## Contours

### Opening

Smooths object contours by first eroding and then dilating: $$O(A,B)=(A⊖B)⊕B$$
- Removes small objects or noise.
- **Idempotent**: Reapplying it doesn't change the result.

### Closing

Smooths object contours by first dilating and then eroding: $$C(A,B)=(A⊕B)⊖B$$
- Fills small holes or gaps.
- **Idempotent** too.

> [!tip] Tip - How to visualise both
> Closing "pushes" the structuring element over the image's outer contour.
> Opening "pushes" the structuring element under the inner contour.

### Hit-or-Miss Transform

Suitable for identifying isolated points or features in an image by finding specific patterns in an image.
It searches for a shape (foreground) and its complement (background) simultaneously, using a pair of structuring elements:
![[hit-or-miss.png|425]]

### Boundary Extraction

Instead of detecting patterns like above, here we are interested in highlighting—and extracting—continuous edges.
We subtract the **eroded version** of the image from the original image. The erosion shrinks the object, and subtracting it leaves only the edge pixels.

However, there are two types of contour we can obtain:
- **Internal Contour** (left): `A − (A ⊖ B)`
- **External Contour** (right): `(A ⊕ B) ∩ A^c`
![[contour-ie.png|450]]

We can combine both in order to obtain the **Double Contour**: `(A ⊕ B) ∩ (A ⊖ B)`
![[contour-double.png|450]]


***
## Distance Transform

The **Distance Transform (DT)** labels each pixel with its distance from the nearest background pixel. It works like so:
1. Initialise `R=∅` (evolving image).
2. Perform:
    - Add current pixels to `R`.
    - Erode the object.
3. Stop when the object vanishes.

![[distance-transform.png|450]]
> [!info] 
> For clarity and to easily identify any pattern, we apply a different colour for each distance label (which increases as we go inwards)

### Medial Axis Transform (MAT)

MAT is derived from the DT. It captures the **skeleton** of an object by identifying the points that are **maximally distant** from the boundary. 
These points represent the **medial axis** or "skeleton" of the object.

![[dt-mat.png|400]]

In essence:
1. Compute the DT
2. Extract the **ridge points** of the DT, which are the points that define the medial axis
    *A point is on the medial axis if its distance value is a **local maximum** in the DT*


***
## Propagation and Recursive Techniques

### Connected Component Propagation

Used to propagate labels across connected regions:
1. Start with a marker pixel `X0` inside the region
2. Iteratively expand using dilation: `X = (X ⊕ B) ∩ A`

![[recursion.png|425]]

### Distance Between Points

To find the shortest path between two points `X` and `Y`:
1. Initialise `A = {X}` (evolving image).
2. Expand `A` iteratively: `A = (A ⊕ K) ∩ F`
3. Stop when `Y ∈ A`.


***
## Convex Hull

The **Convex Hull** is the smallest convex polygon containing all points of an object:
- Algorithm propagates along 8 orientations.
- Combine results using logical operations to form the hull.

![[hull.png|250]]
> [!info]
> Used to simplify object representation, or for geometric analysis.

