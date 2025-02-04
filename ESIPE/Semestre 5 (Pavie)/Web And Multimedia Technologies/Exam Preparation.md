[[Web And Multimedia Technologies]]
***
**Table of Contents**
```table-of-contents
```

****
## Exam 1

### 1-1
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
> As explained [[02 - (Digital) Images#Image Resolution|here]], its defined by "ppi", an integer number that says how many pixel will be put on paper when the image is printed 

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
### 1-2
With reference to bitmap graphics, illustrate the concepts of antialiasing, dithering, and safe palette
- **Antialiasing**: A technique that smooths jagged edges in bitmap graphics by blending pixels with intermediate colours. explained [[02 - (Digital) Images#Vector Graphics|here]]
- **Safe palette**: A standardised set of 216 colours designed to display consistently across different systems and browsers. explained [[02 - (Digital) Images#Graphic Interchange Format (GIF)|here]].
- **Dithering**: A method of simulating unavailable colours by arranging available colours in patterns to create the illusion of a missing colour. explained [[02 - (Digital) Images#Graphic Interchange Format (GIF)|here]]


***
### 1-3

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
### 1-4
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
### 1-5
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
### 1-6
With reference to an inkjet printer, what is the difference between print and printer resolutions? Why augmenting
too much the former (while keeping the latter constant) may be counterproductive?
	- "Print resolution" refers to the **dots per inch (DPI)** of the **final printed output**. It determines the quality and sharpness of the printed image or text. Higher print resolution results in finer details and smoother gradients.
	- "Printer resolution" refers to the **maximum DPI capability of the printer hardware**. It defines the printer's ability to place dots on the paper. For example, a printer with a resolution of 1200 DPI can place up to 1200 dots per inch.
	If we increase the print resolution (e.g., upscaling an image) while keeping the printer resolution constant, the printer may not be able to accurately reproduce the additional detail. This leads to **blurring**, **artifacts**, or **loss of quality** because the printer hardware cannot handle the finer details.


***
### 1-7
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
### 1-8
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
### 1-9
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
### 1-10
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
### 2-1
Say whether the following statements are true (T) or false (F), clearly justifying your choices

1. In the context of XML, DTDs and XML Schemas have the same function
> [!success]- True
> As explained [[10 - XML Meta Language#XML Validation|here]], both DTDs and XML Schemes serves the purpose of ensuring validity of a file.
> An XML document is **considered valid if it implements our XML language correctly** (being well-formed is not enough).
> XML Schemas are a more verbose and sophisticated alternative to DTDs, but they serve the same goal.

2. CSS absolute positioning always arranges elements with respect to the browser's window
> [!fail]- False
> As explained [[9.1 - CSS Display#Position|here]], the `absolute` property arranges elements with respect to the **nearest positioned ancestor** (an ancestor with `position: relative`, `absolute`, `fixed`, or `sticky`).

3. SVG is a language for the description of vector graphics
> [!success]- True
> explained [[02 - (Digital) Images#Representation|here]]

4. The RGB colour model is more suitable for warm colours, while the CMYK model is more suitable for cool colours
> [!fail]- False, I think?
> In this [[01 - Colours#Rendering|chapter]], we can see both model's gamut. It appears the RGB gamut has a wider range for green (cool colour), while CMYK is wider for orange (warm).

5. The colour lookup tables of any two GIF images have always at least one colour in common
> [!caution]- True if they use the safe palette, False otherwise
> The Colour Lookup Table used by GIFs (as each colour is stored on a single byte, not true colour) might use the content of the system palette (provided by the OS). System palettes shares 216 colours in common, this is known as the safe palette. 

6. DHTML is a new HTML, with more tags than HTML5
> [!fail]- False
> DHTML only refers to a combination of technologies used to create interactive and animated websites.

7. Multimedia interfaces are always multimodal interfaces
> [!fail]- False
>Multimedia interfaces involve multiple forms of media (e.g., images, audio, video...) to present information or interact with users.
>Multimodal interfaces, on the other hand, allow users to interact with the system using multiple modes of input and output (e.g., speech, gestures, touch, gaze...).
>A multimedia interface does not necessarily support multiple modes of interaction.


***
### 2-2
What is the purpose of the Colour Management System (CMS) in software tools like GIMP or Adobe Photoshop?
How does it work?

As explained [[01 - Colours#Colour Management System (CMS)|here]], A CMS is here to **standardise colour representation on** different devices which might use a **different colour models**.
It does it by defining a **virtual colour space** that gives an "absolute" colour representation. It simply translates colours from model `a` to model `b` via an intermediate colour space (usually CIE L\*a\*b) 


***
### 2-3

8. With respect to the directory containing the above defined HTML file, where is the file `ee.html`?
The file is contained in the `syllabi` sub-directory **relative** to the directory containing the current HTML file.

9. Supposing that the administrator of the web server has set `c:/myroot` as the root directory, what is the path (i.e.,
	the actual position) of the file `unipv.gif` on the server? (`c:/…`)
The complete path for this file is: `c:/myroot/unipv.gif`

10. Is `<p style="font-weight: bold">` semantically equivalent to `<strong>`?
No. 
- The `<strong>` tag is used to indicate that the enclosed text is of strong importance. Screen readers and other assistive technologies interpret `<strong>` as important text.
- The `<p style="font-weight: bold">` tag only applies a visual style to make the text bold but it is purely aesthetic.

11. Given the following HTML/CSS code, provide a qualitative graphic representation of the resulting web page 
```bash
+---------------------------------------------------+
|                                                   |
|  +------------------------+  +-----------------+  |
|  |                        |  |                 |  |
|  |  [University Logo]     |  |  Description of |  |
|  |                        |  |  MSc Programs   |  |
|  +------------------------+  +-----------------+  |
|  |                        |                       |
|  |  Historical Note       |                       |
|  |                        |                       |
|  +------------------------+                       |
|                                                   |
|  +------------------------------------------------+
|  | © Copyright 2025                                |
|  +------------------------------------------------+
|                                                   |
+---------------------------------------------------+
```


***
### 2-4

Define an external style sheet named `mystyle.css` which specifies that: 
	(1) the page has a 25-pixel margin on all four sides and a black background
	(2) the class `ita` for the `<td>` element is characterised by italic text (value italic for the font-style property). Indicate (clearly): 
		(a) the content of the CSS
		(b) the HTML code necessary to load the external CSS file and to make a `<td>` tag apply the `ita` class

```css
body {
	background-color: #000;
	margin: 25px;
}

td.ita {
	font-style: italic;
}
```
```html
<link rel="stylesheet" ref="mystyle.css">
<td class="ita">blabla in italic</td>
```


***
### 2-5
Showing the operations (it is not necessary to actually perform the calculations), indicate:

a. The size of the file resulting from the (uncompressed) digitisation of 5 seconds of audio signal (stereo, 15 Khz, 16 bits)
	`5 * 15000 * (16/8) * 2`
- `16/8` = Bit Depth (2 bytes)
- `2` = Number of channels (Stereo, so 2)

b. The width and height (in pixels) of the image obtained from the scan of a photograph 5 inches wide and 7 inches high, with a 600-dpi resolution
	`w=5*600 = 3000 pixels`
	`h=7*600 = 4200 pixels`

c. The size of the file resulting from the (uncompressed) digitisation of 3 minutes of video signal (640x480 pixels, 24 bits, 25 fps)
	`(3 * 60) * 640 * 480 * (24/8) * 25`


***
### 2-6
Explain the following assertion: "The RGB and CMY colour models are complementary" 

The RGB and CMY colour models are considered complementary because:
1. **Additive vs. Subtractive Mixing**
	In the RGB model, combining red, green, and blue light at full intensity produces white (additive mixing)
	In the CMY model, combining cyan, magenta, and yellow pigments at full intensity produces black (subtractive mixing)
2. **Colour Inversion**
	The colours in the CMY model are the "inverse" of the RGB model. For example:
    - Cyan is the absence of red
    - Magenta is the absence of green
    - Yellow is the absence of blue


***
### 2-7
In a certain area of a web page there is the photo of a cat that moves from left to right and vice versa; 
	when the photo (not a region external to it) is clicked, another page is loaded. 
Say (explaining why) which of the following technologies can be exploited to achieve such a behaviour and which ones cannot.

a. HTML
> [!fail]- No
> This area can be surrounded by an `<a>` tag which, when clicked, redirects to the page that is pointed by the `href` property. The problem is, HTML by itself cannot manage the animation of the image, nor the movement of this link area.

b. CSS
> [!fail]- No
> Can handle the animation of the cat photo moving from left to right and vice versa using keyframes or transitions. But CSS alone cannot handle the click event to load another page.

c. JS
> [!success]- Yes
> Can handle both the animation and the click event

d. Animated GIFs
> [!fail]- No
> An animated GIF can display the cat photo moving from left to right and vice versa. However, GIFs do not support interactivity like click events. Clicking on the GIF will not trigger any action unless combined with HTML (e.g., wrapping the GIF in an `<a>` tag) or JavaScript.

e. CGI programs or application servers
> [!fail]- No
> CGI programs are for back-end code execution. Everything described here is purely front-end-oriented


***
### 2-8
How does the audio digitization process occur? What parameters must be taken into account? Can the MP3
format be considered somewhat similar to the MIDI format? Why?


***
### 2-9
Given the following HTML/JavaScript code, concisely (but clearly) describe the meaning of each line marked with
a number. Then, also explicate the "general behavior" of the page

1. `document.getElementById("im").src = img[cur_im];`
	Updates the `src` attribute of the `<img>` element with the ID `im` to the image file specified in the `img` array at the index `cur_im`. It changes the displayed image to the next one in the sequence.

2. `document.location.href = addr[cur_im];`
	Redirects the browser to the URL specified in the `addr` array at the index `cur_im`. It loads the webpage associated with the currently displayed image.

3. `<a href="javascript:loadP()">`
	Creates a hyperlink that, when clicked, executes the JavaScript function `loadP()`. The function redirects the user to the URL associated with the currently displayed image.

4. `<input type="button" value="Change" onclick="n()">`
	Creates a button labeled "Change". When clicked, it executes the JavaScript function `n()`, which updates the displayed image to the next one in the sequence.

5. General page behaviour
	- The page displays an image (initially `google.gif`) inside a clickable hyperlink
	- When the image is clicked, the browser redirects to the URL associated with the currently displayed image (e.g., Google, Bing, or Yahoo)
	- A "Change" button is provided below the image. Clicking this button cycles through the images in the sequence (`google.png`, `bing.png`, `yahoo.png`)
	- The image and associated URL are updated in a cyclic manner, and clicking the image navigates to the corresponding website


***
### 2-11
What is XSLT? Is it necessarily connected with the concept of web browser? Provide at least one example of its
use

As explained [[10 - XML Meta Language#XSL|here]], XSLT is used to transform XML documents into other formats, such as HTML, plain text, or even XML documents implementing another language.
It applies a set of rules (defined in an XSLT stylesheet) to an XML document. These rules specify how the XML data should be converted into the desired output format
While XSLT can be used in web browsers to transform XML into HTML for display, it is also widely used in other contexts (e.g., Document generation, server-side processing, data integration...).



***
### 2-12
The entity house is informally described as follows: “a house is characterised by an address, a floor, a total size (in
square meters), a size for each room and a price”. 
To formally describe the entity through an XML language, define proper tags and attributes, and then use them to represent the house located in Milan, via Verdi n. 10, 4th floor, 100 m2 in total, with a kitchen (25 m2), a living room (20 m2), a bedroom (20 m2), another bedroom (23 m2), and a bathroom (12 m2), price 450,000 Euro. 
Directly apply the tags and attributes to the example, and only then define the DTD. Important: use at least one attribute

Pretty much what we did [[10 - XML Meta Language#Example - XML & DTD|here]]

