# Frontend Technical Assessment: Laborant (Junior/Intern)

This assessment evaluates fundamental CSS logic, browser rendering behavior, and architectural best practices.

---

## Level 1: Foundations
**Topics:** Frameworks layout techniques, Selector specificity/weight, Positioning (basic), Margins vs. Paddings.

### 1. Expanded Concept Explanations

**1. Frameworks Layout Techniques:**
Frameworks (like Bootstrap or Tailwind) often use utility classes to handle layout, but understanding the underlying mechanism is key. Most modern frameworks rely on a grid system (historically floats, now Flexbox/Grid) to create responsive columns. A "Laborant" must know that these classes are just pre-written CSS rules, not magic, and that relying solely on them without understanding CSS flow leads to brittle code.

**2. Selector Specificity/Weight:**
CSS Specificity determines which styles are applied when multiple rules target the same element. It is calculated based on a weight system:
* **Inline styles:** 1000 points (Overrides almost everything)
* **IDs:** 100 points each (High priority)
* **Classes, attributes, pseudo-classes:** 10 points each
* **Elements, pseudo-elements:** 1 point each
* *Note:* `!important` overrides this natural hierarchy but is considered a "nuclear option."

**3. Positioning (Basic):**
* **Static (Default):** The element follows the normal document flow. `top`, `right`, `bottom`, `left` have no effect.
* **Relative:** The element remains in the document flow, but can be visually offset relative to its original position.
* **Absolute:** The element is removed from the document flow and positioned relative to its nearest *positioned* ancestor (non-static).

**4. Margins vs. Paddings:**
* **Padding:** The space *inside* the border. It pushes the content inward and takes on the background color of the element.
* **Margin:** The space *outside* the border. It pushes neighbor elements away and is always transparent. Margins can "collapse" vertically, while padding never does.

### 2. Variant Question
"If you have a generic button class `.btn { background: blue; }` and a specific ID `#submit-btn { background: red; }`, what color will the button be? If you then add `!important` to the class, what happens, and why is doing so generally discouraged?"

### 3. Correct Answer
The button will be **red** because an ID (100 points) outweighs a class (10 points). Adding `!important` to the class would turn it blue, but this is discouraged because it breaks the natural cascading logic, forcing other developers to use even more `!important` tags to override your styles later (specificity wars).

### 4. Practical Code Snippet
```css
/* Specificity Hierarchy Example */
#nav-id { color: red; }          /* Weight: 100 (Wins) */
.nav-class { color: blue; }      /* Weight: 10 */
nav { color: green; }            /* Weight: 1 */
```

### 5. Coding Task
**Question:** You have a `div` container. You want to push it 50px away from the header above it, but you also want the text inside the `div` to be 20px away from the `div`'s own border. Write the required CSS.
**Answer:**
```css
.container {
  margin-top: 50px;  /* External space */
  padding: 20px;     /* Internal space */
}
```

---

## Level 2: Visibility and Display
**Topics:** Custom font implementation (@font-face), Element visibility (none vs hidden), Display types (Inline vs Block vs Inline-block).

### 1. Expanded Concept Explanations

**1. Custom Font Implementation (@font-face):**
To use a non-standard font, we must declare it using `@font-face` before assigning it. Key properties include:
* `font-family`: The name you assign to reference the font later.
* `src`: The path to the font file (e.g., .woff2, .ttf).
* `font-display`: Controls loading behavior (e.g., `swap` shows a fallback font immediately until the custom font loads).

**2. Element Visibility (None vs. Hidden):**
* **`display: none`**: Completely removes the element from the document flow. It takes up **zero space**, causing surrounding elements to shift to fill the gap.
* **`visibility: hidden`**: Hides the element visually, but the element **retains its physical space** in the layout. It is "invisible but present."

**3. Display Types:**
* **`block`**: Takes up the full width available and starts on a new line (e.g., `<div>`, `<p>`).
* **`inline`**: Takes only as much width as necessary, sits within text flow, and **ignores** width/height/top-bottom margins (e.g., `<span>`, `<a>`).
* **`inline-block`**: Flows like text (side-by-side) but allows you to set width, height, and margins (e.g., buttons in a toolbar).

### 2. Variant Question
"You are creating a 'Card' component. While the image is loading, you want the empty space for the image to remain reserved so the text below doesn't jump around. Should you use `display: none` or `visibility: hidden` on the image container?"

### 3. Correct Answer
You should use **`visibility: hidden`** (or a defined aspect-ratio). Using `display: none` would remove the element's height, causing the content below to shift up, only to jump back down when the image loads (Cumulative Layout Shift).

### 4. Practical Code Snippet
```css
/* Font Face Implementation */
@font-face {
  font-family: 'MyCustomFont';
  src: url('/fonts/font.woff2') format('woff2');
  font-display: swap; /* Performance best practice */
}
```

