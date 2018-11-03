
# CSS Variables, Without Preprocessors

CSS custom properties are variables that have specific values to be used throughout the document. They are created using double dashes (`--`) and accessed using the `var()` function. For example:
```
:root {
	--theme-color: black; /*declaration*/
}

.example {
	color: var(--theme-color); /*usage*/
}
```
```html
<p class="example">
	Lorem ipsum, dolor sit amet consectetur adipisicing elit...
</p>
```
It's recommended to declare CSS custom properties in the CSS `:root{}` as it targets the highest-level element in the DOM (<HTML>), ensuring that the variables can be used anywhere else in the document through cascading.

CSS values tend to repeat in large applications. One downside of this is if you change a property you will have to do a global find and replace, or worse, look for them manually. 

CSS variables changes this by having a single point of declaration that can be referenced throughout the code, and which you only need to change once. Secondly, these custom variables can add semantic value to the code because you can name them relative to the value it holds or other specific identifiers.

The first CSS variable to be added to the specification was `currentColor`. It refers to the current colour of the element and can be used with any declaration that uses a `color` property.
```css 
.block {
	color: slategray;
	border-color: currentColor; /*this will be slategray*/
}
```
## Not Just Colours
Custom properties can be used to store many different things, not just colours.
```css
:root {
	--radius: 20px;
}

.block {
	border-radius: var(--radius);
}
```

You can now change the `--radius` properties individually and not have to worry about updating it for all other instances. Sure, you can do this in preprocessor like LESS and SASS, but with CSS variables you can use this directly in your HTML too.  

```html
<div class=”block” style=”--radius: 30px;”>
	Lorem ipsum dolor...
</div>
```

In this case CSS variables work just the same as normal CSS properties, but it also allows you to dynamically change your styles outside the CSS file.

For CSS custom properties to work they must be declared inside a selector. In addition, the element using the custom property must be a child of the selector on which it was declared. It may not be immediately obvious that one of these is wrong as no errors will be shown when your CSS is parsed.
This is different from preprocessors such as SASS where variables can be declared outside of a selector, and if invalid it throws a compile error. 

## Inheritance
Custom properties are inherited by default. This means that if a custom property for outline is defined on a parent element, the child elements will inherit this property also. This is because custom properties have higher precedence than `initial`, even for selectors that have no specificity (`*`).

For example,
```html 
<div class="example-1">
	<div></div>
</div>
```

```css
* {
	outline: var(--outline);
}

.example-1 {
  --outline: .2em solid;
  height: 200px;
  width: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.example-1 div {
   height: 100px;
   width: 100px;
   background: blue;
}
``` 
<img src="https://image.ibb.co/dnw7GL/Screen-Shot-2018-10-20-at-12-12-00.png" alt="Screen-Shot-2018-10-20-at-12-12-00" border="0" />

You can easily override this behaviour by setting `--outline: initial;` in `*`. This says, if the element does not have a value, set it to initial. This means that the child element will no longer have an outline.

Example
```html
<div class="example-1">
	<div></div>
</div>
```
```css
* {
  --outline: initial;
	outline: var(--outline);
}

.example-1 {
  --outline: .2em solid;
  height: 200px;
  width: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.example-1 div {
   height: 100px;
   width: 100px;
   background: blue;
}
```
<img src="https://image.ibb.co/m4QEVf/Screen-Shot-2018-10-20-at-12-16-04.png" alt="Screen-Shot-2018-10-20-at-12-16-04" border="0">

## The var() Function & Default Values
The  `var()` function also accepts a default value as its second argument. 

```css
background: var(--accent-color, orange);
```

In this case, if `--accent-color` is not given a value when created, it will default to orange. The `var()` function also accepts other `var()` function as fallbacks.

```css 
var(--color1, var(--color2));
```

It's worth noting that if the custom variable is set with a value CSS does not understand (i.e `--color:100deg;`), it will not default to the value passed in as the second argument. 

The order of fallback will change depending on certain conditions.
In the example below it could easily be assumed that the background colour of the div `.example-1` would default to a colour of orange, seeing that orange is passed in as a second argument. However, that is not the case as the `div` does not actually have a background colour.
```html
<div class="example-1">
</div>
``` 
```css 
div {
  --bg-color: 100;
  bakground-color: --bg-color;
}

.example-1 {
  border: .2em solid black;
  height: 200px;
  width: 200px;
  background-color: var(--bg-color, orange)
}
```

