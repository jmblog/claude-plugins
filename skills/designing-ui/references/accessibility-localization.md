# Accessibility & Localization

Accessibility and localization principles ensure interfaces work for all users across different abilities and cultures.

## Contents
- [Screen Reader Support](#screen-reader-support)
- [Visual Accessibility](#visual-accessibility)
- [Keyboard Navigation](#keyboard-navigation)
- [Multilingual UI](#multilingual-ui)
- [Cultural Considerations](#cultural-considerations)

---

## Screen Reader Support

**Core principle**: Make all content and functionality available to screen reader users.

**Apply when**:
- Building any web interface
- Adding interactive elements
- Creating dynamic content
- Implementing custom controls

**Implementation**:

Semantic HTML:
```html
<!-- Good: Semantic -->
<header>
  <nav>
    <ul>
      <li><a href="/">Home</a></li>
      <li><a href="/about">About</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <h1>Page Title</h1>
    <p>Content</p>
  </article>
</main>

<!-- Bad: Non-semantic -->
<div class="header">
  <div class="nav">
    <span class="link">Home</span>
    <span class="link">About</span>
  </div>
</div>
```

ARIA labels:
```html
<!-- Icon buttons need labels -->
<button aria-label="Close dialog">
  <Icon name="close" />
</button>

<!-- Form inputs need labels -->
<label for="email">Email address</label>
<input id="email" type="email" />

<!-- Status updates -->
<div role="status" aria-live="polite">
  {message}
</div>

<!-- Loading states -->
<button disabled aria-busy="true">
  <Spinner /> Loading...
</button>
```

Skip links:
```html
<a href="#main-content" class="skip-link">
  Skip to main content
</a>

<style>
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
}

.skip-link:focus {
  top: 0;
}
</style>
```

**Examples**:

Good example: Accessible form
```html
<form>
  <div class="form-group">
    <label for="name">
      Full Name
      <span aria-label="required">*</span>
    </label>
    <input
      id="name"
      type="text"
      required
      aria-required="true"
      aria-describedby="name-hint"
    />
    <span id="name-hint" class="hint">
      First and last name
    </span>
  </div>

  {error && (
    <div role="alert" aria-live="assertive">
      {error}
    </div>
  )}
</form>
```

**Anti-patterns**:
- Images without alt text
- Icon buttons without labels
- Custom controls without ARIA
- Dynamic content without announcements

**Related guidelines**: Use User Language (11), Meaningful Interactive Elements (37)

---

## Visual Accessibility

**Core principle**: Ensure sufficient color contrast and don't rely solely on color.

**Apply when**:
- Choosing color schemes
- Designing text and backgrounds
- Indicating states or status
- Creating data visualizations

**Implementation**:

Color contrast (WCAG AA):
```css
/* Text contrast ratios */
/* Normal text: 4.5:1 minimum */
.text {
  color: #333333; /* On white background */
  /* Ratio: 12.6:1 ✓ */
}

/* Large text (18pt+ or 14pt+ bold): 3:1 minimum */
.heading {
  font-size: 24px;
  color: #666666; /* On white */
  /* Ratio: 5.7:1 ✓ */
}

/* Interactive elements: 3:1 minimum */
.button {
  background: #0066cc;
  color: #ffffff;
  /* Ratio: 7:1 ✓ */
}
```

Don't rely on color alone:
```html
<!-- Good: Color + icon + text -->
<Alert variant="error">
  <Icon name="alert-circle" />
  <strong>Error:</strong> Please fix the errors below
</Alert>

<!-- Bad: Color only -->
<div style="color: red">
  Please fix the errors below
</div>

<!-- Good: Multiple indicators -->
<FormField error={hasError}>
  <Input
    className={hasError ? 'error' : ''}
    aria-invalid={hasError}
  />
  {hasError && (
    <ErrorIcon />
    <ErrorText>Invalid input</ErrorText>
  )}
</FormField>
```

Text size adjustment:
```css
/* Use relative units */
body {
  font-size: 16px; /* Base size */
}

h1 {
  font-size: 2rem; /* Scales with user preference */
}

p {
  font-size: 1rem;
  line-height: 1.5; /* Readable line spacing */
}

/* Support zoom up to 200% */
@media (min-resolution: 2dppx) {
  /* Adjust as needed */
}
```

**Examples**:

Good example: Status indicators
```html
<Status status={status}>
  {status === 'success' && (
    <>
      <Icon name="check-circle" color="green" />
      <Text color="green" weight="600">Complete</Text>
    </>
  )}
  {status === 'error' && (
    <>
      <Icon name="alert-circle" color="red" />
      <Text color="red" weight="600">Failed</Text>
    </>
  )}
</Status>
```

**Anti-patterns**:
- Low contrast text (gray on gray)
- Color as only indicator
- Fixed font sizes (px only)
- Disabled text with insufficient contrast

**Related guidelines**: Visual Hierarchy, Signifiers (4)

---

## Keyboard Navigation

**Core principle**: All functionality must be accessible via keyboard.

**Apply when**:
- Implementing interactive elements
- Creating custom controls
- Building modal dialogs
- Designing navigation

**Implementation**:

Focus management:
```css
/* Visible focus indicator */
*:focus {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}

/* Custom focus styles */
.button:focus {
  box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.3);
}
```

Tab order:
```html
<!-- Logical tab order -->
<form>
  <input tabindex="0" /> <!-- Auto tab order -->
  <input tabindex="0" />
  <button tabindex="0">Submit</button>

  <!-- Skip navigation shortcut -->
  <input tabindex="-1" /> <!-- Not in tab order, but focusable -->
</form>
```

Keyboard shortcuts:
```javascript
// Implement common shortcuts
useEffect(() => {
  const handleKeyDown = (e) => {
    // Ctrl/Cmd + S to save
    if ((e.ctrlKey || e.metaKey) && e.key === 's') {
      e.preventDefault();
      handleSave();
    }

    // Escape to close modal
    if (e.key === 'Escape' && modalOpen) {
      closeModal();
    }

    // Arrow keys for navigation
    if (e.key === 'ArrowDown') {
      focusNext();
    }
  };

  document.addEventListener('keydown', handleKeyDown);
  return () => document.removeEventListener('keydown', handleKeyDown);
}, []);
```

**Examples**:

Good example: Accessible modal
```javascript
const Modal = ({ isOpen, onClose, children }) => {
  const modalRef = useRef();

  useEffect(() => {
    if (isOpen) {
      // Save last focused element
      const lastFocused = document.activeElement;

      // Focus modal
      modalRef.current?.focus();

      // Trap focus in modal
      const handleTab = (e) => {
        if (e.key === 'Tab') {
          const focusable = modalRef.current.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
          );
          const first = focusable[0];
          const last = focusable[focusable.length - 1];

          if (e.shiftKey && document.activeElement === first) {
            last.focus();
            e.preventDefault();
          } else if (!e.shiftKey && document.activeElement === last) {
            first.focus();
            e.preventDefault();
          }
        }
      };

      document.addEventListener('keydown', handleTab);

      return () => {
        document.removeEventListener('keydown', handleTab);
        lastFocused?.focus(); // Restore focus
      };
    }
  }, [isOpen]);

  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      tabIndex={-1}
    >
      {children}
    </div>
  );
};
```

**Anti-patterns**:
- No focus indicators
- Keyboard traps
- Illogical tab order
- Missing keyboard shortcuts
- Interactive elements not focusable

**Related guidelines**: User Control (7), Escape Hatch (59)

---

## Multilingual UI

**Core principle**: Design to accommodate text length variations across languages.

**Apply when**:
- Designing layouts
- Creating buttons and labels
- Building forms
- Planning responsive designs

**Implementation**:

Flexible layouts:
```css
/* Allow text expansion */
.button {
  padding: 12px 24px;
  min-width: 120px; /* Accommodate longer text */
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Flexible grids */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  /* Adjusts to content length */
}
```

Text expansion planning:
```javascript
// Languages expand differently
// German: +30% longer than English
// French: +15-20%
// Spanish: +15-20%
// Italian: +10-15%

// Design with 30% buffer
const buttonWidth = baseWidth * 1.3;
```

RTL support:
```css
/* Support right-to-left languages */
[dir="rtl"] .layout {
  direction: rtl;
}

/* Use logical properties */
.element {
  margin-inline-start: 16px; /* Works for both LTR and RTL */
  padding-inline-end: 16px;
}

/* Instead of */
.element {
  margin-left: 16px; /* LTR only */
}
```

**Examples**:

Good example: Flexible button
```html
<!-- English -->
<Button>Save</Button>

<!-- German (longer) -->
<Button>Speichern</Button>

<!-- Both fit due to flexible width -->
<style>
button {
  padding: 12px 24px;
  min-width: fit-content;
}
</style>
```

**Anti-patterns**:
- Fixed-width containers
- Hardcoded dimensions
- No RTL support
- Truncated text in other languages

**Related guidelines**: Layout Consistency (83), Visual Gestalt (10)

---

## Cultural Considerations

**Core principle**: Avoid culture-specific symbols and idioms in international interfaces.

**Apply when**:
- Choosing icons
- Creating illustrations
- Writing copy
- Designing for global audiences

**Implementation**:

Culturally neutral icons:
```javascript
// Good: Universal symbols
<Icon name="home" /> // House
<Icon name="search" /> // Magnifying glass
<Icon name="settings" /> // Gear

// Avoid: Culture-specific
❌ Mailbox (varies by country)
❌ Telephone handset (outdated in some regions)
❌ ○ × △ (Japanese-specific meaning)
```

Number and date formats:
```javascript
// Use locale-appropriate formatting
const formatter = new Intl.NumberFormat(userLocale, {
  style: 'currency',
  currency: userCurrency
});

const dateFormatter = new Intl.DateTimeFormat(userLocale, {
  year: 'numeric',
  month: 'long',
  day: 'numeric'
});

// US: $1,234.56, December 25, 2024
// DE: 1.234,56 €, 25. Dezember 2024
// JP: ¥1,234, 2024年12月25日
```

Avoid idioms:
```javascript
// Good: Clear, literal
"Delete this item?"

// Bad: Culture-specific idiom
"Throw this in the trash?"
```

**Examples**:

Good example: Universal confirmation
```html
<Dialog>
  <Icon name="alert-triangle" /> <!-- Universal warning -->
  <h2>Delete item?</h2>
  <p>This action cannot be undone.</p>
  <Button variant="danger">Delete</Button>
  <Button variant="secondary">Cancel</Button>
</Dialog>
```

**Related guidelines**: Use User Language (11), Signifiers (4)

---

## Accessibility Checklist

**Screen Readers**:
- [ ] Semantic HTML used
- [ ] All images have alt text
- [ ] Interactive elements have labels
- [ ] Dynamic content announced
- [ ] Skip links provided

**Visual**:
- [ ] Color contrast meets WCAG AA (4.5:1 text, 3:1 UI)
- [ ] Information not conveyed by color alone
- [ ] Text is resizable
- [ ] Focus indicators visible

**Keyboard**:
- [ ] All functionality keyboard-accessible
- [ ] Logical tab order
- [ ] Focus management in modals
- [ ] Keyboard shortcuts provided

**Localization**:
- [ ] Flexible layouts for text expansion
- [ ] RTL support if needed
- [ ] Culturally neutral icons
- [ ] Locale-appropriate formatting

**Related guidelines**: Use User Language (11), Simplify (1)
