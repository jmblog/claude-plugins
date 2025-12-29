# UI Design Analysis Checklist

This checklist provides a systematic framework for evaluating user interfaces against Human Interface Guidelines. Use it to conduct comprehensive UI audits.

## How to Use This Checklist

1. Review each category sequentially
2. Rate each item: ✓ (Good), ~ (Needs improvement), ✗ (Issue found)
3. Note specific examples for items marked ~ or ✗
4. Prioritize issues by impact and effort
5. Generate recommendations based on findings

---

## 1. Cognitive Principles

### Mental Model
- [ ] Does the interface match users' existing understanding of similar systems?
- [ ] Are metaphors familiar and appropriate?
- [ ] Is information organized around user tasks (not internal structure)?
- [ ] Can users predict what will happen before taking action?

### Signifiers
- [ ] Can users easily identify interactive elements?
- [ ] Do buttons look clickable?
- [ ] Are links visually distinct from plain text?
- [ ] Is element state (enabled/disabled/active) clear?
- [ ] Do visual cues indicate what actions are possible?

### Mapping
- [ ] Are controls positioned near the content they affect?
- [ ] Is the relationship between controls and effects obvious?
- [ ] Do spatial arrangements reflect logical relationships?
- [ ] Is feedback immediate when controls are manipulated?

### Consistency
- [ ] Do similar elements look and behave the same way?
- [ ] Is terminology consistent throughout?
- [ ] Are interaction patterns consistent across screens?
- [ ] Do colors, spacing, and typography follow consistent rules?
- [ ] Does the interface match platform conventions?

### User Control
- [ ] Can users initiate actions rather than having them forced?
- [ ] Are there clear cancel/undo options?
- [ ] Can users work at their own pace?
- [ ] Is media playback user-controlled (not auto-play)?
- [ ] Can users exit workflows easily?

### Direct Manipulation
- [ ] Can users interact directly with objects (drag, resize, etc.)?
- [ ] Is feedback immediate during manipulation?
- [ ] Are intermediate dialogs minimized?
- [ ] Can actions be reversed?

### Modeless
- [ ] Can users perform tasks in any order?
- [ ] Are modes minimized or eliminated?
- [ ] When modes exist, is current mode extremely obvious?
- [ ] Do common actions work consistently regardless of context?

### Visual Gestalt
- [ ] Are related elements grouped together (proximity)?
- [ ] Do similar elements look similar (similarity)?
- [ ] Is whitespace used effectively to show relationships?
- [ ] Is visual hierarchy clear?

---

## 2. Simplification

### Simplify
- [ ] Is every feature necessary for core user goals?
- [ ] Are advanced features hidden until needed?
- [ ] Could any elements be removed without losing value?
- [ ] Is the interface visually clean without excessive decoration?
- [ ] Are there fewer than 15 interactive elements per screen?

### Make it Easy
- [ ] Can common tasks be completed in minimal steps?
- [ ] Are multi-step processes really necessary?
- [ ] Is information pre-filled when possible?
- [ ] Are smart defaults provided?
- [ ] Are there shortcuts for frequent actions?
- [ ] Is inline editing used instead of modals where appropriate?

---

## 3. Information & Communication

### Use User Language
- [ ] Is terminology familiar to target users?
- [ ] Is technical jargon avoided or explained?
- [ ] Are labels clear and descriptive?
- [ ] Does copy match user vocabulary?

### Don't Rely on User Memory
- [ ] Is necessary information visible at point of need?
- [ ] Are tooltips/help available for complex features?
- [ ] Is context preserved across screens?
- [ ] Are multi-step processes shown with progress indicators?

### Communicate Information, Not Just Data
- [ ] Are metrics presented meaningfully (percentages not just numbers)?
- [ ] Are visualizations used where appropriate?
- [ ] Is data contextualized?
- [ ] Are units clear?

---

## 4. Visual Design

