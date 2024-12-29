[[Web And Multimedia Technologies]]
21/10/2024
****
**Note :** Slide 19 (DV technology) might be at the exam
****
**Table of Contents**
```table-of-contents
```

****
## Analog

For analog videos (interlaced), there were 3 different broadcast standards :

| Broadcast formats | Countries                                                  | Horizontal lines | Frame rate |
| ----------------- | ---------------------------------------------------------- | ---------------- | ---------- |
| PAL               | Europe (almost all),<br>China, South<br>America, Australia | 625              | 25         |
| SECAM             | France, Middle East,<br>most of Africa                     | 625              | 25         |
| NTSC              | USA, Canada, Japan,<br>Korea, Mexico                       | 525              | 29.97 (30) |


Analog videos were typically using the YCC colour model. This model was chosen — instead of RGB — in order to keep compatibility with black and white TV sets (if RGB was chosen, grey-level television sets could not comprehend RGB-based analog videos at all).

YCC is composed of a luminance level **Y** (defined by R, G and B), and two colour components **CC** :
- C1 = B - Y
- C2 = R - Y
The exact formulas depends on the analog display system :
![[ycc.png]]


Digital videos are, however, non-interlaced, no need to generate half odd and half even fields. **Each frame is displayed as a whole**.


****
## Analog to Digital

We acquire it from analog devices (VHS, Hi-8, BetaSP) and process it in a capture device that will digitalize it.
	*We have to decide for a time sampling (fps), a spatial sampling (resolution), and quantisation (number of bits to represent a colour)*


****
## Compression

Nearly every digitisation implies compression:
	*E.g., 640x480 video, 25 fps, RGB colour (i.e., 24 bits)
	1 second = 640 • 480 • 3 • 25 = 23,040,000 bytes = 22 MB
	1 minute = 22 • 60 = 1,320 MB = 1.3 GB
	1 hour = 1.3 • 60 = 78 GB*

There are two kinds of video compression:
- **Intra-frame :** An algorithm is applied on each frame independently of the others (like you would compress a static image)
- **Inter-frame :** Acknowledging the content of the surrounding frames (in general, the one immediately before and after), and compress them by analysing the differences between the frames
	*Only the pixels that changes are coded*


The **codec (COmpressor/DECompressor)** is the device or algorithm that performs the compression process.

| Format | Resolution | Type of compression | Data Rate      | Usage                |
| ------ | ---------- | ------------------- | -------------- | -------------------- |
| MJPEG  | 720 x 486  | Intra-frame         | 0.5-25 MB/s    | Generic              |
| MPEG-1 | 352 x 240  | Intra-frame         | 0.01-0.06 MB/s | CD-Rom               |
| MPEG-2 | 720 x 480  | Intra and Inter     | 0.01-2 MB/s    | DVD, Digital TV      |
| DV     | 720 x 480  | Intra-frame         | 3.5 MB/s       | Domestic, Industrial |
| D1     | 720 x 486  | Uncompressed        | 25 MB/s        | Broadcast            |


The **codec is not necessarily the same as the format**. The codec says what is actually stored in the file, and the format specifies how it is formatted in the file.
	*But for instance, MPEG-2 can be both the format and the codec, but this is not the case for every format*


We can apply spatial compression by reducing resolution (and quantisation, but this one is not recommended).

We can apply time compression by reducing the frame rate, or by coding changes between frames rather than complete images.
	*in a video containing a still person who is talking, relevant changes will be only in the head area.
	Each frame that is very different from the previous one becomes a
	keyframe and is completely coded*


A compression is said **symmetric** if the compression (when producing the video) is as long as the decompression (by the media player). 
On the other hand, it is called **asymmetric** if the duration varies in one direction or another.
	*In general, low decompression times require high compression times*


Like for static digital images, a compression is judged lossless or lossy depending of if it preserve original information or not.


****
## MPEG Technologies

Motion Picture Expert Group (MPEG) is a standard defined in 1988.
MPEG-2 was the most used format for a long time. It includes 3 kinds of frames :
- Intra-Frames : Frames are coded completely, normally
- Predicted Frames : We only code differences with the previous frame
- Bi-directional Frames : Based on both the direct previous and upcoming frame


MPEG-4 arrived a decade later, and quickly became an international standard
It was mainly designed for digital television and interactive content (WWW)


****
## Codecs
28/10/2024

H.264 is the most widely-used MPEG-4 codec
	*It is pretty much the same as Advanced Video Coding (AVC), with minor differences*

It is suitable to Full HD and 4K definitions, thus explaining their use for Blu-Ray discs.

The motion prediction feature is way more precise than it is with MPEG-2, allowing motion prediction block unit to go from a 16x16 pixel size to a 4x4 one
	*It operates on smaller elements of the image, more precise*

The inter-frame of MPEG-4 is more sophisticated than MPEG-2 one, the latest uses the frame before and the frame after, while MPEG-4 uses two for each direction (4 frames instead of 2) :
![[interframe.png]]


****
## Formats

720p is called **HD Ready**
	*1280x720, progressive*

1080p is **Full HD**
	*1920x1080, progressive
	1080i (interlaced, for CRT TVs) is not Full HD*

**Digital Cinema Initiative 4K (DCI 4K)** is twice the height of a Full HD Frame
	*2160, progressive
	8K Is double the size of 4K ...*


****
## Streaming

Generic concept of streaming (for music, videos ...) means that **our player will download portions of files, and plays it progressively** :
- Pure Streaming (not so common)
	- Uses dedicated protocols (RTSP), and servers that supports it
	- Downloaded files are not saved locally
	- Only for specific formats (MP3, Quicktime ...)
- Progressive Download
	- Behaviour is similar to pure streaming
	- Uses HTTP/HTTPS (DASH Protocol is the common solution for modern players)
	- Lesser consumption of bandwidth


****
## Montage (Editing)

Professional video creation usually follows this scheme:
1. Pre-production
	*Script, storyboard, budgeting, casting, locations selection*

2. Production
	*Actual shooting, usually both the shortest and the most expensive part (unless you are JLG or some avant-garde director*

3. **Post-Production**
	*Montage, special effects, titles*

**Non-linear editing** refers to video/audio **editing carried through a computer software**
	*On the other hand, **linear editing** (related to analog video/audio editing) was **carried on tapes** (with glue, scissors and pencils ...)*
