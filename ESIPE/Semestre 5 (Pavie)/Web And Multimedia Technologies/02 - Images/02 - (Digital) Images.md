[[Web And Multimedia Technologies]]
07/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Representation

Graphical information can either be:
- Raster (bitmap)
	*GIF, JPEG, PNG, BMP, TIFF....
	Adobe Photoshop, GIMP...
	For photographs, paintings; The image file contains the colour for each pixel.
- Vector
	*SVG, WMF, Or any format that manipulates coordinates, equations, parameters...
	Adobe Illustrator
	For Logos, drawings, diagrams, or favicons*

OS treats the monitor as a 2-dimensional array of pixels, where each pixel has an associated amount of **video memory (bit depth)**:
![[bit depth.png|400]]
> [!info]
> This is a basic example with only on/off pixels. That's why the bit depth is a single bit value.*

**More bit depth = more colours** *(1 byte = 256 colours; 2 bytes = 65,536 colours ...)*
	We consider it to be **true colour** when each pixel has **3 bytes of bit depth** (one byte for R, G and B respectively).

If an image uses a smaller amount of colours (for instance, GIF only uses 256 colours) than true colour, those colours will be defined within a palette called the **Colour LookUp Table (CLUT)**, where only the most important colours will be specified.


It is also possible—but deprecated nowadays—for an image (GIF, JPEG) to be **interlaced**. The GIF will contains several resolutions (from low to normal quality) of itself.
	*The purpose is to give an idea of what the image looks like to people with a poor internet connection.*


****
## Image Resolution

The image file weight depends on both its size in pixels, and the colour levels. Compression is here to lower the size via two methods:
- **Spatial Sampling Rate**: Reducing the number of pixels
- **Quantisation**: Reducing the colours (the number of bits per pixel)
The compression can either be **lossless** (no information lost), or **lossy**.

**Spatial Resolution** is the number of pixels per inch. It is measured in **pixel per inch (ppi)**. This is necessary to foresee the physical size (cm, inch) of a digital image.
	*This means that an image resolution is heavily related to printing.*

The **Print Resolution** represents the number of pixel that will be put on paper for each linear inch (indicated by ppi value).
The **Printer Resolution** is, for inkjet printers, the number of CMYK dots that will be put on paper for each linear inch (the more dots, the better the render is on paper).


****
## Graphic Interchange Format (GIF)

Colours coded on 8 bits, allowing up to 256 colours (strong colour reduction compared to true colour).
	*This colour reduction effect can be "minimised" via a **Dithering process**, where certain pixels in the CLUT are arranged close together to simulate missing colours.*

The files are lossless compressed via **Lempel-Ziv-Welch (LZW)**, where sequences of same-colour pixels are coded on a "number of pixels -> colour" row.
	*This works well for large single-colour areas.*

**GIF89a** allows a colour of the CLUT to be transparent (funny discord GIFs with invisible background):
![[invisible.png|500]]


A web conversion software usually selects the 256 most relevant colours of the image, and place them into the palette. However, the colours will now always be selected from the **System Palette**, defined by the OS:
![[palettes.png|500]]
> [!info]
> 216 of the 256 colours are the same (between Windows and Mac), they are called the **Safe palette**.

We then obtain something similar to this (Macintosh) :
![[conversion.png]]
> [!tip]
> Dithering process is clearly noticeable when you zoom on the converted image, especially in the top left-hand corner.


****
## Image formats

### Joint Photographic Experts Group (JPEG)

Colours coded on 24 bits (true colour), level of compression is selected when saving the file.
**Designed for real images** (photos, paintings), as it offers smooth colour transition (not suitable for drawings or entities with abrupt colour transition).

### Portable Network Graphics (PNG)

Dedicated to the web (normalised by W3C), lossless.
Gray-level coded on 2 bytes, colours on 6 bytes (true colour + an alpha channel for opacity)

### Others

**Web Picture Format (Webp)**: Both images and animations, offers better compression, higher colour depths 
The rest we don't care lol


****
## Vector Graphics

When drawing, any line that is neither Horizontal, Vertical or Diagonal is **jagged**.
**Anti aliasing** is here to reduce this effect:
![[antialiasing.png]]

Unlike for bitmaps, vectors are **Object-Oriented Graphics** displaying information mathematically.
	*Segments, circles, ellipses, curves ...*

Each shape requires parameters in order to be displayed correctly, and they are stored in the graphic file.
	*E.g: A segment requires start_x start_y, end_x and end_y coordinates, along with colour and thickness.*


The advantage of vectors other raster is the possibility to **resize** drawings **without any loss of quality**. Most of the time, the vector file is way smaller than a raster file which should contain a similar content.



