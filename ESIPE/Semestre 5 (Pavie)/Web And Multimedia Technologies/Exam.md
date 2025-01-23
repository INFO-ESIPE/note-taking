[[Web And Multimedia Technologies]]
***
**Table of Contents**
```table-of-contents
```

****
## Exam 1

### 1
Say whether the following statements are true (T) or false (F), clearly justifying your answers
	*For the real exam, we have to justify answers, and mind words in *italic* in the question (as we have to explain those words). If the answer is true, also explain why.*

1.  Say whether the following statements are true (T) or false (F), clearly justifying your answers (3.5)
Escape sequences are used to include “special characters” in web pages
**True**, [[08 - HTML#Escape Sequences|explained here]]

2. Colour coding in TV broadcast standards (PAL, NTSC, SECAM) is indirectly based on the RGB model
**True**

HTML5 is a direct evolution of XHTML 1.0
**More false than true**. The kind of question is a bit tricky and won't be at the exam

The spatial resolution of an image is given by its width and height
**False**, as explained [[02 - (Digital) Images#Image Resolution|here]], its an integer number that says how many pixel will be put on paper when the image is printed 

The concept of tweening is specific to vector graphics only
**False**, Tweening is when a software is generating an intermediate state between an initial keyframe and a final keyframe. It works for GIF as well, for instance.

The same MIDI file played on different computers may result in different sound qualities, depending on the device used
**True or false depending on how we understand it**. MIDI is an audio format that encodes musical events (notes played on an instrument, force of the note, length...). MIDI doesn't contain any information on how it should sound, it really depends on a software you open the MIDI file with, the usage of a DAC... but the structure and the events will be the same, as it is the point of the MIDI format.

CGI programs and Application Servers are mostly used for the same purposes
**True**, they can both receive data of a form and store it in a database, or generate dynamic HTML pages.



### 3
Given the following HTML/CSS code, provide a qualitative graphic representation of the resulting web page


C. How would you define a form named `myform` which, using the get method, sends its data to the `process.py` CGI
program (with the `/cgi-bin/` absolute path)?
```html
<form name="myform" action="GET" action="/cgi-bin/process.py">
```


### 4
Define an external style sheet named "mystyle.css" which specifies that: (1) the page has a 30-pixel margin on all
four sides and the size of `<h1>` elements is 3 times the size of the default fonts of their containers; (2) the class gr for
the `<td>` element is characterized by a green background. Indicate (clearly): (a) the content of the CSS; and (b) the
HTML code necessary to \[1] load the external CSS file and to \[2] make a `<td>` tag apply the gr class (2)

```css
body {
	margin: 30px;
}

h1 {
	font-size: 3em;
}

td.gr {
	background-color: green;
	/* background-color: #00FF00; Works too */
}
```
```html
<link rel="stylesheet" ref="mystyle.css">
<td class="gr">
```


### 5
Showing the operations (it is not necessary to actually perform the calculations)

A. The size of the file resulting from the (uncompressed) digitisation of 3 seconds of audio signal (mono, 12 Khz, 16 bits
`3 * 1 * 12000 * 2`
	*1 because its mono, it would be 2 for stereo*

B. The width and height (in inches) of the photograph obtained when printing an image 350 pixels wide and 200 pixels high, with a 200-dpi resolution
`w = 350/200`
`h = 200/200`

C. The size of the file resulting from the (uncompressed) digitisation of 1 minute of video signal (320x240 pixels, 24 bits, 15 fps)
`60 * 320 * 240 * 3 * 15`


### 7
With the main purpose of minimising the size of the resulting file, say whether, to implement the following animations,
it is better to use a bitmap or a vector approach, or whether there is no real difference between them (justify your
answers) (2)

A. Photograph of a cat that moves from left to right (one time)
**Vector**. Even though the frames are bitmaps, the animation itself is better done via vectors.
If we go for a full bitmap approach, each frame has to re-encode all the colours, which is huge. With a vector approach, we simply have to encode the colours once, and then only specify:
- x and y coordinates of initial frame
- x and y coordinates of initial frame
- fps
- number of times the animation is played (in our case, 1)

B. Drawing of a cat that moves from left to right (forever)
**Vector**

C. Sequence of 10 photographs (same size) appearing in the same position at regular intervals
**No diff **


***
## Exam 2
