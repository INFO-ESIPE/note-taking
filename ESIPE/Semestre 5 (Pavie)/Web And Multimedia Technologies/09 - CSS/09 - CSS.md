[[Web And Multimedia Technologies]]
Cordoba
****

CSS is used to change the styling (appearance) of the HTML
CSS can be applied to an HTML file in several ways :
1. Style tag in `head` **(bad practice)**
```html
<html>
	<head>
		<title>My page</title>
		<style>
			h1 { color: blue }
		</style>
	</head>
	<body>
		<!-- ... -->
	</body>
</html>
```

2. Directly on a tag, with the `style` attribute **(bad practice)**
```html
<body>
	<p style="color: blue">
</body>
```

3. Linking an external stylesheet (File containing CSS code). Good practice, allows several HTML files to share the same style
```html
<html>
	<head>
		<title>My page</title>
		<link rel="stylesheet" href="mycss.css">
	</head>
	<body>
		<!-- ... -->
	</body>
</html>
```


Example of CSS code :
```css
/* Affects all h1 tags */
h1 {
	color: red;
	margin: 2em;
}

/* Affects all h2 and h3 tags */
h2, h3 {
	color: #abc012 !important; /* Important means it will override all priorities */
	text-transform: uppercase;
}

/* Only affects h2 tags inside the body tag */
body h2 {
	text-align: left;
}

/* 
Styles are applied in cascade :
 - The last modification matters
Even thought we specified h1 to be red above, they will ultimatelty appear in green

 - A style is applied to all it's sub-elements (in the DOM). It is recommended to be precise when applying a style, else it will impact a lot of elements
*/
h1 {
	color: green;
}

h2 {
	color: blue; /* Here, even though we specified a change of colour later in the file for h2 tags, if will remain of colour #abc012 as we attached an "!important" declaration to it. It is better to avoid using this feature as it is overall bad practice */
}
```
- `color: green;` is a **declaration**
- `h1`, `body h2` are **selectors** (the tags we affect)
- `text-align`, `color`, `margin` are **properties**
- `red`, `left`, `2em` are **values**


When a page is rendered by the user's web browser, it can have 3 origins :
- **Author**'s CSS (the one we code for our own website)
- **User-Agent** (web browser), which gives basic styling for tags that were not defined in the author's CSS
	*This creates inconsistency in the page's rendering, as browsers has different ways of rendering tags by default. To avoid this, developers often use a **CSS Reset** to render everything the same*
- **User**'s CSS (rare, but a user can define his own styling if he wants)

**The author CSS has priority over the two others**


****
## Variables

We can store values in variables (useful if we use them often)
```css
:root { /* Global variables, can be used everywhere in the CSS file */
	--color1: #1e90ff;
	--color2: #ffffff;
}

body { 
	background-color: var(--color1); /* We pass it in the var() function */
}

h2 { 
	border-bottom: 2px solid var(--color2); 
}


div {
	--mycolor: red; /* Local variable, not very useful but still exists */
	color: var(--mycolor);
}
```


****
## Selectors

(Combination of) tags we will apply styling to :

| Selector                                     | Aiming at                                                          |
| -------------------------------------------- | ------------------------------------------------------------------ |
| `p`                                          | All paragraphs of the HTML document                                |
| `div p`                                      | All paragraphs placed in a div                                     |
| `div > p`                                    | All paragraphs which has a div as their direct father (in the DOM) |
| `p#paragraph1`                               | Paragraph with `id="paragraph1"`                                   |
| `p.myparagraph`                              | All paragraphs which belongs to `class="myparagraph`               |
| `p[foo]`                                     | All paragraphs which have a `foo` attribute defined                |
| `p[foo="bar"]`                               | All paragraphs which have the `foo=bar` attribute                  |
| `p::first-letter`                            | First letter of all paragraphs                                     |
| `p:first-line`                               | First line of all paragraphs                                       |
| `div:first-child`                            | First element encapsulated by all divs (in the DOM)                |
| `div:last-child`                             | Last element encapsulated by all divs (in the DOM)                 |
| `a:link`, `a:visited`, `a:hover`, `a:active` | State of a link                                                    |
| `h1::before { content: "-->" }`              | Append the string "-->" before all `h1` tags                       |

The aforementioned selectors with `:` are **pseudo-classes** natively defined in CSS.
	More details on the state of links later


****
## Colours

Several ways to apply colours :
- Pre-defined colours with keywords
	*maroon | red | orange | yellow | olive | purple | fuchsia | white | lime | green | navy | blue | aqua | teal | black | silver | gray*

- Hex
	*`#012ABC` | `#FFF`*

- RGB function
	*`rgb(255, 255, 255)` (full white)
	Can also be specified in percentages : `rgb(0%, 100%, 0%)` (full blue)*

