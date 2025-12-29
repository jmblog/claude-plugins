# Visual Design

Visual design principles guide how interfaces look and communicate through color, layout, and visual hierarchy.

## Contents
- [Visual Gestalt](#visual-gestalt)
- [Graphical Tone & Manner Consistency](#graphical-tone--manner-consistency)
- [Layout Consistency](#layout-consistency)
- [Visual Hierarchy](#visual-hierarchy)
- [Visual Restraint](#visual-restraint)
- [Optical Illusions](#optical-illusions)

---

## Visual Gestalt

**Core principle**: Use proximity, similarity, and closure to show relationships and create cohesive layouts.

**Apply when**:
- Organizing form fields and controls
- Designing page layouts
- Grouping related content
- Creating navigation structures

**Implementation**:

**Proximity**: Elements close together appear related
```css
.form-section {
  margin-bottom: 32px; /* Large gap between sections */
}

.form-field {
  margin-bottom: 8px; /* Small gap within section */
}
```

**Similarity**: Similar-looking elements appear related
```css
/* All primary buttons look same */
.btn-primary {
  background: #0066cc;
  color: white;
  border-radius: 4px;
}

/* All links look same */
a {
  color: #0066cc;
  text-decoration: underline;
}
```

**Closure**: Users fill gaps to perceive complete shapes
```html
<!-- Breadcrumb navigation -->
<nav>Home > Products > Electronics</nav>
<!-- Users perceive hierarchy even with minimal visual elements -->
```

**Common Fate**: Elements moving together appear grouped
```javascript
// When accordion opens, content moves together
// indicating it belongs to same section
```

**Examples**:

Good example: Card grid using similarity
```html
<div class="card-grid">
  <div class="card">
    <img src="product1.jpg" />
    <h3>Product Name</h3>
    <p>$29.99</p>
  </div>
  <div class="card">
    <img src="product2.jpg" />
    <h3>Product Name</h3>
    <p>$39.99</p>
  </div>
</div>

<style>
.card {
  /* All cards identical structure */
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 24px; /* Consistent spacing */
}
</style>
```

**Anti-patterns**:
- Equal spacing everywhere (no grouping)
- Mixing styles for related elements
- Related elements that look unrelated

**Related guidelines**: Layout Consistency (83), Visual Restraint (82)

---

## Graphical Tone & Manner Consistency

**Core principle**: Align all graphic elements' visual characteristics: color tone, saturation, gradients, borders, shadows, line styles.

**Apply when**:
- Creating design systems
- Designing UI component libraries
- Establishing brand guidelines
- Reviewing visual consistency

**Implementation**:

Define consistent visual properties:
```css
/* Color palette */
:root {
  --primary: #0066cc;
  --secondary: #6c757d;
  --success: #28a745;
  --danger: #dc3545;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);

  /* Border radius */
  --radius-sm: 2px;
  --radius-md: 4px;
  --radius-lg: 8px;

  /* Spacing */
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  --space-xl: 32px;
}
```

Apply consistently:
```css
/* All buttons use same tone */
.button {
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-sm);
  transition: all 0.2s;
}

/* All cards use same tone */
.card {
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-md);
  border: 1px solid #e0e0e0;
}
```

**Examples**:

Good example: Consistent icon style
```javascript
// All icons same style
<Icon name="user" style="outline" size={24} />
<Icon name="settings" style="outline" size={24} />
<Icon name="logout" style="outline" size={24} />

// Not mixing:
<SolidIcon name="user" />
<OutlineIcon name="settings" />
<FilledIcon name="logout" />
```

**Anti-patterns**:
- Mixing flat and skeuomorphic styles
- Inconsistent shadows/borders
- Varying color saturation randomly
- Different icon styles

**Related guidelines**: Consistency (6), Visual Restraint (82)

---

## Layout Consistency

**Core principle**: Use grid-based layouts with consistent margins, padding, and alignment.

**Apply when**:
- Designing page layouts
- Creating responsive designs
- Building component libraries

**Implementation**:

Use grid system:
```css
.container {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 24px;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
}

/* Consistent spacing scale */
.section {
  margin-bottom: 48px;
}

.subsection {
  margin-bottom: 24px;
}

.element {
  margin-bottom: 12px;
}
```

Align to grid:
```html
<div class="container">
  <header class="col-span-12">Header</header>
  <aside class="col-span-3">Sidebar</aside>
  <main class="col-span-9">Content</main>
</div>
```

**Examples**:

Good example: Consistent alignment
```css
/* All content aligns to grid */
.text-block {
  max-width: 65ch; /* Readable line length */
  margin: 0 auto;
}

.image {
  max-width: 100%;
  height: auto;
}

/* Consistent vertical rhythm */
h1 { margin-bottom: 24px; }
h2 { margin-bottom: 16px; }
p { margin-bottom: 16px; }
```

**Anti-patterns**:
- Random spacing values
- Misaligned elements
- Inconsistent margins/padding
- No grid system

**Related guidelines**: Visual Gestalt (10), Visual Restraint (82)

---

## Visual Hierarchy

**Core principle**: Guide attention through size, color, contrast, and positioning.

**Apply when**:
- Designing pages and screens
- Creating content layouts
- Establishing information priority

**Implementation**:

Use size for hierarchy:
```css
h1 { font-size: 32px; font-weight: 700; }
h2 { font-size: 24px; font-weight: 600; }
h3 { font-size: 18px; font-weight: 600; }
p { font-size: 16px; font-weight: 400; }
```

Use color for emphasis:
```css
.primary-action {
  background: #0066cc;
  color: white;
  font-weight: 600;
}

.secondary-action {
  background: transparent;
  color: #0066cc;
  border: 1px solid #0066cc;
}

.tertiary-action {
  background: transparent;
  color: #6c757d;
}
```

Use positioning:
```html
<!-- Most important content: top-left -->
<Layout>
  <Header>
    <Logo /> <!-- Top-left corner -->
    <MainNav /> <!-- Prominent position -->
  </Header>

  <Main>
    <h1>Primary Heading</h1> <!-- Largest, first -->
    <p>Important content first</p>
    <aside>Secondary content</aside> <!-- Smaller, to side -->
  </Main>
</Layout>
```

**Examples**:

Good example: Clear priority
```html
<Card>
  <h2 style="font-size: 24px; font-weight: 700">
    Main Title
  </h2>
  <p style="font-size: 16px; color: #333">
    Primary description
  </p>
  <span style="font-size: 14px; color: #666">
    Secondary metadata
  </span>
</Card>
```

**Anti-patterns**:
- All text same size/weight
- No clear entry point
- Equal visual weight for all elements
- Confusing hierarchy

**Related guidelines**: Communicate Information Not Data (28), Simplify (1)

---

## Visual Restraint

**Core principle**: Limit color and font variations. Avoid visual clutter.

**Apply when**:
- Creating design systems
- Reviewing interfaces for clarity
- Simplifying complex screens

**Implementation**:

Limit color palette:
```css
/* Primary colors (3-5 max) */
--primary: #0066cc;
--secondary: #6c757d;
--success: #28a745;
--warning: #ffc107;
--danger: #dc3545;

/* Grays for text and borders */
--gray-900: #212529;
--gray-700: #495057;
--gray-500: #adb5bd;
--gray-300: #dee2e6;
```

Limit typography:
```css
/* 2 font families max */
--font-heading: 'Inter', sans-serif;
--font-body: 'Inter', sans-serif;

/* Limited weights */
--weight-normal: 400;
--weight-medium: 500;
--weight-bold: 700;

/* Type scale (5-7 sizes) */
--text-xs: 12px;
--text-sm: 14px;
--text-base: 16px;
--text-lg: 18px;
--text-xl: 24px;
--text-2xl: 32px;
```

**Anti-patterns**:
- Rainbow color schemes
- Many font families
- Excessive decoration
- Visual noise

**Related guidelines**: Simplify (1), Graphical Tone & Manner (27)

---

## Optical Illusions

**Core principle**: Account for visual illusions when designing. Manually adjust designs to counteract perceptual distortions.

**Apply when**:
- Aligning shapes and icons
- Creating balanced layouts
- Designing logos and graphics

**Implementation**:

**Optical centering**:
```css
/* Visual center is not mathematical center */
.icon-button {
  /* Icon appears centered when slightly higher */
  padding: 12px 16px 14px 16px; /* More bottom padding */
}
```

**Size perception**:
```css
/* Circles appear smaller than squares of same size */
.circle-icon {
  width: 26px; /* Slightly larger */
  height: 26px;
}

.square-icon {
  width: 24px;
  height: 24px;
}
```

**Weight perception**:
```css
/* White on dark appears heavier than dark on white */
.dark-background {
  font-weight: 400; /* Normal weight looks good */
}

.light-background {
  font-weight: 500; /* Needs slightly heavier weight */
}
```

**Examples**:

Good example: Optical alignment
```html
<!-- Triangle play button needs offset to appear centered -->
<svg viewBox="0 0 24 24">
  <path d="M8 5v14l11-7z" /> <!-- Shifted right for visual center -->
</svg>

<!-- Not mathematically centered but looks centered -->
```

**Related guidelines**: Visual Hierarchy, Layout Consistency (83)