### Visual Hierarchy
- [ ] Is the most important content most prominent?
- [ ] Do headings clearly indicate content structure?
- [ ] Is size used to show importance?
- [ ] Does color direct attention appropriately?

### Tone & Manner Consistency
- [ ] Is visual style consistent (colors, gradients, shadows)?
- [ ] Do all UI elements match the design system?
- [ ] Are borders, radiuses, and spacing uniform?
- [ ] Is icon style consistent?

### Layout Consistency
- [ ] Is a grid system used?
- [ ] Are margins and padding consistent?
- [ ] Is alignment clean and intentional?
- [ ] Is whitespace used effectively?

### Visual Restraint
- [ ] Is color palette limited and purposeful?
- [ ] Are font variations minimal?
- [ ] Is animation subtle and purposeful?
- [ ] Is the interface free from visual clutter?

---

## 5. Navigation & Information Architecture

### Wayfinding
- [ ] Can users tell where they are in the application?
- [ ] Are breadcrumbs provided when appropriate?
- [ ] Is page title/heading clear?
- [ ] Is active section highlighted in navigation?

### Navigation Structure
- [ ] Is navigation easy to find?
- [ ] Are navigation labels noun-based (not verb-based)?
- [ ] Is hierarchy clear (primary vs secondary nav)?
- [ ] Can users navigate in multiple ways?

### Escape Hatch
- [ ] Is there always a way to return home/back?
- [ ] Can users exit any screen/workflow?
- [ ] Is close/cancel always available in modals?

---

## 6. Interaction Patterns

### Input Forms
- [ ] Are fields grouped logically?
- [ ] Does form flow from simple to complex?
- [ ] Are good defaults provided?
- [ ] Are labels positioned consistently (above field)?
- [ ] Are required fields marked clearly?
- [ ] Are constrained controls used (dropdowns vs free text)?
- [ ] Are field labels positive (not negative)?

### Buttons & Actions
- [ ] Do primary actions use primary button style?
- [ ] Are button labels specific verbs (not generic "OK")?
- [ ] Are destructive actions clearly distinguished?
- [ ] Is button placement consistent (primary on right)?

### User Inputs
- [ ] Are input suggestions provided?
- [ ] Is input flexible (accepts various formats)?
- [ ] Are structured inputs used for formatted data (phone, date)?
- [ ] Are all user inputs saved and editable?

---

## 7. Feedback & Responsiveness

### Responsive Interaction
- [ ] Does interface respond within 0.1 seconds to user input?
- [ ] Are loading states shown for operations >0.1s?
- [ ] Are progress indicators shown for long operations?
- [ ] Does the UI stay responsive during background operations?

### Contextual Feedback
- [ ] Is feedback shown near the point of interaction?
- [ ] Do form errors appear next to relevant fields?
- [ ] Are success messages clear and timely?
- [ ] Is system status visible when relevant?

### Animations & Transitions
- [ ] Are transitions smooth (0.1-0.5 seconds)?
- [ ] Do animations communicate state changes?
- [ ] Are transitions bi-directional?
- [ ] Are animations subtle and purposeful?

---

## 8. Error Handling

### Error Avoidance
- [ ] Are invalid actions disabled/hidden?
- [ ] Are constraints used to prevent errors?
- [ ] Are confusing options clearly distinguished?
- [ ] Is validation performed before submission when possible?

### Error Messages
- [ ] Are error messages constructive (explaining how to fix)?
- [ ] Do errors avoid technical jargon?
- [ ] Are errors shown immediately (not after submit)?
- [ ] Is error state visually clear?

### Fail-Safe Design
- [ ] Can users undo actions?
- [ ] Are destructive actions confirmed?
- [ ] Is work saved automatically?
- [ ] Can users recover from errors easily?

---

## 9. Mobile & Platform-Specific

### Touch Interface (if applicable)
- [ ] Are touch targets at least 44x44 pixels?
- [ ] Is spacing adequate to prevent accidental touches?
- [ ] Are gestures direct manipulation (not arbitrary)?
- [ ] Are swipe actions discoverable?

