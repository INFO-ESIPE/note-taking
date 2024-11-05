[[Web And Multimedia Technologies]]
Cordoba
****
**Note:** We are in Master 2 by the way. Learning HTML so far is wiiiiiiild but whatever mate
****

HTML describes the content of a web page. It derives from Standard Generalised Markup
Language (SGML)
	*It only defines the semantic of the page, and allows for a very few formatting indications*

HTML5.3 is the latest version that was standardised by W3C


****
## Tags

**Tags** defines the structural elements of an HTML document. Most of the time, they are used in pairs (one opens the section with `<...>`, one closes it with `</...>`)
	*Tags are usually in lowercase. They can technically be in uppercase, but since XHTML (old ass technology) requires lowercase, we kept it like so)*
```html
<!DOCTYPE html> <!-- Doctype. Indicates which version of HTML we use (5 Here) -->
<html>
	<head> <!-- Informations about document, link to stylesheet, etc -->
		<title>Welcome Page</title>
	</head>
	<body> <!-- Content of the page -->
		<h1>Hiiiii !</h1> <!-- Title (go from h1 to h6 in DESC importance) -->
		<p>Welcome to this complex class</p> <!-- Paragraph -->
		
		<br> <!-- Line break (no need to close it or to use a pair) -->
		
		<h3>Musical genres I like</h3>
		<ul> <!-- Unordered list -->
			<li>Free Jazz</li>
			<li>Psychedelic Folk</li>
			<li>Harsh Noise Wall</li>
		</ul>

		<h3>My Top 3 best movies</h3>
		<ol> <!-- Ordered list -->
			<li>La Jetée</li>
			<li>Apocalypse Now</li>
			<li>Hana-Bi</li>
		</ol>

		<h3>Albums I like</h3>
		<dl> <!-- Description List -->
			<dt>Wet Land</dt>
			<dd>Very meditative album, for relaxing</dd>
			
			<dt>Down On The Road By The Beach</dt>
			<dd>Summertime album, of a rare beauty</dd>
			
			<dt>Bocanada</dt>
			<dd>Latinamerica has lots of great music, worth looking into it</dd>
		</dl>

		<h3>Preformatted Text Example</h3>
		<pre>
		This is preformatted text. 
					It preserves spaces      and line breaks. 
		
		
		Code snippets or ASCII art are often displayed using the <pre> tag. 
		
		Useful to include code too : 
		if (true == true) { 
			System.out.println("obviously true"); 
		}
		</pre>
	</body>
</html>
```


Elements are organised under a tree shape. This is the **Document Object Model (DOM)**
![[dom.png]]


****
## Text-related tags

| Tag                 | Purpose                          |
| ------------------- | -------------------------------- |
| `<b>` or `<strong>` | Make the text **bold**           |
| `<i>` or `<em>`     | Make the text *italic*           |
| `<small>`           | Makes the text small             |
| `<s>`               | Makes the text ~~Strikethrough~~ |
| `<sup>`/`<sub>`     | Superscript / Subscript          |
| `<code>`            | Showcases code                   |


****
## (Basic) Structural tags

| Tag             | Purpose               |
| --------------- | --------------------- |
| `<section>`     | Section               |
| `<div>`         | Groups elements       |
| `<h1>` - `<h6>` | Titles of the section |
| `<p>`           | Paragraph             |
| `<blockquote>`  | Quoting               |


****
## Attributes

Tags can have one or several attributes :
```html
<body>
	<p title="myparagraph" class="myparagraph" id="paragraph1"> <!-- Written: attributename="value" -->
		blabla...
	</p>
	
	<hr size="4" width="50%"> <!-- Horizontal Ruler to separate content -->

	<p>
		More text...
	</p>
</body>
```

`class` allows to group several elements under a same family
`id` is an unique identifier attributed to a single element
	*Those are especially used for styling (CSS) and dynamic interactions (JavaScript)*


****
## Links

```html
<!-- Jump to a section of a document -->
<a href="#section1">Jump to section 1</a>

<!-- ... -->

<div id="section1"> <!-- Will jump there -->
	<p>
		...
	</p>
</div>


<!-- Link to another page -->
<a href="https://boxd.it/5knAd">
	My Letterboxd account (good taste, better than yours for sure)
</a>

<!-- link to content of the current webserver (can be relative, it's just like a path) -->
<a href="/cats/kitties.html">
	My Letterboxd account (good taste, better than yours for sure)
</a>


<!-- Contact by mail -->
<a href="mailto:noreply@spotify.com">
	Send useless mail
</a>
```


