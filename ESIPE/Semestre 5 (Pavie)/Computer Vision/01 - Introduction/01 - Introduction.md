[[Computer Vision]]
01/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## What is Computer vision ?

Computer Vision is a field that enables machines to interpret and analyze visual information from the world, mimicking human visual perception. It encompasses both the **analysis** (understanding visual data) and **synthesis** (creating or altering visual data).
- **Analysis**: Deriving information from images or video (e.g., detecting objects, faces, or patterns).
- **Synthesis**: Generating or transforming images (e.g., image editing, 3D modelling).


Computer vision is connected to several other disciplines:
- **Graphics**: While Computer Graphics focuses on creating visual content, Computer Vision focuses on understanding it. The two fields often complement each other.
- **Physics and Mathematics**: Underpinning principles like optics, geometry, and calculus are essential for algorithms.
- **Artificial Intelligence**: Modern applications heavily rely on machine learning and neural networks.


Computer vision applies in a lot of domains:
- **Healthcare**: Medical imaging and diagnostics (e.g., MRI, ultrasound analysis).
- **Autonomous Vehicles**: Object detection and scene understanding.
- **Security**: Face recognition, license plate reader and surveillance systems.
- **Augmented Reality (AR)**: Real-time overlay of information onto physical environments.
- **Retail**: Image-based product search and inventory management.


Understanding Computer Vision is essential for solving real-world problems involving visual data. It combines theoretical principles with practical applications, offering tools to enhance how machines perceive and interact with the world.


****
## Key Concepts

Image representation depends on the following concepts:
- **Electromagnetic Spectrum**: Vision extends beyond visible light, leveraging other spectrums (infrared, ultrasound...) for applications such as thermal imaging and medical scans.

- **Spatial Resolution**: Refers to the smallest detail distinguishable in an image (pixels)
	*So, the number of pixels per length unit (inches, centimetres). Usually measured in **pixels per inch (ppi)**
	Higher resolution means finer detail. More details [[02 - (Digital) Images#Image Resolution|here]]*
![[resolution.png]]

- **Colour Depth**: Number of bits used per pixel to define colours. Common formats:
    - Binary (1 bit per pixel, black/white).
    - Greyscale (e.g., 8 bits for 256 shades).
    - RGB (24 bits per pixel, ~16 million colours).
	    *One byte for R, G and B makes 3 bytes. This allows to represent **true colour**, even though it is theoretically possible to have RGB on one byte only. See [[01 - Colours|this class]] and [[02 - (Digital) Images|this one]] for more details*
![[colour.png]]
	*Here, the first image is an RGB, the last image is a binary, and the others are greyscale*

- **Compression**: Images can be lossless (e.g., GIF) or lossy (e.g., JPEG), with trade-offs in file size and quality.
	*We compress a file by either reducing the number of pixels (spatial resolution) or the number of colours represented (colour depth)*


****
## Colour Models

As explained above, digital colours is represented on three channels: Red, Green and Blue. When those three channels are assembled, we obtain the colour image.
	*Each channel has an entire byte if we achieve **true colour** (best representation of colours). However, we can reduce this number if we don't mind losing precision of the colours*

RGB is the most common colour model, but others exists:
- **CMY(K)** – Cyan, Magenta, Yellow, Black
	*Usually for printers, it is complementary with the RGB model*
- **HSV/HSB** - Representing hue, saturation, and value/brightness
- **YIQ**: Used in television systems for luminance and chrominance separation.

Again, more details in [[01 - Colours|this class]]


****
## Image file formats

The image is stored as a file. Different formats are available to represent the image:
- **BMP**: Uncompressed format with a header, LUT (look-up table), and pixel values.
- **PGM/PPM**: Portable greyscale/pixel map images with simple ASCII headers.
- **GIF**: Lossless compression for simple animations, limited to 256 colours.
- **JPEG**: Lossy compression with configurable quality, often used for photography.
- **PNG**: Supports transparency through an alpha channel, commonly used for web graphics.

Others are available (e.g., WEBP), but they aren't used for our field.

