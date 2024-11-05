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
## Boxes (out of class scope)

When it comes to display, entities can be :
- **Inline** (left to right display, like text)
	*`<a>`, `<bold>`...*
- **Blocks** (Takes all width available, and height is the sum of all child element's height)
	*`<p>`, `<ul>`/`<li>`...*


Two tags are important to remember, as they are very convenient to organise and style the HTML :
`<span>` (inline)
`<div>` (block)
```html
<div class="error"> <!-- Used to groups element together -->
  An <span class="important">error</span> occured ... <!-- Isolate -->
</div>
```


There are three main properties allowing to manipulate **boxes** :
- **Padding** (Space inside the box)
- **Border** (Space the border of the box takes)
- **Margin** (Space outside of the box)
![[boxmodel.webp]]


They obviously offer a different render :
```css
.box1, .box2 { 
  background-color: silver; margin: 20px; padding: 10px;
  border-style: dotted dashed solid double;
}
```
![[box.png]]


****
## Floating (out of class scope)

**Floating** is used to make an element go around another

```html
<div class="box">
	Box
</div>
<p>
   Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
</p>
```
```css
.box { 
	background-color: silver; 
	margin: 10px; 
	padding: 20px; 
	float: left; /* "right" also available */
}
```
![[float.png]]


The `clear` property allows to stop the floating display
```html
<div class="box-left">Box1</div>
<div class="box-left">Box2</div>
<div class="box-left">Box3</div>
<div class="box-right">Box4</div>
<div class="box-clear">Box3</div>
```
```css
.box-left { 
	background-color: teal; 
	margin: 10px; 
	padding: 20px; 
	float: left; 
}
.box-right { 
	background-color: silver; 
	margin: 10px; 
	padding: 20px; 
	float: right; 
}
.box-clear { 
	background-color: olive; 
	padding: 20px; 
	clear: both; 
}
```
![[clear.png]]


**The `vertical-align` property can not be used on a floating object**. However, the `display: inline-block;` declaration allows to see an element as an inline, which allows us to now use vertical-align
```html
<div class="box">
	<img src="css3-logo.png" width="50" height="70">
</div>
<div class="box">
	<img src="css3-logo.png" width="100" height="141">
</div>
<div class="box-clear">Box3</div>
```
```css
.box { 
	background-color: teal; 
	border: 1px solid black; 
	display: inline-block; 
	vertical-align: bottom; 
}
.box-clear { 
	background-color: olive; 
	clear: both; 
}
```
![[inline.png]]


****
## Position (out of class scope)

The `position` property allows to place an element on the page :
- `static` - Normal behaviour
	*Top, right, bottom, and left properties do not apply*
- `relative` - Positions the element relative to its original position in the normal document flow
	*Top, right, bottom, and left properties offset the element from where it would normally be*
- `absolute` - Positions the element relative to its nearest positioned (non-static) ancestor or, if none, the document body
	*It **removes the element from the normal document flow***
- `fixed` - Remains attached to the top-left hand corner of the viewport
	*Often used for headers, sidebars, footers...*
- `sticky` - Combines relative and fixed positioning. Element behaves as a `relative` element until it reaches the top-left hand corner of the viewport, and then acts as a `fixed` element
	*Useful if an element has to display correctly, and then sticks to the stop when we scroll down*
