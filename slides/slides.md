# Programming with Sass

^ - I'm Alyssa
- Here to talk about why advanced programming features of Sass language are useful
- With every release, Sass becomes more usable as a programming language

---

# Why Program in Sass?

- Create re-usable components
- Let the computer do the boring work
- Do things that wouldn't be feasible in raw CSS

^ - Create re-usable parameterised components
- Save time
- Keep code maintainable
- Let the computer do the boring work
- Can do things that wouldn't be feasible in raw CSS

---

```css
div:nth-child(1) { background: hsl(  0deg, 0.5, 0.5); }
div:nth-child(2) { background: hsl( 60deg, 0.5, 0.5); }
div:nth-child(3) { background: hsl(120deg, 0.5, 0.5); }
div:nth-child(4) { background: hsl(180deg, 0.5, 0.5); }
div:nth-child(5) { background: hsl(240deg, 0.5, 0.5); }
div:nth-child(6) { background: hsl(300deg, 0.5, 0.5); }
```

^ - Premise
	- Based on colour theory from online
		- "Keep saturation and luminosity the same and vary the hue for colours that work well together"
	- Maybe not particularly accurate
		- There's a talk on colour theory at some point today
		- Still a good example
Repetitive. Eliminate repetition with a for loop.

---

```scss
$total: 6;
@for $i from 0 to $total {
  div:nth-child(#{$i + 1}) {
    $hue: $i * (360deg / $total);
    background: hsl($hue, 0.5, 0.5);
  }
}
```

^ We can do even better

---

```scss
@mixin color-pallette(
	$base-color, $total, $step
) {
  @for $i from 0 to $total {
    &:nth-child(#{$i + 1}) {
      $hue-offset: $i * $step;
      background: adjust-hue($base-color, $hue-offset);
    }
  }
}

div {
  @include color-pallette(hsl(0, 0.5, 0.5), 6, 60deg);
}
```

^ Weird formatting is because of video overlay.
- Easier to understand
- Easier to modify and reuse
	- Can change base colour and have the rest update
	- Can change number of colours, range (in degrees)
Now re-usable across project(s)

---

# API Library

^ Based on a project of mine
  Will focus less on programming constructs and more on challenges you'll face working with Sass

^ Not for HTTP from Sass directly.
For embedding third party CSS into your own
Google Web Fonts

---

# Google Web Fonts

Can specify

- Fonts
- Variants
- Subsets

^ Google web fonts provides API for getting font-face rules.
Heavily customisable.
- Fonts
- Variants (numeric weight, whether italic)
- Subsets (latin, cyrillic, etc.)
- Before Sassification
	- Have to go to the website
	- Interface is clunky
	- Have to redo every time you want to change
		- Might want to optimise
			- Choose a subset
			- Choose only certain weights or variants
		- Might need more than originally envisaged
			- Multiple subsets
			- More weights/variants

---

# Design an API

- Customisable parameters
- Concise
- Compatible with Ruby Sass and Libsass

^ - Needs to allow parameter customisation
- Needs to be concise for defaults
- Need to be compatible with Ruby Sass and Libsass
	- While they are theoretically the same language, Libsass lags a bit
		- Some edge-cases that Libsass doesn't handle you have to hack around
		- For example @import

---

# Libsass Compatibility

```scss
// doesn't work with libsass
@include web-fonts("Open Sans");

// doesn't work with libsass
@import url(web-fonts-url("Open Sans"));

// works with libsass!
$url: web-fonts-url("Open Sans");
@import url($url);
```

^ First doesn't work because @import can't be embedded in mixin
^ Second doesn't work because @import url() can't take function

---

# The API

```scss
// import a single font
@import url(web-fonts-url("Open Sans"));

// specify variants for a font
@import url(web-fonts-url( ("Open Sans": "bold") );

// multiple variants
@import url(web-fonts-url( ("Open Sans": ("500", "600 italic")) ));

// multiple fonts
@import url(web-fonts-url(
  "Open Sans",
  ("Ubuntu": ("400", italic))
));
```

^ - Customisable parameters
- Concise
- Compatible with Ruby Sass and Libsass

---

# URL Encoding

