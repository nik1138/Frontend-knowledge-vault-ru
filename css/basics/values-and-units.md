# CSS Values and Units

CSS values and units determine the size, spacing, colors, and other properties of elements. Understanding the different types of values and units is essential for creating responsive and well-designed websites.

## Table of Contents

- [Types of CSS Values](#types-of-css-values)
- [CSS Units](#css-units)
- [Length Units](#length-units)
- [Color Values](#color-values)
- [Functional Values](#functional-values)
- [Keywords](#keywords)
- [Best Practices](#best-practices)

## Types of CSS Values

CSS values fall into several categories:

### Length Values
Represent distances, such as sizes, margins, and padding.

### Percentage Values
Relative to the parent element or the element's own size.

### Color Values
Define colors for text, backgrounds, and borders.

### Functional Values
Values that take parameters, like `calc()`, `var()`, `rgb()`, etc.

### Keywords
Predefined words that represent specific values (e.g., `auto`, `inherit`, `initial`).

## CSS Units

CSS units can be categorized as absolute or relative:

### Absolute Units
Fixed physical measurements that don't change based on context:

- `px` - Pixels (1/96th of 1 inch)
- `pt` - Points (1/72nd of 1 inch)
- `pc` - Picas (12 points)
- `cm` - Centimeters
- `mm` - Millimeters
- `in` - Inches

### Relative Units
Measurements relative to other values:

- `em` - Relative to font size of the element
- `rem` - Relative to font size of the root element
- `ch` - Relative to width of the "0" character
- `ex` - Relative to x-height of the element's font
- `%` - Percentage of parent element
- `vw` - 1% of viewport width
- `vh` - 1% of viewport height
- `vmin` - 1% of viewport's smaller dimension
- `vmax` - 1% of viewport's larger dimension

## Length Units

### Absolute Units

```css
/* Absolute units */
.absolute {
  width: 2in;     /* 2 inches */
  height: 5cm;    /* 5 centimeters */
  margin: 12pt;   /* 12 points */
  padding: 10px;  /* 10 pixels */
}
```

### Relative Units

```css
/* Relative units */
.relative {
  font-size: 1.2em;      /* 1.2 times the parent's font size */
  margin: 2rem;          /* 2 times the root font size */
  width: 50%;            /* 50% of parent width */
  height: 80vh;          /* 80% of viewport height */
  padding: 1.5ch;        /* 1.5 times the width of "0" */
}
```

### When to Use Each Unit

#### Use `px` for:
- Fine control over small elements
- Border widths
- Exact positioning when necessary

#### Use `em` for:
- Scaling within components
- Responsive typography
- Elements that should scale with font size

#### Use `rem` for:
- Consistent spacing throughout the site
- Maintaining consistent rhythm
- When you want to scale everything together

#### Use `%` for:
- Responsive layouts
- Flexible containers
- Elements that should adapt to their container

#### Use `vw`/`vh` for:
- Full-screen elements
- Responsive typography
- Viewport-based layouts

## Color Values

CSS supports several ways to specify colors:

### Named Colors
```css
.named-colors {
  color: red;
  background-color: lightblue;
  border-color: tomato;
}
```

### Hexadecimal Colors
```css
.hex-colors {
  color: #ff0000;        /* Red */
  background-color: #f0f; /* Shorthand for #ff00ff (magenta) */
  border-color: #336699;  /* Blue */
}
```

### RGB Colors
```css
.rgb-colors {
  color: rgb(255, 0, 0);          /* Red */
  background-color: rgba(255, 0, 0, 0.5);  /* Red with 50% opacity */
  border-color: rgb(100%, 50%, 0%);        /* Orange */
}
```

### HSL Colors
```css
.hsl-colors {
  color: hsl(0, 100%, 50%);         /* Red */
  background-color: hsla(0, 100%, 50%, 0.5);  /* Red with 50% opacity */
  border-color: hsl(240, 100%, 50%);         /* Blue */
}
```

### Current Color
```css
.current-color {
  color: blue;
  border: 1px solid currentColor;  /* Will be blue */
}
```

## Functional Values

### calc()
The `calc()` function allows mathematical expressions:

```css
.calc-example {
  width: calc(100% - 20px);           /* Full width minus 20px */
  margin: calc(1rem + 2px);           /* 1rem plus 2px */
  padding: calc(5vh + 10px);          /* 5% of viewport height plus 10px */
}
```

### var()
The `var()` function references CSS custom properties:

```css
:root {
  --main-color: #007bff;
  --spacing-unit: 1rem;
}

.custom-properties {
  color: var(--main-color);
  margin: var(--spacing-unit);
  padding: calc(var(--spacing-unit) * 2);
}
```

### attr()
The `attr()` function retrieves attribute values:

```css
.tooltip::after {
  content: attr(data-tooltip);
}
```

## Keywords

CSS includes various keywords that represent specific values:

### Global Keywords
```css
.global-keywords {
  color: inherit;     /* Inherits from parent */
  display: initial;   /* Resets to initial value */
  visibility: unset;  /* Acts as either inherit or initial */
}
```

### Property-Specific Keywords
```css
.property-keywords {
  display: block;
  position: relative;
  text-align: center;
  overflow: hidden;
  cursor: pointer;
}
```

## Best Practices

### 1. Choose the Right Unit

- Use `rem` for consistent spacing throughout your site
- Use `em` for component-level scaling
- Use `%` for responsive layouts
- Use `vw`/`vh` for full-screen elements
- Avoid `px` for anything that should scale

### 2. Use Relative Units for Typography

```css
/* Good for responsive typography */
.typography {
  font-size: 1.125rem;  /* 18px if root is 16px */
  line-height: 1.5;     /* Unitless multiplier */
  margin-bottom: 1em;   /* Scales with font size */
}
```

### 3. Combine Units with calc()

```css
/* Mix units safely */
.combined-units {
  width: calc(100% - 2rem);
  padding: calc(0.5rem + 2px);
  margin: calc(2vh + 1em);
}
```

### 4. Use Custom Properties for Consistency

```css
:root {
  --base-font-size: 1rem;
  --spacing-xs: 0.5rem;
  --spacing-sm: 1rem;
  --spacing-md: 1.5rem;
  --spacing-lg: 2rem;
}

.component {
  font-size: var(--base-font-size);
  margin: var(--spacing-md);
}
```

### 5. Consider Accessibility

```css
/* Ensure text remains readable */
.accessible-typography {
  font-size: clamp(1rem, 2.5vw, 1.5rem);  /* Responsive font size */
  line-height: 1.4;                       /* Good readability */
}
```

## Practical Exercise

Create a responsive card component that uses different units for different properties. Use `rem` for consistent spacing, `em` for scaling within the card, `%` for responsive width, and `vw`/`vh` for viewport-based elements.

## Key Takeaways

- Different units serve different purposes in CSS
- Relative units create more flexible and responsive designs
- Understanding color value formats provides more control over design
- Functional values like `calc()` and `var()` add flexibility
- Choosing the right unit for the right context improves maintainability
- Consistent use of custom properties promotes design system consistency

## Next Steps

Continue to [[CSS Box Model]] to understand how CSS handles the layout of elements in terms of content, padding, border, and margin.

## Tags

#css #values #units #web-development #frontend #colors #typography #responsive-design