### 5. Coding Task
**Question:** Transform a standard, vertical list of links (`<a>` inside `<li>`) into a horizontal navigation bar where each link has a defined clickable area of 40px height.
**Answer:**
```css
li {
  display: inline-block; /* Allows items to sit side-by-side */
}
a {
  display: block;        /* Makes the anchor fill the list item */
  height: 40px;          /* Enforces clickable height */
  padding: 0 10px;
}
```

---

## Level 3: Layout Logic and Depth
**Topics:** Z-index and Stacking Contexts, Advanced positioning (Fixed/Sticky), Display types (Flex vs Grid).

### 1. Expanded Concept Explanations

**1. Z-index and Stacking Contexts:**
`z-index` controls the vertical stacking order, but it is **relative** to the Stacking Context. A high z-index (e.g., 999) inside a container with a low z-index (e.g., 1) cannot appear above an element in a completely different, higher context. A Stacking Context is created by properties like `opacity < 1`, `transform`, `filter`, or `position: relative/absolute` with a z-index.

**2. Advanced Positioning (Fixed/Sticky):**
* **`fixed`**: Positioned relative to the **viewport** (browser window). It stays in the same place even when scrolling.
* **`sticky`**: Toggles between `relative` and `fixed` depending on the scroll position. It behaves normally until it hits a defined threshold (e.g., `top: 0`), then it "sticks."

**3. Display Types (Flex vs. Grid):**
* **Flexbox (1-Dimensional):** Designed for laying out items in a single row OR a single column. Ideal for alignment and distribution of space.
* **Grid (2-Dimensional):** Designed for laying out items in rows AND columns simultaneously. Ideal for complex page structures.

### 2. Variant Question
"I have a modal window with `z-index: 9999`, but it is still appearing *behind* a website header that has `z-index: 10`. What is the likely cause involving the modal's parent container?"

### 3. Correct Answer
The modal is likely nested inside a parent container that has created a new, lower **Stacking Context** (e.g., the parent has `opacity: 0.9` or `z-index: 5`). The modal is trapped in that "layer 5" context and cannot rise above the header's "layer 10", regardless of its own internal z-index.

### 4. Practical Code Snippet
```css
.sticky-nav {
  position: sticky;
  top: 0;       /* Essential: Sticky won't work without a threshold */
  z-index: 100;
}
```

### 5. Coding Task
**Question:** Using **Flexbox**, write the CSS to center a child element both vertically and horizontally within a parent container that fills the entire browser viewport.
**Answer:**
```css
.parent {
  display: flex;
  justify-content: center; /* Horizontal center */
  align-items: center;     /* Vertical center */
  height: 100vh;           /* Full viewport height */
}
```

---

## Level 4: Architecture and Responsiveness
**Topics:** Flexbox alignment logic, Responsive Design (Mobile-first concept), CSS Box Model (content-box vs border-box).

### 1. Expanded Concept Explanations

**1. Flexbox Alignment Logic:**
Flexbox alignment relies on two axes:
* **Main Axis:** Defined by `flex-direction` (row or column). `justify-content` aligns items along this axis.
* **Cross Axis:** Perpendicular to the main axis. `align-items` aligns items along this axis.
* *Note:* If direction is `column`, `justify-content` controls vertical alignment, and `align-items` controls horizontal.

**2. Responsive Design (Mobile-first):**
This strategy involves writing CSS for mobile devices (smallest screens) as the default, base styles. We then use `min-width` media queries to *add* complexity for tablets and desktops. This results in cleaner code and better performance than "Desktop-first" (which requires un-doing styles with `max-width`).

**3. CSS Box Model (content-box vs. border-box):**
* **`content-box` (Default):** The `width` property only sets the width of the content area. Padding and Border are *added* to this width (Total = Width + Padding + Border).
* **`border-box` (Modern Standard):** The `width` property includes Content, Padding, and Border. If you set `width: 50%` and add padding, the element stays at 50%, preventing layout breakage.

### 2. Variant Question
"Why do developers universally reset `box-sizing` to `border-box` at the start of a project? Specifically, how does it help when creating a grid where two columns are set to `width: 50%`?"

### 3. Correct Answer
If you leave the default (`content-box`), adding `padding: 10px` to a `width: 50%` column makes the actual calculated width `50% + 20px`. This makes the total width > 100%, pushing the second column onto a new line. `border-box` forces the padding to remain *inside* the 50% width, keeping the grid intact.

### 4. Practical Code Snippet
```css
/* Mobile-First Media Query Structure */
.container {
  flex-direction: column; /* Default: Stacked for mobile */
}

@media (min-width: 768px) {
  .container {
    flex-direction: row;  /* Changed: Side-by-side for desktop */
  }
}
```

### 5. Coding Task
**Question:** Write the global CSS reset rule that applies `border-box` sizing to every element in the application, including pseudo-elements.
**Answer:**
```css
*, *::before, *::after {
  box-sizing: border-box;
}
```