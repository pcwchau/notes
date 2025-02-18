# Index

- [Index](#index)
- [HTML Elements](#html-elements)
  - [Container](#container)
    - [div](#div)
    - [span](#span)
  - [Content](#content)
    - [nav](#nav)
- [CSS](#css)
  - [Overview](#overview)
  - [Syntax](#syntax)
    - [Selector](#selector)
    - [Usage](#usage)
  - [Layout](#layout)
    - [Outer Display Type](#outer-display-type)
    - [Inner Display Type](#inner-display-type)
    - [Position](#position)
  - [Properties](#properties)
    - [Spacing](#spacing)
    - [Sizing](#sizing)
  - [Length](#length)
  - [Rules](#rules)
    - [@media](#media)
- [Tailwind CSS](#tailwind-css)
  - [Class](#class)

# HTML Elements

- [HTML elements reference - MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)

## Container

### div

It is a block container to group elements for styling purpose. It will take the entire row of the screen.

It should be used only when no other semantic element (such as `<article>` or `<nav>`) is appropriate.

### span

It is an inline container to group elements for styling purpose. It just takes the size of its children.

## Content

The elements in this category should be semantically meaningful.

Advantages:

- Make the code more meaningful and easier to understand for developers and browsers
- Search engines can better understand your site's structure
- Improve the accessibility of the webpage

### nav

It is a block container and should be used to provide navigation links.

# CSS

## Overview

All browsers have their default styles - very basic styling that the browser applies to HTML to make sure that the page will be basically readable even if no explicit styling is specified by the author of the page. These styles are defined in default CSS stylesheets contained within the browser.

For example, headings will look larger than regular text, paragraphs break onto a new line and have space between them. Links are colored and underlined to distinguish them from the rest of the text.

Using CSS, you can control exactly how HTML elements look in the browser, including the design, layout and variations in display for different devices and screen sizes.

## Syntax

```
<selector> {
  <property>: <value>;
}
```

### Selector

```
p { ... }        // Element selector, e.g. all <p> elements
#para1 { ... }   // Id selector, e.g. all HTML element with id="para1"
.center { ... }  // Class selector, e.g. all HTML elements with class="center"

You can group different selectors:

h1, h2, p { ... } // All of them apply the same style
```

**Declaring global CSS variables**

```css
/* Declare global variables */
:root {
  --main-color: hotpink;
  --pane-padding: 5px 42px;
}

/* Use global variables */
body {
  color: var(--foreground);
  background: var(--background);
  font-family: Arial, Helvetica, sans-serif;
}
```

### Usage

External CSS:

```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="mystyle.css">
  </head>
  <body>
  </body>
</html>
```

Internal CSS:

```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      body {
        background-color: linen;
      }
    </style>
  </head>
  <body>
  </body>
</html>
```

Inline CSS:

```html
<!DOCTYPE html>
<html>
<body>

<h1 style="color:blue;text-align:center;">This is a heading</h1>
<p style="color:red;">This is a paragraph.</p>

</body>
</html>
```

All the styles in a page will "cascade" into a new "virtual" style sheet by the following rules, where the first one has the highest priority:

- Inline style (inside an HTML element)
- External and internal style sheets (in the head section)
- Browser default

## Layout

- [CSS Flow Layout - MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flow_layout)

CSS page layout refers to the techniques used to structure and position elements on a webpage. It determines how content is arranged, how elements interact, and how the page adapts to different screen sizes.

The `display` property is the most important CSS property for controlling layout, including:

- the outer display type - the element itself
- the inner display type - the layout of the element's children

### Outer Display Type

The outer display type sets whether an element is treated as a block or inline box in a normal flow.

| Value of `display` | Description                              |
|--------------------|------------------------------------------|
| `block`            | Displays an element as a block element   |
| `inline`           | Displays an element as an inline element |

**Block elements**

In normal flow, block elements display one after the other vertically, start at the top and move down the page. They span 100% of the width of their parent element, and are as tall as their child content.

If two vertically adjacent elements both have a margin set on them and their margins touch, the larger of the two margins remains and the smaller one disappears.

Examples of block elements include `<div>`, `<h1> ... <h6>`, `<p>`, `<form>`, `<header>`, `<footer>`, `<section>`.

**Inline elements**

In normal flow, inline elements sit on the same line along with any adjacent (or wrapped) text content as long as there is space for them to do so inside the width of the parent block level element. If there isn't space, then the overflowing content will move down to a new line.

Examples of inline elements include `<span>`, `<a>`, `<img>`.

### Inner Display Type

The inner display type of an element sets the layout used for its children.

| Value of `display`  | Description |
|---------------------|-------------|
| `flex` | Display an element as a block element and lay out its children according to the flexbox model |
| `grid` | Display an element as a block element and lay out its children according to the grid model |

**Flexbox Layout Model**

- [CSS flexible box layout - MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout)

The CSS flexible box layout (Flexbox) is a layout model that makes it easier to design flexible and responsive layouts.

In the flex layout model, the children of a flex container can be laid out in any direction, and can "flex" their sizes, either growing to fill unused space or shrinking to avoid overflowing the parent. Both horizontal and vertical alignment of the children can be easily manipulated.

To use a flexbox, a container is needed to be set to `display: flex`.

Properties related to a flexbox:

| Property          | Default  | Description |
|-------------------|----------|-------------|
| `align-content`   |
| `align-items`     | `normal` | Control alignment of children on the cross axis
| `align-self`      |
| `flex`            |
| `flex-basis`      |
| `flex-direction`  | `row`    |
| `flex-flow`       |
| `flex-grow`       |
| `flex-shrink`     |
| `flex-wrap`       |
| `justify-content` | `normal` | Control alignment of children on the main axis

### Position

- https://developer.mozilla.org/en-US/docs/Web/CSS/position

The `position` property sets how an element is positioned in a document.

The `top`, `right`, `bottom`, and `left` properties determine the final location of positioned elements, whose value is either `relative`, `absolute`, `fixed`, or `sticky`.

The `z-index` property controls the stacking order of elements along the Z-axis (depth). Elements with a higher z-index appear in front of elements with a lower z-index.

**Types of positioning**

`position: static` is the default positioning for all elements.

- The element follows the normal flow.
- It is not affected by `top`, `right`, `bottom`, `left`, or `z-index` properties.

`position: relative`

- The element stays in the normal flow but can offset from the original position.
- Space is given for the element in the layout. Thus, the offset does not affect the position of any other elements.

`position: absolute`

- The element is removed from the normal flow. Thus, no space is created in the layout.
- Positioned relative to the nearest positioned ancestor. If no such ancestor exists, it is positioned relative to `<html>`.

`position: fixed`

- The element is removed from the normal flow. Thus, no space is created in the layour.
- Positioned relative to the `<html>` (does not move when scrolling).

`position: sticky`

- Acts like `relative` until it reaches a scroll position, then behaves like `fixed`.

## Properties

- [CSS Properties Reference - w3schools](https://www.w3schools.com/cssref/index.php)
- [CSS Reference -> Properties - MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference)

### Spacing

**`margin`**

**`padding`**

### Sizing

**`width`**

`width: auto`

In normal flow, it will try as hard as possible to keep an element the same width as its parent container when additional space is added from margins, padding, or borders.

Elements with `position: fixed` or `position: absolute` are sized to fit their contents.

`width: 100%`

It will make the element as wide as the parent container. Extra spacing will be added to the element's size without regards to the parent. This typically causes problems.

**`height`**

## Length

- https://developer.mozilla.org/en-US/docs/Web/CSS/length
- Use `px` if need hardcode
- Use `em / rem` if based on font size
  - `1 em / rem = 1 font size`, default font size = 16px
  - `rem` inherit HTML root size, `em` inherit parent element size
- Use `vh / vw` if need responsive design (view height / view width)

## Rules

### @media

`@media` is a CSS media query that allows you to apply different styles depending on the screen size, resolution, or device type. It is commonly used for responsive design, ensuring that your website looks good on different devices like mobile, tablet, and desktop.

**max-width / min-width**

```css
/* If width is 600px or smaller */
@media (max-width: 600px) {
  body {
    background: lightblue;
  }
}

/* If width is 1024px or larger */
@media (min-width: 1024px) {
  .container {
    width: 80%;
    margin: auto;
  }
}
```

**prefers-color-scheme**

```css
@media (prefers-color-scheme: dark) {
  :root {
    --background: #0a0a0a;
    --foreground: #ededed;
  }
}
```


# Tailwind CSS

- https://tailwindcss.com/docs/styling-with-utility-classes

## Class

Responsive design: `container`