- RGBa function (RGB with opacity)
	*`rgba(255,0,0,0.3)` (red with 30% opacity*


Define colours and background :
```css
h1 {
	color: #a2eb53; /* Text colour */
	background-color: aqua; /* Colour of the background (entire zone of the elt) */
}

div.mybackground {
	background-image: url("images/kitties.jpg"); /* Background from an image */
	background-repeat: no-repeat; /* Only one instance of the image */
	background-attachment: scroll; /* Follows as we scroll */
	background-position: 20px 30px; /* 20px from left, 30px from top */ 
}
```


****
## Length units

| Unit           | Behaviour                                                                             |
| -------------- | ------------------------------------------------------------------------------------- |
| %              | Percentage of the parent element if it has one<br>Otherwise, percentage of the screen |
| px             | Pixels                                                                                |
| em             | Relative to the current font size (or parent)<br>By default, 16px                     |
| ex             | Height of the 'x' character from the font                                             |
| ch             | Width of the '0' character from the font                                              |
| rem            | Font size of the **r**oot **e**le**m**ent                                             |
| vw             | 1% of the **viewport** (browser window) width                                         |
| vh             | 1% of the viewport height                                                             |
| vmin           | 1% of the smallest viewport dimension                                                 |
| vmax           | 1% of the largest viewport dimension                                                  |
| mm, cm, in ... | Absolute units                                                                        |


****
## Text Properties

Text decoration :
```css
h1 {
	text-decoration: line-through; /* none, underline, overline available */
	text-align: left; /* center, right and justify available */
	text-indent: 30px; /* Indent at the beginning of the paragraph */
	line-height: 1.6;
	text-transform: uppercase; /* none, lowercase and capitalize available */
}
```


Font :
```css
p {
	font-family: "Arial", sans-serif; 
	font-size: 10em; /* Can be in other units */
	font-weight: bold; /* Boldness level can be indicated from 100 to 700 as well */
	font-style: italic; /* Normal or oblique available */
}
```

**Tip**: It is better to define font size in `em` rather than in `px`, as it avoids problems when the screen resolution changes


****
## List and Tables

For lists `ul` and numbered lists `ol`, we can change the display of elements/numbering like so :
```css
ul {
	list-style-type: circle;	
}

ol {
	list-style-type: upper-alpha;
}

/* Others available: none, disc, square, decimal, lower-alpha, upper-roman, lower-roman */
```


We can add borders to tables (or `th` and `td`) :
```css
table { 
	border-collapse: collapse; /* Melts collapsing borders into one only */
}

table, th, td { 
	border: 1px solid black; 
}
```
![[border.png]]


The size of a table is calculated by it's elements, unless we use the `width` and  `height` properties
```css
table { 
	width: 100%; 
	border: 1px solid black; 
}

th { 
	height: 30px; 
} 

td { 
	padding: 3px; 
}
```
![[size.png]]


By default, text is horizontally aligned on the left. We can change this behaviour via the `text-align` property we mentioned earlier
```css
table { 
	width: 100%; 
	border-collapse: collapse; 
}

th { 
	border-bottom: 1px solid black; 
}

.col1 { 
	text-align: center; 
}

.col2, .col3 { 
	text-align: right; 
}
```
![[align.png]]


We can also vertically align the text in cells, and add a different colouring on half of the bands in order to facilitate reading (via the `:nth-child` class)
```css
table { 
	border-collapse: collapse; 
}

th { 
	height: 40px; 
	background-color: orange; 
	vertical-align: center; 
}

th { 
	background-color: orange;
	border-bottom: 1px solid black;
}

tr:nth-child(odd) { /* "even" is also available */
	background-color: lightgray; 
}

.col2, .col3 { 
	text-align: right; 
}
```
![[final.png]]


****
## CSS3

CSS3 adds new features

**Borders for boxes:**
```css
div {
	border: 2px solid #a1a1a1;
	padding: 10px 40px;
	background: #dddddd;
	width: 300px;
	-moz-border-radius: 25px; /* Only applies to firefox */
	-webkit-border-radius: 50px; /* Only applies to chrome & safari */
	-ms-border-radius: 75px; /* Only applies to IE */
	-o-border-radius: 100px; /* Only applies to Opera */
	border-radius: 25px;
}
```

**Background size and origin:**
```css
div {
	background: url(img_flwr.gif);
	background-size: 80px 60px;
	background-repeat: no-repeat;
}
div {
	background: url(“img_flwr.gif”);
	background-repeat: no-repeat;
	background-size: 100% 100%;
	background-origin: content-box;
}
```

**Special effects for text:**
```css
h1 {
	text-shadow: 5px 5px 5px #FF0000;
}
```
![[text.png]]

**Font via files:**
Custom fonts can be added to the web server. Those will be automatically downloaded via an HTTP request when the user loads a page.
	*In the past, websites only used to use `Times` and `Arial`, as they were the only widely-used fonts that every web browser had by default.*
```css
@font-face {
	font-family: myFirstFont;
	src: url('Sansation_Light.ttf'),
	url('Sansation_Light.eot'); /* IE9+ */
}

div {
	font-family: myFirstFont;
}
```

**Column display:**
```css
.newspaper {
	column-count: 3;
	column-gap: 40px;
}
```
![[columns.png]]

**Media Queries for Responsive Design (can be found [[10 - CSS Display#Responsive Design|here]])**

**Animations and transitions (can be found [[11 - CSS Animations|here]])**

