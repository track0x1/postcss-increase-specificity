# PostCSS Specificity

Originally forked from [postcss-increase-specificity](https://github.com/MadLittleMods/postcss-increase-specificity)- to increase the specificity of your selectors.

### Install

`npm install postcss-specificity --save-dev`

# Usage

## Basic Example

```js
var postcss = require('postcss');
var increaseSpecificity = require('postcss-specificity');

var fs = require('fs');

var mycss = fs.readFileSync('input.css', 'utf8');

// Process your CSS with postcss-specificity
var output = postcss([
        increaseSpecificity(/*options*/)
    ])
    .process(mycss)
    .css;

console.log(output);
```


## Results

Input:

```css
html {
	background: #485674;
	height: 100%;
}

.blocks {
	background: #34405B;
}

#main-nav {
	color: #ffffff;
}

[id="main-nav"] {
	border: 1px solid #ffffff;
}

.foo,
.bar {
	display: inline-block;
	width: 50%;
}

```

Output (result):

```css
html:not(#\9):not(#\9):not(#\9) {
	background: #485674;
	height: 100%;
}

:not(#\9):not(#\9):not(#\9) .blocks {
	background: #34405B;
}

:not(#\9):not(#\9):not(#\9) #main-nav {
	color: #ffffff !important;
}

:not(#\9):not(#\9):not(#\9) [id="main-nav"] {
	border: 1px solid #ffffff !important;
}

:not(#\9):not(#\9):not(#\9) .foo,
:not(#\9):not(#\9):not(#\9) .bar {
	display: inline-block;
	width: 50%;
}

```


## Only apply to certain sections of styles

There are two ways handle this. If the majority of your CSS needs to increase specificity, then you can leverage inline comments to disable/enable blocks of rules.

To temporarily disable `postcss-specificity`, use block comments in the following format:

```css
/* postcss-specificity disable */
.foo {
  content: 'cant touch this';
}
/* postcss-specificity enable */

.bar {
  content: 'but i can!';
}
```

Otherwise, [`postcss-plugin-context`](https://github.com/postcss/postcss-plugin-context) allows you to apply plugins to only in certain areas of of your CSS using `@context`.


```js
var postcss = require('postcss');
var context = require('postcss-plugin-context');
var increaseSpecificity = require('postcss-specificity');

var fs = require('fs');

var mycss = fs.readFileSync('input.css', 'utf8');

// Process your CSS with postcss-specificity
var output = postcss([
		context({
	        increaseSpecificity: increaseSpecificity(/*options*/)
	    })
    ])
    .process(mycss)
    .css;

console.log(output);
```

Input:

```css
/* these styles will be left alone */
html {
	background: #485674;
	height: 100%;
}

p {
	display: inline-block;
	width: 50%;
}


/* these styles will have the hacks prepended */
@context increaseSpecifity {
	#main-nav {
		color: #ffffff;
	}

	.blocks {
		background: #34405b;
	}

	.baz,
	.qux {
		display: inline-block;
		width: 50%;
	}
}
```

# What it does? *(by default)*

 - Prepend a descendant selector piece: `:not(#\9)` repeated the specified, `options.repeat`, number of times.
 - Add `!important` declarations to any selectors that have to do with an id.


# Options

 - `repeat`: number - The number of times we prepend `options.stackableRoot` in front of your selector
 	 - Default: `3`
 - `overrideIds`: bool - Whether we should add `!important` to all declarations that use id's in any way. Because id's are so specific, the only way(essentially) to overcome another id is to use `!important`.
 	 - Default: `true`
 - `stackableRoot`: string - Selector that is repeated to make up the piece that is added to increase specificity
 	 - Default: `:not(#\9)`
 	 - *Warning:* The default `:not(#\9)` pseudo-class selector is not supported in IE8-. To support IE-, you can change this option to a class such as `.my-root` and add it to the `<html class="my-root">` tag in your markup.


# Tests

`npm test`