<img src="https://image.ibb.co/muARbL/Screen-Shot-2018-10-20-at-12-19-42.png" alt="Screen-Shot-2018-10-20-at-12-19-42" border="0">

The fallback value is only used if the custom property value has not been set. In the example above, the value is indeed set (`--bg-color: 100;`), but not to a valid CSS property value. Given its cascading mechanism, for CSS variables to work in a scenario such as this (it has parsed the file and the custom property value does not exist or is invalid), it will always fallback to its default value. In the case of `background-color` the default value will be `transparent`.  

This type of error is called an [‘invalid at computed-value time’](https://www.w3.org/TR/css-variables-1/#using-variables) error and will  default to the same value you would get if the property was defined as `initial`.

## Browser Compatibility

CSS Custom properties are well supported in modern browsers.
<img src="https://preview.ibb.co/dfRMAf/Screen-Shot-2018-10-20-at-11-37-16.png" alt="Screen-Shot-2018-10-20-at-11-37-16" border="0">

However, you can easily check for support in your CSS using the following declaration,
```css 
@supports(--css: variables){
	//your styles
}
```
Styles within this declaration will only get process if the browser supports CSS variables.

## Single property Mixins
CSS variables also lets you create single property mixins with prefilled values. In the below example you can change the box-shadow directions, but the colour will always be purple.

```css 
* {
	--purple-shadow: initial;
	box-shadow: var(--purple-shadow) purple;
}

.example {
	/*example's box-shadow default to purple*/
	--purple-shadow: -0.3em 0.5em 1.6em; 
}
```

## CSS Variables & JavaScript
The same JavaScript methods you use with normal CSS properties can be used with CSS variables, including,  `getPropertyValue()`, `getComputedStyle().getProperty()` and `setProperty()`.

Some use cases would include,
```javascript
/*get variable if inline*/
element.style.getPropertyValue(--custom);

/*get variable if inline or otherwise*/
getComputedStyle(element).getPropertyValue('‘--custom'); 

/*set variable value if inline*/
element.style.setProperty('--custom, 10 + 5); 
```

This opens up a new realm of possibilities and makes it very simple to update CSS properties via JavaScript and still keep a great order of separation.
In the example below I am setting a property of `--value` on an input of range and changing the gradient percentage based on the range input.

HTML
```html
<input type="range">
```
JS
```javascript
for (let input of document.querySelectorAll('input')) {
	input.style.setProperty('--value', input.value);
}

document.addEventListener('input', evt => {
	let input = evt.target;
	input.style.setProperty('--value', input.value);
});
```
CSS
```css
input[type=range]:focus {
  outline: none;
}
input[type=range]::-webkit-slider-runnable-track {
  background: linear-gradient(to right, #fc6 calc(1% * var(--value)), white 0);
  border-radius: 8px;
  border: 1px solid #010101;
}
```
<img src="https://image.ibb.co/cFY1RL/Screen-Shot-2018-10-20-at-12-58-09.png" alt="Screen-Shot-2018-10-20-at-12-58-09" border="0">

<img src="https://image.ibb.co/gJJqLf/Screen-Shot-2018-10-20-at-12-58-54.png" alt="Screen-Shot-2018-10-20-at-12-58-54" border="0">

*The fill of the background colour changes depending on the range (input) value.*

### Progress Bar
You could even create a progress bar using this same principle.  

The code below works very similar to that used in the range input example. It updates the scroll variable to the percentage that was scrolled (each time a scroll event happens) and then adds it to the custom CSS property `--scroll`.

JS
```javascript
for (let el of document.querySelectorAll(‘.scrolling’)) {
	el.addEventListener(‘scroll’, evt => {
		let maxScroll = el.scrollHeight - el.offsetHeight;
		let  scroll - el.scrollTop / maxScroll;
		el.style.setProperty(‘--scroll’, scroll);
	});
}
```
CSS
```css
.scrolling {
	background-image: linear-gradient(to right, purple calc(100% * var(--scroll)), white 0);
	background-size: 100% 1em;
}
```
  
## Conclusion
CSS variables offer a great deal of separation between style and behaviour. It offers the ability to rely purely on native CSS to do all your styling, instead of third party additions and workarounds, while giving great flexibility for added transformation using JavaScript.

You can checkout a working example of CSS custom properties [here](https://codepen.io/kerronp/pen/xyzXvb).