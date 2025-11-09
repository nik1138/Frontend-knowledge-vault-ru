# CSS Basics

CSS (Cascading Style Sheets) is the language used to describe the presentation of a document written in HTML or XML. This module covers the fundamental concepts of CSS that every developer should understand.

## Table of Contents

- [Introduction to CSS](#introduction-to-css)
- [CSS Syntax](#css-syntax)
- [How CSS Works](#how-css-works)
- [CSS Selectors](#css-selectors)
- [CSS Properties and Values](#css-properties-and-values)
- [CSS Units](#css-units)
- [Basic Styling Concepts](#basic-styling-concepts)
- [Advanced Syntax Features](#advanced-syntax-features)
- [Best Practices](#best-practices)

## Introduction to CSS

CSS stands for Cascading Style Sheets and is used to style and layout web pages. It controls the look and feel of HTML elements, including colors, fonts, spacing, and positioning.

CSS allows you to separate the presentation of a document from its content, making it easier to maintain and update the visual design of websites. The "cascading" part refers to how styles cascade down from parent elements to children, and how rules from different sources are combined according to specific rules.

### The Role of CSS in Web Development

CSS serves several critical functions in web development:
- **Presentation**: Controls visual appearance (colors, fonts, spacing)
- **Layout**: Positions elements on the page
- **Responsiveness**: Adapts designs to different screen sizes
- **Performance**: Optimizes rendering and user experience
- **Accessibility**: Ensures content is perceivable by all users

## CSS Syntax

CSS rules consist of three parts:
- **Selector**: Specifies which HTML element(s) to style
- **Property**: The aspect of the element to change
- **Value**: The setting for that property

```css
selector {
  property: value;
}
```

Example:
```css
p {
  color: blue;
  font-size: 16px;
}
```

### CSS Declaration Blocks

A declaration block is a group of CSS declarations enclosed in curly braces:

```css
/* Declaration block */
{
  color: red;
  font-size: 18px;
  margin: 10px;
}
```

Each declaration consists of a property name, followed by a colon, then a property value, and ending with a semicolon.

### Comments in CSS

CSS supports single-line and multi-line comments:

```css
/* This is a single-line comment */

/*
This is a
multi-line comment
*/

/* Comments are useful for explaining complex styles */
.grid-container {
  display: grid; /* Enable grid layout */
  grid-template-columns: repeat(3, 1fr); /* Create 3 equal columns */
}
```

## How CSS Works

CSS works by matching selectors to HTML elements and applying the specified styles. The browser reads CSS rules from top to bottom and applies them to the DOM elements.

The cascade mechanism determines which styles are applied when multiple rules target the same element, considering:
- Importance (using `!important`)
- Specificity of selectors
- Source order of CSS rules

### The CSS Processing Model

1. **Parsing**: The browser parses HTML and CSS into DOM and CSSOM trees
2. **Combining**: The DOM and CSSOM trees are combined into a render tree
3. **Layout**: The browser calculates the size and position of each element
4. **Painting**: The browser draws the elements on the screen

### CSS Inclusion Methods

CSS can be included in HTML documents in three ways:

#### Inline Styles
```html
<p style="color: blue; font-size: 16px;">This paragraph has inline styles</p>
```

#### Internal Stylesheets
```html
<head>
  <style>
    p {
      color: blue;
      font-size: 16px;
    }
  </style>
</head>
```

#### External Stylesheets
```html
<head>
  <link rel="stylesheet" href="styles.css">
</head>
```

## CSS Selectors

Selectors are patterns used to select which HTML elements to style. Understanding selectors is crucial for writing efficient and maintainable CSS.

### Element Selectors
Selects all elements of a specific type:

```css
p { color: red; } /* Selects all <p> elements */
div { margin: 10px; } /* Selects all <div> elements */
h1, h2, h3 { font-weight: bold; } /* Selects multiple element types */
```

### Class Selectors
Selects elements with a specific class attribute:

```css
.highlight { background-color: yellow; } /* Selects elements with class="highlight" */
.btn { padding: 10px 15px; } /* Selects elements with class="btn" */
```

In HTML:
```html
<p class="highlight">This text is highlighted</p>
<button class="btn">Click me</button>
```

### ID Selectors
Selects a single element with a specific ID:

```css
#header { font-size: 24px; } /* Selects the element with id="header" */
#navigation { background-color: #333; }
```

In HTML:
```html
<div id="header">Header content</div>
<nav id="navigation">Navigation links</nav>
```

### Attribute Selectors
Selects elements based on attribute values:

```css
/* Selects elements with a specific attribute */
[title] { border: 1px solid #ccc; }

/* Selects elements with a specific attribute value */
[type="text"] { border: 1px solid gray; }

/* Selects elements with attribute values containing a substring */
[class*="btn"] { padding: 10px; }

/* Selects elements with attribute values starting with a string */
[href^="https"] { color: green; }

/* Selects elements with attribute values ending with a string */
[href$=".pdf"] { background: url(pdf-icon.png); }
```

### Pseudo-classes
Selects elements in a specific state:

```css
/* Hover state */
a:hover { color: red; }

/* First child */
li:first-child { font-weight: bold; }

/* Last child */
li:last-child { border-bottom: none; }

/* Nth child (even rows) */
tr:nth-child(even) { background-color: #f2f2f2; }

/* Focus state */
input:focus { outline: 2px solid blue; }

/* Link states */
a:link { color: blue; }    /* Unvisited links */
a:visited { color: purple; } /* Visited links */
a:hover { color: red; }    /* Hovered links */
a:active { color: orange; } /* Active links */
```

### Pseudo-elements
Selects specific parts of an element:

```css
/* First line of text */
p::first-line { font-weight: bold; }

/* First letter of text */
p::first-letter { font-size: 2em; }

/* Before pseudo-element */
.quote::before { content: """; }

/* After pseudo-element */
.quote::after { content: """; }

/* Selection pseudo-element */
::selection { background-color: yellow; }
```

### Combinators
Combine selectors to create more specific rules:

#### Descendant Combinator (Space)
Selects elements that are descendants of another element:

```css
/* Selects all <p> elements inside <div> elements */
div p { color: blue; }
```

#### Child Combinator (>)
Selects elements that are direct children of another element:

```css
/* Selects only <p> elements that are direct children of <div> */
div > p { margin: 0; }
```

#### Adjacent Sibling Combinator (+)
Selects elements that immediately follow another element:

```css
/* Selects <p> elements that immediately follow <h1> */
h1 + p { margin-top: 0; }
```

#### General Sibling Combinator (~)
Selects elements that follow another element (not necessarily adjacent):

```css
/* Selects all <p> elements that follow <h1> */
h1 ~ p { color: gray; }
```

## CSS Properties and Values

CSS properties define what aspect of an element to style, while values specify how to style it.

### Common Properties

#### Text Properties
- `color`: Text color
- `font-family`: Font type
- `font-size`: Size of text
- `font-weight`: Thickness of text
- `text-align`: Horizontal alignment
- `text-decoration`: Text decoration (underline, etc.)
- `line-height`: Space between lines of text

#### Box Model Properties
- `width`, `height`: Size of the content area
- `margin`: Space outside the border
- `padding`: Space inside the border
- `border`: Border around the content
- `box-sizing`: How width and height are calculated

#### Layout Properties
- `display`: How an element is displayed
- `position`: Positioning method
- `float`: Float positioning
- `clear`: Clearing floats
- `overflow`: Handling content overflow

#### Visual Properties
- `background-color`: Background color
- `background-image`: Background image
- `opacity`: Element transparency
- `visibility`: Element visibility
- `cursor`: Mouse cursor style

### Property Values

CSS properties can accept various types of values:

#### Keywords
```css
.text-center { text-align: center; }
.block { display: block; }
.relative { position: relative; }
```

#### Lengths
```css
.padding-small { padding: 10px; }
.margin-large { margin: 2em; }
.width-half { width: 50%; }
```

#### Colors
```css
.red-text { color: red; }
.hex-color { color: #ff0000; }
.rgb-color { color: rgb(255, 0, 0); }
.rgba-color { color: rgba(255, 0, 0, 0.5); }
.hsl-color { color: hsl(0, 100%, 50%); }
```

#### URLs
```css
.background-image { background-image: url('image.jpg'); }
.cursor-custom { cursor: url('cursor.png'), auto; }
```

#### Functions
```css
.gradient-bg { background: linear-gradient(to right, #fff, #000); }
.transformed { transform: rotate(45deg); }
.calculated { width: calc(100% - 20px); }
```

## CSS Units

CSS supports various units for specifying sizes, each with specific use cases and advantages.

### Absolute Units
Absolute units are fixed and do not change based on other elements or screen sizes:

- `px` - Pixels (most commonly used)
- `pt` - Points (1/72 inch, often used in print)
- `cm` - Centimeters
- `mm` - Millimeters
- `in` - Inches

```css
.print-margin { margin: 1in; } /* 1 inch margin */
.paper-size { width: 21cm; height: 29.7cm; } /* A4 paper size */
```

### Relative Units
Relative units scale based on other factors, making them ideal for responsive designs:

#### Font-relative units
- `em` - Relative to the font size of the element
- `rem` - Relative to the font size of the root element (html)
- `ch` - Relative to the width of the "0" character
- `ex` - Relative to the x-height of the font

```css
.container {
  font-size: 16px; /* Base font size */
}

.child {
  margin: 1em;  /* 16px (relative to parent's font size) */
  padding: 0.5em; /* 8px */
}

.root-relative {
  margin: 2rem; /* 32px (relative to root font size) */
}

.width-based-on-zero { width: 10ch; /* Width of 10 "0" characters */ }
```

#### Viewport-relative units
- `vw` - 1% of viewport width
- `vh` - 1% of viewport height
- `vmin` - 1% of the smaller of viewport width or height
- `vmax` - 1% of the larger of viewport width or height

```css
.full-height { height: 100vh; } /* Full viewport height */
.responsive-text { font-size: 4vw; } /* Responsive text size */
.square-element { width: 50vmin; height: 50vmin; } /* Square relative to smaller dimension */
```

#### Percentage units
Percentages are relative to the parent element's corresponding property:

```css
.parent { width: 400px; }
.child { width: 50%; } /* 200px (50% of parent width) */
```

### Modern CSS Units

#### The `fr` unit
The `fr` unit represents a fraction of the available space in a grid container:

```css
.grid-container {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr; /* Three columns with ratio 1:2:1 */
}
```

#### The `min()`, `max()`, and `clamp()` functions
These functions allow for more sophisticated responsive sizing:

```css
/* Minimum of 200px or 50% of container */
.responsive-width { width: min(200px, 50%); }

/* Maximum of 400px or 80% of container */
.max-width { width: max(400px, 80%); }

/* Clamped between 16px and 24px, with preferred value of 5vw */
.clamped-text { font-size: clamp(16px, 5vw, 24px); }
```

## Basic Styling Concepts

### The Box Model

Every HTML element is treated as a rectangular box with:
- Content: The actual content (text, images, etc.)
- Padding: Space between content and border
- Border: Line around the padding and content
- Margin: Space outside the border

```css
.box {
  width: 200px;
  height: 100px;
  padding: 10px;      /* Space inside the border */
  border: 5px solid black;  /* The border */
  margin: 15px;       /* Space outside the border */
}
```

The total width of the box is: width + left padding + right padding + left border + right border + left margin + right margin

#### Box-sizing Property

The `box-sizing` property changes how the width and height of elements are calculated:

```css
/* Default: width/height only apply to content */
.content-box { box-sizing: content-box; }

/* Width/height include padding and border */
.border-box { box-sizing: border-box; }
```

### Display Property

Controls how an element is displayed:

- `block`: Takes up full width, starts on new line
- `inline`: Takes up only necessary width, doesn't start new line
- `inline-block`: Behaves like inline but can have width/height set
- `none`: Hides the element completely
- `flex`: Creates a flex container
- `grid`: Creates a grid container

```css
.block-element { display: block; }
.inline-element { display: inline; }
.flex-container { display: flex; }
.hidden-element { display: none; }
```

## Advanced Syntax Features

### CSS Variables (Custom Properties)

CSS variables allow you to store values that can be reused throughout your stylesheet:

```css
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --font-size-base: 16px;
}

.button {
  background-color: var(--primary-color);
  font-size: var(--font-size-base);
}
```

### CSS Functions

CSS includes various built-in functions for calculations and transformations:

#### calc() Function
Performs calculations to determine CSS values:

```css
.responsive-container {
  width: calc(100% - 40px); /* Full width minus 40px */
  margin: calc(10px + 5%);  /* 10px plus 5% of container */
}
```

#### var() Function
References CSS custom properties:

```css
.element {
  color: var(--main-color, blue); /* Uses --main-color or blue as fallback */
}
```

#### attr() Function
Retrieves the value of an HTML attribute:

```css
.label::after {
  content: attr(data-label); /* Uses the data-label attribute value */
}
```

### At-Rules

CSS includes special instructions called at-rules:

#### @import
Imports other CSS files:

```css
@import url('typography.css');
@import 'layout.css';
```

#### @media
Defines responsive styles:

```css
@media (max-width: 768px) {
  .container { width: 100%; }
}
```

#### @keyframes
Defines animation sequences:

```css
@keyframes slideIn {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}
```

## Best Practices

### 1. Use Semantic Class Names
Use descriptive, semantic class names that describe the element's purpose:

```css
/* Good: Semantic names */
.card { }
.navigation { }
.article-content { }

/* Avoid: Presentational names */
.red-text { }
.big-font { }
.left-align { }
```

### 2. Organize Your CSS Logically
Structure your CSS in a logical, maintainable way:

```css
/* 1. Reset/Normalize */
* { box-sizing: border-box; }

/* 2. Base styles */
body { font-family: Arial, sans-serif; }

/* 3. Layout components */
.container { max-width: 1200px; }

/* 4. Components */
.button { padding: 10px 20px; }

/* 5. Utilities */
.text-center { text-align: center; }
```

### 3. Use Shorthand Properties
Use shorthand properties to reduce code and improve readability:

```css
/* Instead of multiple declarations */
.element {
  margin-top: 10px;
  margin-right: 15px;
  margin-bottom: 10px;
  margin-left: 15px;
}

/* Use shorthand */
.element {
  margin: 10px 15px; /* vertical horizontal */
}

/* Or even more specific */
.element {
  margin: 10px 15px 10px 15px; /* top right bottom left */
}
```

### 4. Comment Your CSS
Add comments to explain complex styles or design decisions:

```css
/* Responsive navigation that collapses on mobile */
.nav-responsive {
  display: flex;
  justify-content: space-between;
}

/* Uses flexbox to maintain equal spacing between items */
.nav-links {
  display: flex;
  gap: 2rem;
}
```

### 5. Consider Performance
Write CSS that performs well:

- Avoid overly complex selectors
- Use efficient properties for animations (transform, opacity)
- Minimize the use of expensive properties (box-shadow, gradients)
- Use appropriate units for responsive design

## Key Takeaways

- CSS controls the presentation layer of web documents
- Understanding selectors is crucial for targeting the right elements
- The cascade determines which styles are applied when conflicts occur
- The box model is fundamental to understanding element sizing and spacing
- CSS units allow for responsive and scalable designs
- Modern CSS features like variables and functions provide powerful tools
- Following best practices leads to maintainable and performant code

## Next Steps

Continue to [[CSS Cascade and Specificity]] to understand how CSS determines which rules take precedence when multiple rules apply to the same element.

## Tags

#css #basics #web-development #frontend #selectors #properties #values #units #syntax