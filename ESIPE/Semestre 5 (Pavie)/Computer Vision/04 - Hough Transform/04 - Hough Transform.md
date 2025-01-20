**Table of Contents**
```table-of-contents
```

****
## Concept

**Hough Transform (HT)** is a method for detecting geometric shapes like straight lines in an image.
This is widely used in the domain of **Pattern Recognition (PR)**

The idea is: Every point in an image space contributes to a set of possible shapes (e.g., lines, circles) in a **parameter space (PS)**. A shape is identified if multiple points in the image space contribute to the same set of parameters in the PS.


***
## Classical line detection

### Line Equation

The standard straight-line equation is: `y = mx + q`

Given a point `(xi, yi)` in the image space, the equation becomes: `q = yi − mxi`
	*This equation represents a family of lines passing through `(xi, yi)` in the **parameter space** `(m, q)`.*

![[straight_line.png|400]]

> [!caution] Issues with `(m, q)`
> Parameters `m` and `q` are not bounded (*-∞ < m, q < +∞*).
> This can cause computational inefficiencies.

### Polar Representation

To overcome the issue mentioned above, we represent lines using the following formula:
`r = x⋅cos(θ) + y⋅sin(θ)`

### Full algorithm

1. Detect edge points in the image (e.g., using Canny Edge Detection).
2. For each edge point:
    - Compute all possible `(r, θ)` pairs corresponding to lines passing through the point.
    - Vote for these `(r, θ)` pairs in an accumulator array in the parameter space.
3. Identify peaks in the accumulator array, which correspond to detected lines.
![[ht_edges.png|500]]


***
## Extensions to Curves

Many HT extensions were developed along the years, mostly for general analytical curves and circle detection.

### Circle detection

For circles with a fixed radius `r`, the equation is: `(x−xc)² + (y−yc)² = r²`
The parameter space becomes 2D `(xc, yc)`. Each edge point votes for possible circle centres `(xc, yc)`.

If the radius is unknown, the parameter space extends to 3D `(xc, yc, r)`.

### General Analytical Curves

HT can be extended to detect any curve defined by a parametric equation:
	`f((x, y), (a1, a2, ...an)) = 0`
where `(x, y)` are image space coordinates and `(a1, a2, ... an)` are parameters.


***
## Generalised Hough Transform (GHT)

### Concept

This extends the original HT in order to detect **any arbitrary shapes**. 
It is particularly useful for:
- Non-parametric shapes
- Rigid transformations (translation, rotation, scaling)

Works like so (given a template shape):
1. Define a **reference point** `Pref(xref, yref)` for the template.
2. For each boundary point `P(x, y)` in the template, compute: 
$$r = \sqrt{(x - xref)^2 + (y - yref)^2} \theta = \arctan\left(\frac{y - yref}{x - xref}\right)$$
3. Store this mapping in a **Reference Table (RT)**.

### Detection

1. Detect edges in the image and compute local properties (e.g., gradient).
2. For each detected edge point, use the RT to vote in the parameter space for potential matches.
3. Peaks in the parameter space indicate the presence of the shape.

### Parameter Space (PS)

For fixed-size objects: 2D PS `(xref, yref)`.
For variable-sized objects: 3D PS `(xref, yref, s)` where `s` is the scale factor.
For rotated objects: 4D PS `(xref, yref, s, θ)`.