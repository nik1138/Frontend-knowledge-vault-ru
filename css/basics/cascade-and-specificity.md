# CSS Cascade and Specificity

Understanding the cascade and specificity is crucial for mastering CSS. These concepts determine how browsers resolve conflicts when multiple CSS rules apply to the same element.

## Table of Contents

- [The Cascade](#the-cascade)
- [Specificity](#specificity)
- [Calculating Specificity](#calculating-specificity)
- [The !important Declaration](#the-important-declaration)
- [Inheritance](#inheritance)
- [Advanced Specificity Concepts](#advanced-specificity-concepts)
- [Performance Considerations](#performance-considerations)
- [Best Practices](#best-practices)

## The Cascade

The cascade is the algorithm CSS uses to determine which styles are applied to elements when multiple rules conflict. The cascade considers:

1. **Importance**: Whether `!important` is used
2. **Specificity**: How specific a selector is
3. **Source Order**: The order CSS rules appear in the stylesheet

### Importance

Styles marked with `!important` have the highest priority in the cascade:

```css
p {
  color: blue !important;
}

#text {
  color: red;
}

/* The paragraph will be blue despite #text having higher specificity */
```

### Source Order

When rules have the same importance and specificity, the later rule wins:

```css
p {
  color: blue;
}

p {
  color: red;
}

/* Paragraphs will be red */
```

### Origin and Importance

CSS rules can come from different sources with different priority levels:

1. **Transition declarations** (highest priority)
2. **Animation declarations** 
3. **User agent important declarations** (browser defaults with !important)
4. **User important declarations** (user stylesheets with !important)
5. **Author important declarations** (your styles with !important)
6. **User agent normal declarations** (browser defaults)
7. **User normal declarations** (user stylesheets)
8. **Author normal declarations** (your styles - normal priority)
9. **Animation declarations** (lowest priority)

```css
/* Example of different origins */
p {
  color: blue; /* Author normal declaration */
}

/* If a user stylesheet has: */
p { color: green; } /* User normal declaration - would override author in some browsers */

/* If browser has: */
p { color: black; } /* User agent normal declaration - overridden by author */
```

## Specificity

Specificity is a scoring system that determines which CSS rule takes precedence. Each type of selector contributes points to the specificity score:

### Specificity Hierarchy

1. **Inline styles** (style attribute) - 1000 points
2. **IDs** - 100 points each
3. **Classes, attributes, pseudo-classes** - 10 points each
4. **Elements, pseudo-elements** - 1 point each

### Calculating Specificity

Let's look at examples of how specificity is calculated:

```css
/* Specificity: 0,0,0,1 (element) */
p {
  color: blue;
}

/* Specificity: 0,0,1,0 (class) */
.text {
  color: red;
}

/* Specificity: 0,1,0,0 (ID) */
#header {
  color: green;
}

/* Specificity: 0,1,1,1 (ID + class + element) */
#header .nav p {
  color: purple;
}

/* Specificity: 0,0,2,1 (2 classes + element) */
.sidebar.highlight p {
  color: orange;
}
```

### Specificity Examples

```css
/* Specificity: 0,0,0,1 */
div { }

/* Specificity: 0,0,1,1 */
div.example { }

/* Specificity: 0,1,0,1 */
div#content { }

/* Specificity: 0,1,1,1 */
#content .example { }

/* Specificity: 0,2,0,0 */
#content #footer { } /* Invalid - same element can't have 2 IDs */
```

## Complex Specificity Scenarios

### Descendant vs Child Selectors

Both descendant and child selectors contribute equally to specificity:

```css
/* Specificity: 0,0,0,2 (two elements) */
div p { }

/* Specificity: 0,0,0,2 (two elements) */
div > p { }
```

### Pseudo-classes and Pseudo-elements

Pseudo-classes count as classes (10 points), while pseudo-elements count as elements (1 point):

```css
/* Specificity: 0,0,1,1 (class + pseudo-class) */
.button:hover { }

/* Specificity: 0,0,1,2 (class + element + pseudo-element) */
.nav::before { }

/* Specificity: 0,0,2,1 (class + pseudo-class + element) */
.item:first-child { }
```

### Complex Selectors

```css
/* Specificity: 0,0,2,2 (2 classes, 2 elements) */
.container .item.active:hover { }

/* Specificity: 0,1,1,1 (1 ID, 1 class, 1 element) */
#header .navigation li { }

/* Specificity: 0,0,3,1 (3 classes, 1 element) */
.form .input.error:focus { }
```

### Universal Selector Impact

The universal selector (*) has no impact on specificity:

```css
/* Specificity: 0,0,0,0 (no points for universal selector) */
* { margin: 0; }

/* Specificity: 0,0,0,1 (only the element selector counts) */
*.highlight { color: red; }
```

## The !important Declaration

While `!important` can override specificity, it should be used sparingly:

```css
/* Avoid this pattern */
p {
  color: blue !important;
}

/* Better approach - increase specificity */
.content p {
  color: blue;
}
```

### When to Use !important

- Overriding third-party library styles
- Quick fixes during development (should be refactored later)
- User preferences (e.g., high contrast mode)
- Critical accessibility enhancements

```css
/* Example: High contrast mode for accessibility */
@media (prefers-contrast: high) {
  * {
    background-color: #000 !important;
    color: #fff !important;
    border-color: #fff !important;
  }
}
```

### !important in CSS Custom Properties

CSS custom properties can work with !important in complex ways:

```css
:root {
  --primary-color: red;
}

.element {
  --primary-color: blue !important; /* This will take precedence */
  color: var(--primary-color);
}
```

## Inheritance

Some CSS properties are inherited by child elements from their parents:

### Inheritable Properties
- `color`
- `font-family`
- `font-size`
- `line-height`
- `text-align`
- `visibility`
- `cursor`
- `letter-spacing`
- `word-spacing`
- `text-transform`
- `text-decoration`
- `text-shadow`
- `white-space`
- `word-break`
- `word-wrap`
- `direction`
- `unicode-bidi`

### Non-Inheritable Properties
- `margin`
- `padding`
- `border`
- `width`
- `height`
- `position`
- `top`, `right`, `bottom`, `left`
- `display`
- `background`
- `box-sizing`
- `transform`
- `transition`
- `animation`

### Controlling Inheritance

```css
/* Force inheritance */
.child {
  color: inherit;
}

/* Reset to initial value */
.child {
  color: initial;
}

/* Use browser default */
.child {
  color: unset;
}

/* Revert to user-agent value */
.child {
  color: revert;
}
```

### Inheritance Examples

```css
/* Font family inherited by all children */
.parent {
  font-family: Arial, sans-serif;
}

.child {
  /* No need to specify font-family - it's inherited */
  font-size: 1.2em; /* Relative to parent's font size */
}

/* Text alignment inherited */
.container {
  text-align: center;
}

.text {
  /* This text will be centered without specifying */
}
```

## Advanced Specificity Concepts

### The :not() Pseudo-class

The `:not()` pseudo-class doesn't add specificity itself, but its content does:

```css
/* Specificity: 0,0,1,0 (class inside :not) */
div:not(.container) { }

/* Specificity: 0,1,0,0 (ID inside :not) */
div:not(#header) { }

/* Specificity: 0,0,0,2 (two elements inside :not) */
div:not(.container p) { } /* This is invalid syntax - :not can only contain simple selectors */
```

Actually, the correct example is:
```css
/* Specificity: 0,0,1,1 (element + class) */
div:not(.container) { }

/* Specificity: 0,0,2,1 (element + 2 classes) */
div:not(.container.item) { }
```

### The :is(), :where(), and :has() Pseudo-classes

Modern CSS introduces new pseudo-classes for complex selection:

#### :is() - Matches if any of the selectors match
```css
/* Specificity: 0,0,1,0 (takes the highest specificity among options) */
:is(.nav, .menu, .sidebar) { background: #f0f0f0; }
```

#### :where() - Same as :is() but with 0 specificity
```css
/* Specificity: 0,0,0,0 (no specificity contribution) */
:where(.nav, .menu, .sidebar) { background: #f0f0f0; }
```

#### :has() - Matches if the element contains matching descendants
```css
/* Specificity: 0,0,1,1 (class + :has) */
.container:has(.featured) { border: 2px solid gold; }
```

### Specificity in CSS Custom Properties

CSS custom properties have unique behavior with specificity:

```css
:root {
  --text-color: blue;
}

.component {
  --text-color: red; /* This overrides the root value for this scope */
  color: var(--text-color); /* Will be red */
}

/* The property 'color' itself doesn't participate in specificity calculation */
/* The variable value is resolved based on where the variable is defined */
```

## Performance Considerations

### Selector Performance

While modern browsers are highly optimized, some selectors can impact performance:

#### Fast Selectors
```css
/* Fast: ID selectors */
#header { }

/* Fast: Class selectors */
.button { }

/* Fast: Element selectors */
p { }
```

#### Slower Selectors
```css
/* Slower: Descendant selectors */
.container p { }

/* Slower: Attribute selectors */
input[type="text"] { }

/* Slower: Pseudo-selectors */
.item:nth-child(odd) { }
```

### Optimizing Complex Selectors

```css
/* Instead of this complex selector */
body div.container ul.navigation li.item a.link:hover { }

/* Use more specific classes */
.nav-link:hover { }

/* Or add a class to the container */
.navigation .nav-link:hover { }
```

### Selector Matching Process

Browsers match selectors from right to left for performance:

```css
/* Browser first finds all .link elements, then checks if they match the full selector */
.container .item a.link:hover { }

/* This is more efficient because there are fewer .nav-link elements to check */
.nav-link:hover { }
```

## Best Practices

### 1. Write Low-Specificity CSS

Start with simple selectors and increase specificity only when necessary:

```css
/* Good */
.button { }

/* Avoid */
div.container > .button.special:first-child { }
```

### 2. Use Classes Over IDs

IDs have high specificity and make CSS harder to override:

```css
/* Better */
.button-primary { }

/* Avoid */
#button { }
```

### 3. Organize CSS Logically

Structure your CSS to take advantage of the cascade:

```css
/* Base styles */
.button { }

/* Variant styles */
.button--primary { }

/* State styles */
.button.is-active { }
```

### 4. Avoid !important

Instead of using `!important`, refactor your CSS to use proper specificity:

```css
/* Instead of */
.button {
  color: blue !important;
}

/* Use more specific selectors */
.form .button {
  color: blue;
}
```

### 5. Use CSS Architecture Methodologies

Consider using methodologies like BEM (Block Element Modifier):

```css
/* Block */
.card { }

/* Element */
.card__header { }

/* Modifier */
.card--featured { }

/* Element with modifier */
.card__button--primary { }
```

### 6. Leverage CSS Custom Properties

Use CSS custom properties to reduce specificity issues:

```css
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
}

/* Instead of high-specificity selectors */
.component {
  background-color: var(--primary-color);
}
```

### 7. Plan Your CSS Architecture

Create a consistent specificity hierarchy:

```css
/* 1. Base styles (lowest specificity) */
* { box-sizing: border-box; }
body { font-family: Arial, sans-serif; }

/* 2. Layout styles */
.container { max-width: 1200px; }

/* 3. Component styles */
.button { padding: 10px 15px; }

/* 4. State styles */
.button.is-active { background-color: #0056b3; }

/* 5. Theme overrides (highest specificity) */
.theme-dark .button { background-color: #454545; }
```

## Practical Exercise

Create a complex HTML document with nested elements and apply CSS rules with different specificity levels. Test how the cascade resolves conflicts and observe how inheritance works with various properties. Use browser developer tools to inspect computed styles and understand how the cascade works.

## Key Takeaways

- The cascade determines which CSS rules take precedence
- Specificity is calculated using a point system for different selector types
- Higher specificity wins over lower specificity
- The `!important` declaration overrides normal specificity
- Understanding inheritance helps predict which properties are passed to child elements
- Good CSS architecture minimizes the need for high specificity selectors
- Modern CSS features like :is(), :where(), and :has() offer new ways to write selectors
- Performance considerations should guide selector complexity

## Next Steps

Continue to [[CSS Values and Units]] to learn about the different types of values and units you can use in CSS.

## Tags

#css #cascade #specificity #web-development #frontend #selectors #inheritance #performance