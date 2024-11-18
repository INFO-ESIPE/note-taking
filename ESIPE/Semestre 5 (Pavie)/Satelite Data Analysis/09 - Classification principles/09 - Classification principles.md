[[Satellite Data Analysis]]
idk lol
****
**Table of Contents**
```table-of-contents
```

****
## What is classification ?

Classification in remote sensing involves assigning each portion of the Earth's surface to a predefined **class** (e.g., land cover types like water, vegetation, or urban areas). 
The output is typically a **thematic map**, providing actionable insights from satellite data.

This is useful in Machine Learning in various situations:
- Involves optimisation techniques to minimise error metrics like OA.
- Adequate training data is essential for supervised methods.

To obtain decent classification, it is mandatory to provide a **reliable training** to our algorithm.

Example of classification:
![[classification.png]]

*Note that (Remote sensing) classification overlaps with **semantic segmentation** in computer vision.*


### Key Concepts

**Land Cover (LC) & Land Use (LU)**:
-  **LC**: The physical materials covering the Earth's surface (e.g., grass, water).
-  **LU**: The human-defined usage of the land (e.g., agriculture, residential).

**Classification Unit**: The smallest portion of data assigned to a class, often a pixel or a group of pixels (superpixel).

The **Classifier** is what assigns LC/LU to each classification unit:
![[classifier.png]]

**Hard vs. Fuzzy Classification**:    
-  **Hard**: Each pixel belongs to one and only one class.
![[hard.png]]

- **Fuzzy**: Pixels can partially belong to multiple classes with assigned probabilities.
![[fuzzy.png]]

****