### Platform Conventions
- [ ] Do buttons follow platform placement (iOS vs Android)?
- [ ] Are platform-specific patterns used?
- [ ] Does navigation match platform expectations?
- [ ] Are platform gestures supported?

### Responsive Design
- [ ] Does layout adapt appropriately to screen size?
- [ ] Is content prioritized differently on mobile?
- [ ] Are touch and mouse interactions both supported?
- [ ] Is text readable without zooming?

---

## 10. Accessibility

### Screen Reader Support
- [ ] Are all interactive elements labeled?
- [ ] Is semantic HTML used?
- [ ] Are ARIA labels provided where needed?
- [ ] Is focus order logical?

### Visual Accessibility
- [ ] Does color contrast meet WCAG AA (4.5:1 for text)?
- [ ] Is information conveyed without relying solely on color?
- [ ] Can text size be increased?
- [ ] Are focus indicators visible?

### Keyboard Navigation
- [ ] Can all functions be accessed via keyboard?
- [ ] Is tab order logical?
- [ ] Are keyboard shortcuts provided for common actions?
- [ ] Can users see which element has focus?

---

## 11. Content & Messaging

### Progressive Disclosure
- [ ] Are advanced features hidden initially?
- [ ] Is information revealed gradually?
- [ ] Can users access more detail when needed?
- [ ] Are common tasks simple, rare tasks possible?

### Immediate Gratification
- [ ] Can users get value in first 10 seconds?
- [ ] Is sign-up delayed until necessary?
- [ ] Can users try before committing?
- [ ] Are quick wins obvious?

### Silent Execution
- [ ] Are only critical actions confirmed?
- [ ] Do state changes provide implicit feedback?
- [ ] Are confirmation dialogs minimized?

---

## 12. Advanced Considerations

### Performance
- [ ] Do pages load in under 3 seconds?
- [ ] Are images optimized?
- [ ] Is perceived performance good (skeleton screens, etc.)?

### Localization Ready (if applicable)
- [ ] Can UI accommodate longer text strings?
- [ ] Are icons culturally neutral?
- [ ] Is layout flexible for text expansion?
- [ ] Are date/number formats localizable?

### Customization
- [ ] Are defaults optimal for most users?
- [ ] Is customization available but not required?
- [ ] Are customizations preserved?

---

## Analysis Summary Template

After completing checklist:

### Strengths
List 3-5 things the interface does well

### Critical Issues (High Priority)
Issues that significantly impact usability:
1. [Issue] - Violates [Guideline] - Impact: [Description]
2. ...

### Moderate Issues (Medium Priority)
Issues that should be addressed:
1. [Issue] - Violates [Guideline] - Impact: [Description]
2. ...

### Minor Issues (Low Priority)
Polish and refinement opportunities:
1. [Issue] - Violates [Guideline] - Impact: [Description]
2. ...

### Recommendations
Prioritized list of improvements:
1. **[High Priority]** [Recommendation] - Expected impact: [Description]
2. **[Medium Priority]** [Recommendation] - Expected impact: [Description]
3. **[Low Priority]** [Recommendation] - Expected impact: [Description]

---

## Quick Evaluation (5-Minute Version)

For rapid assessment, check these critical areas:

**Cognitive Basics**:
- [ ] Can users tell what's interactive? (Signifiers)
- [ ] Does it work like users expect? (Mental Model)
- [ ] Is it consistent? (Consistency)

**Simplification**:
- [ ] Is it simple enough? (Simplify)
- [ ] Is it easy enough? (Make it Easy)

**Critical UX**:
- [ ] Can users complete main task easily?
- [ ] Are errors handled gracefully?
- [ ] Is feedback immediate and clear?

**Accessibility**:
- [ ] Can it be used with keyboard only?
- [ ] Is color contrast sufficient?
- [ ] Are interactive elements large enough?

If any critical items fail, prioritize those issues.