```scss
@function url-encode($string) {
  $replacements: (
    "!": "%21", "#": "%23", "$": "%24", "&": "%26",
    "'": "%27", "(": "%28", ")": "%29", "*": "%2A",
    "+": "%2B", ",": "%2C", "/": "%2F", ":": "%3A",
    ";": "%3B", "=": "%3D", "?": "%3F", "@": "%40",
    "[": "%5B", "]": "%5D", " ": "%20");

  @each $from, $to in $replacements {
    $string: str-replace($string, $from, $to);
  }

  @return $string;
}
```

^ - To do this, need to handle string replacement

---

# String replacement

```scss
@function str-replace($string, $find, $replace, $all: true) {
  $index: str-index($string, $find);
  @if $index {
    $before: str-slice($string, 1, $index - 1);
    $after: str-slice($string, $index + str-length($find), str-length($string));
    $string: $before + $replace + $after;

    @if $all and not str-index($find, $replace) {
      $string: str-replace($string, $find, $replace);
    }
  }
  @return $string;
}
```

^ 	- String replacement not implemented in Sass, but it's possible
	- Strings used to be completely opaque
		- No functions available from Sass for string data
		- This made string processing impossible
			- I did it anyway
				- Ask me how after


---

# No Module System

Imports use file paths

```scss
@import "bower_components/sass-web-fonts/web-fonts";
```

All functions are public

```scss
@function wf-str-replace...
```

^ prefix functions to avoid collision (hopefully)

---

# Modules in Sass 4

##### https://github.com/sass/proposal.module-system

`@use` module loading system

```scss
@use sass-web-fonts;
```

Functions whose names start with `-` or `_` are private.

```scss
@function -str-replace...
```

^ Private mixins/functions/variables can only be referenced from inside the module

---

# Unit Testing

^ Example: when changing API don't want output to change.

---

# Bootcamp _(no longer maintained)_
##### https://github.com/thejameskyle/bootcamp

```scss
@include describe("Math Power") {
  @include it("should expect positive values to be calculated correctly") {
    @include should( expect( power( 10, 2) ), to( equal(  100 )));
    @include should( expect( power(  2, 2) ), to( equal(    4 )));
    @include should( expect( power(0.5, 2) ), to( equal( 0.25 )));
  }
}
```

^ Abridged version of example from the Wiki.
- BDD-style tests implemented in pure Sass
- Abadoned
	- Author says "Don't use this" at top of README
- Felt overcomplicated


---

# `git diff`
##### From https://github.com/sass-mq/sass-mq

	test/
	    test-1.css
	    test-1.scss
	    test-2.css
	    test-2.scss

Compile, then run `git diff` and check for changes.

^ - They have since moved to custom scripts
- SCSS and CSS file pairs
- Compile SCSS files
	- Overwrite corresponding CSS files
- Run git diff to check nothing has changed
- Nice idea, but manual

---

# SassUnit
##### https://github.com/alyssais/sassunit

- Same structure as `git diff` approach
- Built on an existing testing library
- Multiple Sass implementations at once (soon!)

^ - Same structure as git diff approach
	- SCSS and CSS file pairs
- Built around a Ruby testing library
	- Get the benefit of a mature test library
- Can test with multiple Sass implementations at once
	- (Soon!)

---

# Sass Web Fonts
##### https://github.com/alyssais/sass-web-fonts

^ - Simple API
	- Don't have to go through web interface
	- Easy to change
- relatively Popular!
	- few hundred stars on GitHub

---

# Use @ statements

- `@debug`: print abitrary output to the terminal
- `@error`: halt execution
- `@warn`: print warning to the Terminal

^ Some advice from development of Sass libraries:
^ No way to catch errors

---

# `sass -i`

	~ $ sass -i
	>> 3px + 1pt
	3.25pt
	>> adjust-hue(red, 40deg)
	#ffaa00
	>>
	
^ - Test out SassScript expressions in a REPL
	- Subset of the Sass language
		- Things that would go on the RHS of a property
		- Useful for testing:
			- String operations
			- Units
				- The way math operations handle units in Sass is fiddly
		- Variables
			- Can assign to variables with colon
		- Functions
			- Can be called
			- Cannot be defined
				- AFAIK

---

# Extending Sass

- Hook into the Sass compiler
- Write in Ruby (Sass) or C (libsass)
- Not portable

^ Possible to write in C with implementation-specific adapters
