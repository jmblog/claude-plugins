# Interaction Patterns

Interaction patterns define how users engage with interface elements and complete tasks.

## Contents
- [Input Forms](#input-forms)
- [Buttons & Actions](#buttons--actions)
- [Data Input](#data-input)
- [Error Handling](#error-handling)
- [Interaction Targets](#interaction-targets)

---

## Input Forms

### Form Storytelling & Flow

**Core principle**: Group fields meaningfully. Order from simple to complex.

**Implementation**:
```html
<Form>
  <!-- Start simple -->
  <Section>
    <h3>Basic Information</h3>
    <Input name="name" required />
    <Input name="email" type="email" required />
  </Section>

  <!-- Progress to complex -->
  <Section>
    <h3>Address</h3>
    <Input name="street" />
    <Input name="city" />
    <Select name="state" />
    <Input name="zip" />
  </Section>

  <!-- Advanced at end -->
  <Section>
    <h3>Preferences (Optional)</h3>
    <Checkbox name="newsletter" />
    <Select name="notifications" />
  </Section>
</Form>
```

**Related guidelines**: Make it Easy (2), Progressive Disclosure (65)

### Good Defaults

**Core principle**: Use high-probability default values.

**Implementation**:
```javascript
// Consider: risk, commonality, neutrality, current state
const defaults = {
  country: getUserCountry(), // From browser/IP
  timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
  language: navigator.language,
  theme: matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light'
};

<Select name="country" defaultValue={defaults.country}>
  <option value="US">United States</option>
  <option value="CA">Canada</option>
  ...
</Select>
```

**Related guidelines**: Precomputation (14), Make it Easy (2)

### Constrained Controls

**Core principle**: Use selection controls instead of free text when options are known.

**Implementation**:
```html
<!-- Good: Constrained -->
<Select name="state">
  <option value="CA">California</option>
  <option value="NY">New York</option>
</Select>

<!-- Bad: Free text -->
<Input name="state" placeholder="Enter state" />

<!-- Good: Radio for few options -->
<RadioGroup name="plan">
  <Radio value="basic">Basic ($9/mo)</Radio>
  <Radio value="pro">Pro ($29/mo)</Radio>
  <Radio value="enterprise">Enterprise (Custom)</Radio>
</RadioGroup>

<!-- Good: Checkbox for multi-select -->
<CheckboxGroup name="features">
  <Checkbox value="analytics">Analytics</Checkbox>
  <Checkbox value="api">API Access</Checkbox>
  <Checkbox value="support">Priority Support</Checkbox>
</CheckboxGroup>
```

**Related guidelines**: Error Avoidance (15), Make it Easy (2)

### Structured Input Fields

**Core principle**: Divide input fields to match expected format.

**Implementation**:
```html
<!-- Phone number: Structured -->
<PhoneInput>
  <Select name="countryCode">
    <option value="+1">+1</option>
    <option value="+44">+44</option>
  </Select>
  <Input name="phone" pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}"
         placeholder="555-123-4567" />
</PhoneInput>

<!-- Credit card: Structured -->
<CardInput>
  <Input name="number" maxlength="16" placeholder="1234 5678 9012 3456" />
  <Input name="expiry" placeholder="MM/YY" />
  <Input name="cvv" maxlength="3" placeholder="123" />
</CardInput>

<!-- Date: Use native picker -->
<Input type="date" name="birthdate" />
```

**Related guidelines**: Error Avoidance (15), User Input Flexibility (50)

---

## Buttons & Actions

### Specific Verbs for Buttons

**Core principle**: Use action-specific verbs, not generic "OK" or "Yes".

**Implementation**:
```html
<!-- Good: Specific -->
<Dialog>
  <p>Delete this project permanently?</p>
  <Button variant="danger">Delete Project</Button>
  <Button variant="secondary">Cancel</Button>
</Dialog>

<!-- Bad: Generic -->
<Dialog>
  <p>Delete this project permanently?</p>
  <Button>OK</Button>
  <Button>Cancel</Button>
</Dialog>

<!-- Good: Action-oriented -->
<Button>Save Changes</Button>
<Button>Create Account</Button>
<Button>Download Report</Button>
<Button>Send Invite</Button>

<!-- Bad: Generic -->
<Button>Submit</Button>
<Button>Continue</Button>
<Button>OK</Button>
```

**Related guidelines**: Use User Language (11), Clarity

### Button Hierarchy

**Core principle**: Visual weight matches action importance.

**Implementation**:
```html
<Actions>
  <!-- Primary: Most important action -->
  <Button variant="primary">Save Changes</Button>

  <!-- Secondary: Alternative action -->
  <Button variant="secondary">Save as Draft</Button>

  <!-- Tertiary: Less important -->
  <Button variant="tertiary">Preview</Button>

  <!-- Danger: Destructive action -->
  <Button variant="danger">Delete</Button>
</Actions>

<style>
.btn-primary {
  background: #0066cc;
  color: white;
  font-weight: 600;
}

.btn-secondary {
  background: white;
  color: #0066cc;
  border: 1px solid #0066cc;
}

.btn-tertiary {
  background: transparent;
  color: #6c757d;
}

.btn-danger {
  background: #dc3545;
  color: white;
}
</style>
```

**Related guidelines**: Visual Hierarchy, Consistency (6)

### Positive Language

**Core principle**: Frame choices positively, especially for toggles.

**Implementation**:
```html
<!-- Good: Positive framing -->
<Toggle label="Enable notifications" />
<Toggle label="Show email address" />
<Toggle label="Allow comments" />

<!-- Bad: Negative framing -->
<Toggle label="Disable notifications" />
<Toggle label="Hide email address" />
<Toggle label="Prevent comments" />

<!-- Good: Clear state -->
<Checkbox checked>
  Send me weekly updates
</Checkbox>

<!-- Bad: Double negative -->
<Checkbox>
  Don't not send me updates
</Checkbox>
```

**Related guidelines**: Use User Language (11), Clarity

---

## Data Input

### Input Suggestions (Autocomplete)

**Core principle**: Provide selectable options while typing.

**Implementation**:
```javascript
<Autocomplete
  options={cities}
  onSearch={(query) => filterCities(query)}
  renderOption={(city) => (
    <Option>
      <Icon name="location" />
      {city.name}, {city.state}
    </Option>
  )}
/>

// Common patterns:
// - Search (Google-style)
// - Address autocomplete
// - @ mentions
// - Tag selection
```

**Related guidelines**: Make it Easy (2), Error Avoidance (15)

### User Input Flexibility

**Core principle**: Accept various input formats automatically.

**Implementation**:
```javascript
// Phone number: Accept any format
const normalizePhone = (input) => {
  // Accepts: (555) 123-4567, 555-123-4567, 5551234567
  return input.replace(/\D/g, '');
};

// Date: Accept multiple formats
const parseDate = (input) => {
  // Accepts: 12/25/2024, 2024-12-25, Dec 25 2024
  return new Date(input);
};

// Email: Trim whitespace, lowercase
const normalizeEmail = (input) => {
  return input.trim().toLowerCase();
};
```

**Related guidelines**: Error Avoidance (15), Make it Easy (2)

### Select Results, Not Values

**Core principle**: Show outcomes, not parameter details.

**Implementation**:
```html
<!-- Good: Show result -->
<FontSizeSelector>
  <option value="12" style="font-size: 12px">Small text</option>
  <option value="16" style="font-size: 16px">Normal text</option>
  <option value="20" style="font-size: 20px">Large text</option>
</FontSizeSelector>

<!-- Bad: Show value -->
<Select>
  <option value="12">12px</option>
  <option value="16">16px</option>
  <option value="20">20px</option>
</Select>

<!-- Good: Visual color picker -->
<ColorPicker>
  <Swatch color="#FF0000" />
  <Swatch color="#00FF00" />
  <Swatch color="#0000FF" />
</ColorPicker>

<!-- Bad: Hex input -->
<Input placeholder="Enter hex color" />
```

**Related guidelines**: Communicate Information Not Data (28), Direct Manipulation (8)

---

## Error Handling

### Error Avoidance

**Core principle**: Prevent errors before they occur.

**Implementation**:
```javascript
// Disable invalid actions
<Button disabled={!formValid}>
  Submit
</Button>

// Hide irrelevant options
{user.role === 'admin' && (
  <MenuItem>Admin Panel</MenuItem>
)}

// Validate as user types
<Input
  type="email"
  onChange={(e) => {
    const valid = isValidEmail(e.target.value);
    setEmailError(valid ? null : 'Invalid email format');
  }}
  error={emailError}
/>
```

**Related guidelines**: Constraints (13), Error Messages

### Constructive Error Messages

**Core principle**: Explain what happened and how to fix it.

**Implementation**:
```javascript
// Good: Helpful
"Password must be at least 8 characters and include a number"

// Bad: Vague
"Invalid password"

// Good: Actionable
"Email address already in use. Try signing in or use password reset."

// Bad: Unclear
"Error 409"

// Good: Specific
"File size must be under 5MB. Your file is 8.2MB. Try compressing the image."

// Bad: Technical
"MAX_FILE_SIZE_EXCEEDED"
```

**Related guidelines**: Use User Language (11), Error Avoidance (15)

### Inline Validation

**Core principle**: Validate immediately, show errors near fields.

**Implementation**:
```html
<FormField>
  <Label>Email</Label>
  <Input
    type="email"
    value={email}
    onChange={handleEmailChange}
    onBlur={validateEmail}
    aria-invalid={emailError ? 'true' : 'false'}
    aria-describedby="email-error"
  />
  {emailError && (
    <ErrorMessage id="email-error">
      {emailError}
    </ErrorMessage>
  )}
</FormField>

<style>
.error-message {
  color: #dc3545;
  font-size: 14px;
  margin-top: 4px;
}

input[aria-invalid="true"] {
  border-color: #dc3545;
}
</style>
```

**Related guidelines**: Contextual Feedback (64), Responsive Interaction (63)

---

## Interaction Targets

### Touch Target Size

**Core principle**: Ensure tap targets are at least 44x44 pixels (7-10mm).

**Implementation**:
```css
/* Minimum touch target */
button, a, input[type="checkbox"] {
  min-height: 44px;
  min-width: 44px;
  padding: 12px;
}

/* Expand hit area without changing visual size */
.icon-button {
  position: relative;
  /* Visual size */
  width: 24px;
  height: 24px;
}

.icon-button::after {
  content: '';
  position: absolute;
  /* Expanded hit area */
  top: -10px;
  left: -10px;
  right: -10px;
  bottom: -10px;
}
```

**Related guidelines**: Fitts's Law (16), Touch Interface Design (76)

### Spacing for Accidental Touches

**Core principle**: Provide adequate spacing between interactive elements.

**Implementation**:
```css
/* Mobile: More spacing */
@media (max-width: 768px) {
  .button-group {
    gap: 16px; /* Prevent accidental touches */
  }

  .nav-item {
    padding: 16px;
    margin-bottom: 8px;
  }
}

/* Desktop: Less spacing needed */
@media (min-width: 769px) {
  .button-group {
    gap: 8px;
  }

  .nav-item {
    padding: 8px 16px;
  }
}
```

**Related guidelines**: Touch Interface Design (76), Platform-Specific

---

## Quick Reference

**Form best practices**:
- Group related fields
- Simple to complex order
- Provide good defaults
- Use constrained controls
- Structure complex inputs

**Button best practices**:
- Specific action verbs
- Clear visual hierarchy
- Positive language
- Consistent placement

**Input best practices**:
- Accept flexible formats
- Provide autocomplete
- Show results not values
- Validate inline

**Error handling**:
- Prevent when possible
- Constructive messages
- Show near context
- Make fixable

**Touch targets**:
- Minimum 44x44px
- Adequate spacing
- Expanded hit areas for small visuals
