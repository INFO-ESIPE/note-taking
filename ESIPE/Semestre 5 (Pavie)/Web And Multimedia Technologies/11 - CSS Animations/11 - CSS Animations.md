[[Web And Multimedia Technologies]]
11/11/2024
****
Note: All of those were introduced in CSS3
****
**Table of Contents**
```table-of-contents
```

****
## Transforms

Transformations applies on DOM elements when a specific event happens (hovering an element, clicking...)
Some defaults behaviours exists in native CSS (e.g., `:hover`), but we can manually trigger them in JavaScript:
```js
let box = document.querySelector('.box1');
box.onclick = () => {
  box.classList.toggle('anim1');
};
```

```css
.box1 {
  background: red;
}
.box1.anim1 {
  background: green;
}
```
In this example, the `box1` DOM element will appear green instead of red when clicked.

It is good practice to specify transformations via the `transform` property:
```js
let box = document.querySelector('.box2');
box.onclick = () => {
  box.classList.toggle('anim2');
};
```
```css
.box2 {
  transform: translate(0, 0);
}
.box2.anim2 {
  /* Goes 100px down, and 250px right */
  transform: translate(250px, 100px);
}
```


We can use functions to rotate or upscale an object:
```js
let boxes = document.querySelectorAll('.box4');
boxes.forEach(box => box.onclick = () => {
  box.classList.toggle('anim4');
});
```
```css
.box4 {
  transform: scale(0.5) rotate(0);
  transition: transform 1s;
} 
.box4.anim4 {
 transform: scale(2) rotate(180deg);
}
```


****
## Transitions

Introduces time in stylesheets. When an event happens (usually depends on JavaScript or basic user interactions like mouse hover), it changes the aspect of the object:
```css
div {
	opacity: 1.0;
	transform: scale(1);
	transition: opacity 1.5s, transform 1s;
}
div.anim6 {
	opacity: 0.3;
	transform: scale(1.2);
}
```

We can also specify an interpolation function for those transitions, that changes the transition effect (`linear`|`ease-in`|`ease-out`|`ease-in-out`)
	*Usually to make movement or colour-change looks more natural*
```css
.box8 {
  transform: translate(0px, 50px);
  transition: transform 1000ms linear;
}
.box8.anim8 {
  transform: translate(800px, 50px);
}

/* Define with advanced bézier curves */
.box9 {
  transform: translate(0px, 50px);
  transition: transform 1000ms cubic-bezier(0.42, 0, 0.58, 1);
}
.box9.anim9 {
  transform: translate(800px, 50px);
}
```


****
## (CSS) Animations
*There are also animations that can be made in JavaScript, those will not be detailed here.*
Working draft: [https://www.w3.org/TR/css-animations-1/](https://www.w3.org/TR/css-animations-1/)

Works on two concepts:
- `keyframe`: State of the animation. Defined by the developer at specific timeframes
- `animation`: Indicates how the sequence of keyframes should behave

`@keyframes` allows to specify and name a keyframe, while `animation` animates those keyframes:
```css
/* Will change colour each second, as the entire effect happens on 4s for 4 colours */
@keyframes hello-anim {
  0%   { background-color: red; }
  33%  { background-color: green; }
  66%  { background-color: gold; }
  100% { background-color: red; }
}
.box20 {
  animation: hello-anim 4s infinite;
}
```

We can, of course, use transformations for our animations:
```css
@keyframes hello-anim2 {
  0%   { transform: translate(0px, 0px); }
  25%  { transform: translate(0px, 200px); }
  50%  { transform: translate(0px, 200px) rotateY(180deg); }
  75%  { transform: translate(0px, 100px) rotateZ(180deg); }
  100% { transform: rotateX(180deg) }
}
.box21 {
  animation: hello-anim2 6s linear infinite alternate;
}
```


