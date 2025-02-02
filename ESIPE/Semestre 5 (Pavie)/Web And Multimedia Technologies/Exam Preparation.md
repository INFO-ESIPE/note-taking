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

1. Escape sequences are used to include “special characters” in web pages
> [!success]- True
> [[08 - HTML#Escape Sequences|explained here]]

2. Colour coding in TV broadcast standards (PAL, NTSC, SECAM) is indirectly based on the RGB model
> [!success]- True 
> TV broadcast standards (PAL, NTSC, SECAM) is indirectly based on the RGB model, as these systems convert RGB signals into other formats (like YUV or YIQ) for transmission.

3. HTML5 is a direct evolution of XHTML 1.0
> [!fail] More false than true
> The kind of question is a bit tricky and won't be at the exam

4. The spatial resolution of an image is given by its width and height
>[!fail]- False 
> As explained [[02 - (Digital) Images#Image Resolution|here]], its an integer number that says how many pixel will be put on paper when the image is printed 

5. The concept of tweening is specific to vector graphics only
> [!fail]- False
> Tweening is when a software is generating an intermediate state between an initial keyframe and a final keyframe. It works for GIF as well, for instance.

6. The same MIDI file played on different computers may result in different sound qualities, depending on the device used
> [!caution]- True or false depending on how we understand/justify it
> MIDI is an audio format that encodes musical events (notes played on an instrument, force of the note, length...). MIDI doesn't contain any information on how it should sound, it really depends on a software you open the MIDI file with, the usage of a DAC... but the structure and the events will be the same, as it is the point of the MIDI format.

7. CGI programs and Application Servers are mostly used for the same purposes
> [!success]- True
> They can both receive data of a form and store it in a database, or generate dynamic HTML pages.


***
### 2
With reference to bitmap graphics, illustrate the concepts of antialiasing, dithering, and safe palette
- **Antialiasing**: A technique that smooths jagged edges in bitmap graphics by blending pixels with intermediate colours. explained [[02 - (Digital) Images#Vector Graphics|here]]
- **Safe palette**: A standardised set of 216 colours designed to display consistently across different systems and browsers. explained [[02 - (Digital) Images#Graphic Interchange Format (GIF)|here]].
- **Dithering**: A method of simulating unavailable colours by arranging available colours in patterns to create the illusion of a missing colour. explained [[02 - (Digital) Images#Graphic Interchange Format (GIF)|here]]


***
### 3
Given the following HTML/CSS code, provide a qualitative graphic representation of the resulting web page

a. With respect to the directory containing the HTML file defined above, where is the file iae.html?
	The file is contained in the `syllabi` sub-directory **relative** to the directory containing the current HTML file.

b. Supposing that the administrator of the web server has set `c:/internet/web/` as the root directory, what is the path (i.e., the actual position) of the file `unipv.gif` on the server? (`c:/…`)
	The complete path for this file is: `c:/internet/web/img/unipv.gif`

c. How would you define a form named `myform` which, using the get method, sends its data to the `process.py` CGI
program (with the `/cgi-bin/` absolute path)?
```html
<form name="myform" method="GET" action="/cgi-bin/process.py"></form>
```

d. Given the following HTML/CSS code, provide a qualitative graphic representation of the resulting web page
```bash
+---------------------------------------------------+
|                      Header                       |
|              (lightgreen background)              |
|               Computer Engineering                |
+---------+-----------------------------------------+
|         |                                         |
|  Menu   |              Content                    |
| (yellow)|                                         |
|         |  +-----------------------------------+  |
|         |  | The School of Engineering of the  |  |
|         |  | University of Pavia (Italy) offers|  |
|         |  | three international Master of     |  |
|         |  | Science programs...               |  |
|         |  +-----------------------------------+  |
|         |  +-----------------+-----------------+  |
|         |  | [unipv.png]     | The University  |  |
|         |  |                 | of Pavia is one |  |
|         |  |                 | of the oldest...|  |
|         |  +-----------------+-----------------+  |
|         |                                         |
+---------+-----------------------------------------+
```


***
### 4
Define an external style sheet named `mystyle.css` which specifies that: 
	(1) the page has a 30-pixel margin on all four sides and the size of `<h1>` elements is 3 times the size of the default fonts of their containers; 
	(2) the class `gr` for the `<td>` element is characterised by a green background. Indicate (clearly): 
		(a) the content of the CSS 
		(b) the HTML code necessary to 
			(1) load the external CSS file
			(2) make a `<td>` tag apply the gr class
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


***
### 5
Showing the operations (it is not necessary to actually perform the calculations)

a. The size of the file resulting from the (uncompressed) digitisation of 3 seconds of audio signal (mono, 12 Khz, 16 bits
	`3 * 1 * 12000 * 16/8`
	`= 3 * 1 * 12000 * 2`
> [!info]- Details
> - 3: Time in seconds.
>- 1: Mono (1 channel).
>- 12000: Sampling rate (12 kHz).
>- 16: Bit depth (16 bits).
>- /8: Convert bits to bytes.

b. The width and height (in inches) of the photograph obtained when printing an image 350 pixels wide and 200 pixels high, with a 200-dpi resolution
	`w = 350/200`
	`h = 200/200`

c. The size of the file resulting from the (uncompressed) digitisation of 1 minute of video signal (320x240 pixels, 24 bits, 15 fps)
	`60 * 320 * 240 * 24 * 15/8`
	`= 60 * 320 * 240 * 3 * 15`
> [!info]- Details
>- 60: Time in seconds (1 minute).
>- 320×240: Resolution (width x height).
>- 24: Bit depth (24 bits).
>- 15: Frame rate (15 fps).
>- /8: Convert bits to bytes.


***
### 6
With reference to an inkjet printer, what is the difference between print and printer resolutions? Why augmenting
too much the former (while keeping the latter constant) may be counterproductive?
	- "Print resolution" refers to the **dots per inch (DPI)** of the **final printed output**. It determines the quality and sharpness of the printed image or text. Higher print resolution results in finer details and smoother gradients.
	- "Printer resolution" refers to the **maximum DPI capability of the printer hardware**. It defines the printer's ability to place dots on the paper. For example, a printer with a resolution of 1200 DPI can place up to 1200 dots per inch.
	If we increase the print resolution (e.g., upscaling an image) while keeping the printer resolution constant, the printer may not be able to accurately reproduce the additional detail. This leads to **blurring**, **artifacts**, or **loss of quality** because the printer hardware cannot handle the finer details.


***
### 7
With the main purpose of minimising the size of the resulting file, say whether, to implement the following animations,
it is better to use a bitmap or a vector approach, or whether there is no real difference between them (justify your
answers) (2)

a. Photograph of a cat that moves from left to right (one time)
	**Vector**. Even though the frames are bitmaps, the animation itself is better done via vectors.
	If we go for a full bitmap approach, each frame has to re-encode all the colours, which is huge. With a vector approach, we simply have to encode the colours once, and then only specify:
	- x and y coordinates of initial frame
	- x and y coordinates of initial frame
	- fps
	- number of times the animation is played (in our case, 1)

b. Drawing of a cat that moves from left to right (forever)
	**Vector**

C. Sequence of 10 photographs (same size) appearing in the same position at regular intervals
	**No diff**


***
### 8
Answer questions

a. What is the difference between intra-frame and inter-frame compressions in digital video?
	- **Intra-frame:** An algorithm is applied on each frame independently of the others (like you would compress a static image)
	- **Inter-frame:** Acknowledging the content of the surrounding frames (in general, the one immediately before and after), and compress them by analysing the differences between the frames
		*Only the pixels that changes are coded, explained [[06 - Videos#Compression|here]]*

b. Which kind of compression is used in the DV format, and what is its working principle?
	DV relies on intra-frame compression, explained [[06 - Videos#Compression|here]]

c. What "philosophy" is at the basis of recent developments of the MPEG format (MPEG-7 and MPEG-21)?
	I guess its about standardising multimedia content description, interoperability, and accessibility


***
### 9
Given the following HTML/JavaScript code, concisely (but clearly) describe the meaning of each line marked with
a number. Then, also explicate the "general behaviour" of the page

1. `document.body.style.background = col[i];`
	Changes the background colour of the webpage to the colour stored in the `col` array at index `i`. The colour is either "red", "yellow", or "white" depending on the value of `i`.

2. `document.f.display.value = "";`
	Clears the text in the input field with the name `display`. It is executed when `i` is 2, which corresponds to the "Reset" functionality.

3. `<td style="background-color: red" onmouseover="cs(0)">&nbsp;</td>`
	Creates a table cell (`<td>` tag) and applies a red background styling on it. 
	When the user's mouse hovers this cell, the `cs(0)` function is called, changing the page background to red and updating the input field with the colour name.

4. `<a href="javascript:cs(2)">Reset</a><br>`
	Creates a hyperlink labeled "Reset". 
	When clicked, it calls the `cs(2)` function, which resets the page background to white and clears the input field.

5. General behaviour of the page
	- When the page loads, an alert box displays: "Welcome! Here you can test JavaScript!"
	- The page background is yellow (as defined in the `<style>` section).
	- The input field is read-only and displays the current background colour (or is cleared when reset).
	- Hovering over the red table cell changes the page background to red and displays "red" in the input field
	- Clicking the yellow table cell changes the page background to yellow and displays "yellow" in the input field.
	- Clicking "Reset" changes the page background to white and clears the input field.


***
### 10
With reference to the form defined at the previous point 9, add a text field that, when its content changes (and you
click "elsewhere"), the function check() is executed. This function checks the content of the field and: 
	(a) if it is `0`, it substitutes it with the string "null"
	(b) if it is `1`, the background colour of the page is changed to cyan
	(c) if it is `2`, the page text colour is changed to red
	(d) if the content of the field is neither `0` nor `1` nor `2`, a message box with the warning "Wrong value!" is displayed
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<title>A page containing JavaScript code</title>
	<style>
		body {background: yellow}
	</style>
	<script>
		col = new Array(3);
		col[0] = "red"; col[1] = "yellow"; col[2] = "white";
		
		function cs(i) {
		    document.body.style.background = col[i];
		    if (i != 2)
		        document.f.display.value = col[i];
		    else
		        document.f.display.value = "";
		}
		
		function check() {
		    const inputValue = document.f.textField.value; // Get text field val
		
		    if (inputValue === "0") {
		        document.f.textField.value = "null"; // (a) Replace "0" with "null"
		    } else if (inputValue === "1") {
		        document.body.style.background = "cyan"; // (b) Change background to cyan
		    } else if (inputValue === "2") {
		        document.body.style.color = "red"; // (c) Change text color to red
		    } else {
		        alert("Wrong value!"); // (d) Display warning for invalid input
		    }
		}
	</script>
</head>
<body onload="alert('Welcome! Here you can test JavaScript!')">
	<form name="f" id="f">
	    <label for="display">Color:</label>
	    <input type="text" id="display" name="display" readonly size="20"><br><br>
	    <label for="textField">Enter a value (0, 1, or 2):</label>
	    <input type="text" id="textField" name="textField" onchange="check()">
	</form>
	<table width="300" height="100">
	    <tr>
	        <td style="background-color: red" onmouseover="cs(0)">&nbsp;</td>
	        <td style="background-color: yellow" onclick="cs(1)">&nbsp;</td>
	    </tr>
	</table><br>
	<a href="javascript:cs(2)">Reset</a><br>
</body>
</html>
```

*Should work, haven't tested*


***
## Exam 2
