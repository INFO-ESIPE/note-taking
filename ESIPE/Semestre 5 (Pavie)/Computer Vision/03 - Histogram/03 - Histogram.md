[[Computer Vision]]
****
**Table of Contents**
```table-of-contents
```

****
## Histogram

An histogram **counts how much of each colour occurs in the image**
![[histogram.png|500]]

Since each colour is represented on a byte, it has a value that can go from 0 (`0b00000000`) to 255 (`0b11111111`).

Commonly, histograms (`H[v]`) are calculated separately for each colour channel (e.g., Red, Green, Blue), but they can also merge into a single greyscale channel.
	*So, `H[R]` is the histogram of the Red channel, `H[B]` is for the blue channel, etc*


### Cumulative Function

The cumulative function `C[v]` represents the number of pixels with intensities less than or equal to `v`.
- Calculated as `C[v] = C[v−1] + H[v]`.
- For the 0-255 intensity range, `C[255]` equals the total number of pixels in the image.


****
## Local operation

A local operations **modifies pixel values based only on their initial values, independent of neighbouring pixels**.
	*A typical use is when you invert colours of an image (via a negative filter on Photoshop for instance). You just take a pixel's colour, invert all bits and obtain your result. This does not depend on other pixels at all... Another way of doing this is to do `255 - n`, where 255 is the max possible value, and `n` is the colour we want to invert.*

Usually, we accomplish this via a **Look-Up table (LUT)**, which map each pixel's original value to a new one via a pre-computed array:
`newImage(i,j) = LUT[originalImage(i,j)]`
	*So, the LUT will return the original colour stored at the coordinates i and j*


### Image Equalisation

Equalisation redistributes pixel intensities to achieve a uniform histogram, improving contrast.
	*Often applied to greyscale images but can be extended to colour channels independently.*
![[equalisation.png|300]]

The result looks like this:
![[equalisation curve.png|400]]
![[equalisation result.png|350]]


****
## Automatic thresholding

Thresholding divides the image into regions by intensity:
- **Simple method**: Calculate a threshold based on the average of the means of two regions.
	`T = (m1 + m2) / 2`

- **Otsu's Algorithm**: Optimises the separation between regions by maximising inter-class variance using the histogram.
	`p1(m1-m)² + p2(m2-m)²` where `m` is the mean of the image, and `p<i>` is the probability that a pixel belongs to region `i`


****
## In Java

### Colour representation

We represent a colour on an int (four bytes, expressed in hexadecimal bellow):
	`int pixel = 0xAARRGGBB`

Where:
- `AA`: Most significant byte, represents the alpha channel (0 = transparent, 255 = fully visible)
- `RR`: Red channel
- `GG`: Green channel
- `BB`: Least significant byte, blue channel


### Colour to Greyscale

A greyscale image is an image where the RGB channels are equal. A simple way to convert a coloured image into a greyscale:
`G = (R + G + B) / 3`

The problem is that we don't have the same sensibility for every colour (we perceive green way better, for instance). So, we need to balance the formula like so:
`G = 0.299*R + 0.587*G + 0.114*B`


### Image representation - `BufferedImage`

Default API to handle image manipulation in Java.
- `getRGB(x, y)`: Retrieves pixel value.
- `setRGB(x, y, value)`: Sets pixel value.

```java
try {
	File inputFile = new File("input.jpg"); // Retrieve file
	BufferedImage image = ImageIO.read(inputFile); // Load into BufferedImage

	// Negative filter (inverting colours) in O(n²)
	for (var y = 0; y < image.getHeight(); y++) {
		for (var x = 0; x < image.getWidth(); x++) {
			int pixel = image.getRGB(x, y);

			// Extract colours
			int alpha = (pixel >> 24) & 0xFF; // Alpha channel
			int red = (pixel >> 16) & 0xFF;   // Red channel
			int green = (pixel >> 8) & 0xFF;  // Green channel
			int blue = pixel & 0xFF;          // Blue channel

			// Create negative (as explained in the "Local operation" chapter)
			red = 255 - red;
			green = 255 - green;
			blue = 255 - blue;

			// Obtain negative from this
			int negativePixel = (alpha << 24) | (red << 16) | (green << 8) | blue;

			// Set the new value
			image.setRGB(x, y, negativePixel);
		}
	}

	// Save modified file
	File outputFile = new File("output.jpg");
	ImageIO.write(image, "jpg", outputFile);
} catch (IOException e) { // If a problem occurs
	throw new AssertionError(e);
}
```

### Image representation - Matrix

We convert the image to a 2D array for processing.
Example: Process row per row:
```java
var[][] mat = new int[h][w];
for (var i = 0; i < h; i++) {
	mat[i] = img.getRGB(0, i, w , h, null, 0, w);
}

var pixel = mat[i][j]; // Handle a single pixel
mat[i][j] = LUT[pixel]
```