****
## Multimedia

```html
<!-- Simple inclusion -->
<img src="./kitty.jpg">

<!-- Detailed -->
<img src="./penpen.jpg" width="500" height="1000" alt="Image of that lovely penguin from NGE (Good Anime)">
```


****
## Tables

Works with rows and columns, hence the name of tables
```html
<table border="1" cellpadding="10" cellspacing="0">
	<caption>Some movies</caption> <!-- Title of table -->
	<thead>
		<tr> <!-- All elements in a tr will be on the same row -->
			<th>Movie Title</th>
			<th>Director</th>
			<th>Country</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Sonatine</td>
			<td>Takeshi Kitano</td>
			<td>Japan</td>
		</tr>
		<tr>
			<td>Sans Soleil</td>
			<td>Chris Marker</td>
			<td>France</td>
		</tr>
		<tr>
			<td>Man With a Movie Camera</td>
			<td>Dziga Vertov</td>
			<td>USSR</td>
		</tr>
	</tbody>
</table>
```

This is what it looks like (render in markdown) :

| Movie Title             | Director       | Country |
| ----------------------- | -------------- | ------- |
| Sonatine                | Takeshi Kitano | Japan   |
| Sans Soleil             | Chris Marker   | France  |
| Man With a Movie Camera | Dziga Vertov   | USSR    |

**Note :**
`tr` means **T**able **R**ow
`th` means **T**able **H**eadercell
`td` means **T**able **D**atacell


****
## Forms

Forms are used to allows the user to submit data to an endpoint
	*Registration pages, login, edit profile ...*


It is good practice to link a label to an input, we match the `for` property of the label with the `id` property of the input we want to associate it with.
The `name` property is the unique field name, this is useful for the back-end as it has to identify each input

```html
<!-- 
	"action" attribute specifies where we want to send the data to (its an HTTP request).

	"method" specifies what HTTP method to use. HTML only allows "get" and "post", even though several other methods exists for HTTP. Some frameworks (like Laravel) fix this

GET is the code to retrieve data (get a resource on the server)
POST is to submit data the server needs to process (usually a creation form, inserting new data in the database)
-->
<form action="./endpoint" method="post">
	<label for="uname">Username</label>  <!-- Label for "uname" input -->
	<input type="text" id="uname" name="form_uname"> <!-- "uname" text field -->

	<!-- Button that submits the form data to the endpoint ("action") when pressed -->
	<button type="submit">
</form>
```


Other kinds of inputs :
```html
<form>
	 <!-- Password input, with stars hiding the cleartext -->
	<input type="password">

	<!-- Checkbox. If checked, the value will be passed to the backend, else it is ignored -->
	<input type="checkbox" value="Bike">

	<!-- Radio buttons, they exclude each other -->
	<input type="radio" value="My favourite language is Rust">
	<input type="radio" value="My favourite language is Zig">
	<input type="radio" value="My favourite language is Intel x86 asm (???)">

	<!-- Allow user to upload a file of his choice -->
	<input type="file">

	<!-- 
		Pass hidden values, not good practice
		This will not display on the page, but will appear in the devtools
	-->
	<input type="hidden" name="userID" value="12345">

	<!-- Clear all inputs data when pressed, usefull if user has written a whole lot of bullshit and needs to restart from scratch -->
	<input type="reset">
</form>
```


There are other types of fields along with the traditional `input` one :
```html
<!-- Multiple lines input, if user has to write a big paragraph -->
<textarea name="hugeassparagraph"></textarea>

<!-- Dropdown menu -->
<select name="country">
	<option value="fr">France</option>
	<option value="jp">Japan</option>
	<option value="it">Italy</option>
</select>
```


****
## Escape Sequences

To escape values that has a special meaning to HTML 

| Escape     | Equivalent                                                              |
| ---------- | ----------------------------------------------------------------------- |
| `&lt;`     | <                                                                       |
| `&gt;`     | >                                                                       |
| `&amp;`    | & (ampersand)                                                           |
| `&egrave;` | è                                                                       |
| `&eacute;` | é                                                                       |
| `&Egrave;` | È                                                                       |
| `&copy;`   | ©                                                                       |
| `&nbsp;`   | [non-breakable space](https://en.wikipedia.org/wiki/Non-breaking_space) |
