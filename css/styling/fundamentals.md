# CSS Styling Fundamentals

CSS styling encompasses the visual presentation of web content, including colors, typography, backgrounds, borders, and more. This module covers the essential styling properties and techniques used to create visually appealing web interfaces.

## Table of Contents

- [Introduction to CSS Styling](#introduction-to-css-styling)
- [Color Theory in CSS](#color-theory-in-css)
- [Typography and Text Styling](#typography-and-text-styling)
- [Background Properties](#background-properties)
- [Border Properties](#border-properties)
- [Visual Formatting](#visual-formatting)
- [Styling Patterns](#styling-patterns)
- [Best Practices](#best-practices)

## Introduction to CSS Styling

CSS styling transforms plain HTML content into visually appealing web pages. Styling properties control the appearance of elements, including their colors, fonts, spacing, and visual effects. Effective styling enhances user experience by making content more readable, accessible, and engaging.

CSS styling involves several key areas:
- Color and visual design
- Typography and text presentation
- Background and border styling
- Visual effects and transformations

## Color Theory in CSS

### Color Models and Values

CSS supports multiple color specification methods:

#### Named Colors
```css
.named-colors {
  color: red;
  background-color: lightblue;
  border-color: darkslategray;
}
```

#### Hexadecimal Colors
```css
.hex-colors {
  color: #ff0000;        /* Red */
  background-color: #f0f0f0;  /* Light gray */
  border-color: #336699;  /* Blue */
}
```

#### RGB Colors
```css
.rgb-colors {
  color: rgb(255, 0, 0);          /* Red */
  background-color: rgba(0, 0, 255, 0.5);  /* Semi-transparent blue */
  border-color: rgb(100%, 50%, 0%);        /* Orange */
}
```

#### HSL Colors
```css
.hsl-colors {
  color: hsl(0, 100%, 50%);         /* Red */
  background-color: hsla(120, 100%, 50%, 0.7);  /* Semi-transparent green */
  border-color: hsl(240, 100%, 50%);         /* Blue */
}
```

### Color Accessibility

When choosing colors, consider accessibility requirements:
- Ensure sufficient contrast between text and background
- Don't rely solely on color to convey information
- Test color combinations for colorblind users

```css
.accessible-colors {
  color: #000;           /* High contrast with white background */
  background-color: #fff;
  /* Contrast ratio: 21:1 (exceeds WCAG AA requirements) */
}
```

## Typography and Text Styling

### Font Properties

```css
.font-properties {
  font-family: Arial, sans-serif;    /* Font family stack */
  font-size: 16px;                   /* Font size */
  font-weight: bold;                 /* Font weight (100-900 or keywords) */
  font-style: italic;                /* Font style */
  font-variant: small-caps;          /* Font variant */
  line-height: 1.5;                  /* Line height */
}
```

### Text Properties

```css
.text-properties {
  text-align: center;                /* Text alignment */
  text-decoration: underline;        /* Text decoration */
  text-transform: uppercase;         /* Text transformation */
  text-shadow: 2px 2px 4px #000;    /* Text shadow */
  letter-spacing: 1px;               /* Letter spacing */
  word-spacing: 2px;                 /* Word spacing */
  white-space: nowrap;               /* White space handling */
}
```

### Web Fonts

```css
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap');

.web-fonts {
  font-family: 'Roboto', Arial, sans-serif;
  font-weight: 400;
}
```

## Background Properties

### Background Color

```css
.background-color {
  background-color: #f0f0f0;  /* Simple background color */
}
```

### Background Images

```css
.background-image {
  background-image: url('image.jpg');
  background-repeat: no-repeat;          /* Don't repeat image */
  background-position: center center;    /* Center the image */
  background-size: cover;               /* Cover entire element */
  background-attachment: fixed;         /* Fixed background during scroll */
}

/* Shorthand */
.background-shorthand {
  background: #f0f0f0 url('image.jpg') no-repeat center/cover fixed;
}
```

### Multiple Backgrounds

```css
.multiple-backgrounds {
  background-image: 
    url('overlay.png'),
    url('texture.jpg'),
    linear-gradient(45deg, #ff0000, #0000ff);
  background-repeat: no-repeat, repeat, no-repeat;
  background-position: center, top left, center;
}
```

## Border Properties

### Border Styles

```css
.border-styles {
  border-width: 2px;           /* Border width */
  border-style: solid;         /* Border style */
  border-color: #000;          /* Border color */
  
  /* Shorthand */
  border: 2px solid #000;
  
  /* Individual sides */
  border-top: 1px solid #ccc;
  border-right: 2px dashed #999;
  border-bottom: 3px dotted #666;
  border-left: 4px double #333;
}
```

### Border Radius

```css
.border-radius {
  border-radius: 10px;                    /* Uniform radius */
  border-radius: 10px 5px;               /* Top/Bottom, Left/Right */
  border-radius: 10px 5px 15px 20px;     /* Top-left, Top-right, Bottom-right, Bottom-left */
  
  /* Elliptical corners */
  border-radius: 10px / 20px;
}
```

## Visual Formatting

### Box Shadow

```css
.box-shadow {
  box-shadow: 
    2px 2px 5px 0px rgba(0, 0, 0, 0.3),    /* Horizontal, vertical, blur, spread, color */
    inset 0 0 10px rgba(0, 0, 0, 0.1);      /* Inner shadow */
}
```

### Opacity

```css
.opacity {
  opacity: 0.8;  /* 80% opacity */
}
```

### Transformations

```css
.transformations {
  transform: rotate(45deg);                 /* Rotate */
  transform: scale(1.2);                    /* Scale */
  transform: translate(10px, 20px);         /* Move */
  transform: skew(10deg, 5deg);             /* Skew */
  
  /* Multiple transforms */
  transform: rotate(45deg) scale(1.2) translate(10px, 10px);
}
```

### Transitions

```css
.transitions {
  transition: all 0.3s ease;                /* All properties, 0.3s duration, ease timing */
  transition: 
    background-color 0.2s ease,
    transform 0.3s ease-in-out;
}
```

## Styling Patterns

### Pattern 1: Card Component

```css
.card {
  background-color: white;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 20px;
  margin: 10px;
  transition: box-shadow 0.3s ease;
}

.card:hover {
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}
```

### Pattern 2: Button Styling

```css
.button {
  display: inline-block;
  padding: 12px 24px;
  background-color: #007bff;
  color: white;
  text-decoration: none;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.button:hover {
  background-color: #0056b3;
}

.button:active {
  transform: translateY(1px);
}
```

### Pattern 3: Form Styling

```css
.form-input {
  width: 100%;
  padding: 10px 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.2s ease;
}

.form-input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}
```

## Best Practices

### 1. Use System Fonts When Possible

```css
.system-fonts {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  /* Uses system fonts for better performance */
}
```

### 2. Define Color Palettes

```css
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --success-color: #28a745;
  --danger-color: #dc3545;
  --light-color: #f8f9fa;
  --dark-color: #343a40;
  
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.25rem;
}
```

### 3. Maintain Visual Hierarchy

```css
.typography-scale {
  --heading-font-size: 2rem;
  --subheading-font-size: 1.5rem;
  --body-font-size: 1rem;
  --small-font-size: 0.875rem;
  
  --heading-weight: 700;
  --body-weight: 400;
}
```

### 4. Consider Performance

```css
.performant-styling {
  /* Use transform for animations instead of changing position properties */
  transition: transform 0.3s ease;
}

.performant-styling:hover {
  transform: scale(1.05);
}
```

### 5. Ensure Accessibility

```css
.accessible-styling {
  /* Ensure focus indicators are visible */
  outline: 2px solid transparent;
  outline-offset: 2px;
}

.accessible-styling:focus {
  outline: 2px solid #4d90fe;
}
```

## Practical Exercise

Create a complete component library with:
- A set of consistently styled buttons
- Form elements with proper focus states
- Card components with hover effects
- A color palette using CSS custom properties
- Typography scale for headings and body text

## Key Takeaways

- CSS styling encompasses colors, typography, backgrounds, and visual effects
- Understanding color theory and accessibility is crucial for effective styling
- Typography significantly impacts readability and user experience
- Background and border properties add visual depth and structure
- Visual formatting properties like shadows and transforms enhance design
- Consistent styling patterns improve user interface cohesion
- Performance and accessibility should guide styling decisions

## Next Steps

Continue to [[CSS Color Systems and Palettes]] to learn about creating and managing color schemes for web projects.

## Tags

#css #styling #web-development #frontend #typography #colors #backgrounds #borders #visual-design