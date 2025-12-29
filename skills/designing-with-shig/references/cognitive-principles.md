# Cognitive Principles

These fundamental principles form the foundation of intuitive interface design. Understanding how users think and perceive enables creation of interfaces that feel natural and require minimal learning.

## Contents
- [Mental Model](#mental-model)
- [Signifiers](#signifiers)
- [Mapping](#mapping)
- [Consistency](#consistency)
- [User Control](#user-control)
- [Direct Manipulation](#direct-manipulation)
- [Modeless](#modeless)
- [Visual Gestalt](#visual-gestalt)

---

## Mental Model

**Core principle**: Design structures and movements that match users' imagined usage model.

**Apply when**:
- Designing new features or interfaces
- Choosing metaphors for abstract concepts
- Organizing information architecture

**Implementation**:
- Research user expectations and prior experiences
- Use familiar real-world metaphors (folders for file organization, shopping carts for purchases)
- Design interactions that follow users' existing understanding
- Test with users to validate mental model alignment

**Examples**:

Good example: File system using folder metaphor
```
Desktop/
  Documents/
    Work/
    Personal/
```
Users understand physical folders, so digital folders feel natural.

Good example: E-commerce shopping cart
- Add items → Cart fills
- Remove items → Cart empties
- Checkout → Go to register
Matches physical shopping experience.

**Anti-patterns**:
- Using unfamiliar metaphors that require explanation
- Breaking established conventions (e.g., up arrow meaning "delete")
- Organizing content by internal system logic rather than user tasks

**Related guidelines**: Signifiers (4), Use User Language (11), Conventional over Intuitive (69)

---

## Signifiers

**Core principle**: Make interactive elements visible and self-explanatory through visual cues.

**Apply when**:
- Designing buttons, links, and interactive controls
- Indicating element state (enabled, disabled, active)
- Showing possible actions

**Implementation**:
- Use familiar shapes: rounded rectangles for buttons, underlines for links
- Apply appropriate visual affordances:
  - Buttons appear raised or have borders
  - Links use distinct color and/or underline
  - Draggable elements show cursor changes
- Show state through visual feedback:
  - Hover states
  - Active/pressed states
  - Disabled states (grayed out)

**Examples**:

Good example: Clear button signifiers
```html
<!-- Primary action button -->
<button style="background: #0066cc; color: white; padding: 12px 24px; border-radius: 4px">
  Save Changes
</button>

<!-- Link with underline -->
<a href="/help" style="color: #0066cc; text-decoration: underline">
  Learn more
</a>
```

Good example: Icon buttons with labels
```html
<button>
  <Icon name="trash" />
  <span>Delete</span>
</button>
```
Icon + label removes ambiguity.

**Anti-patterns**:
- Flat design that makes clickable elements indistinguishable from static content
- Inconsistent signifiers (some buttons look different from others)
- Relying solely on color to indicate interactivity (fails accessibility)
- Using non-standard signifiers that users must learn

**Related guidelines**: Mental Model (3), Mapping (5), Consistency (6), Accessibility (92)

---

## Mapping

**Core principle**: Clarify relationships between controls and their effects using spatial, visual, or logical connections.

**Apply when**:
- Positioning controls relative to content they affect
- Designing forms and data entry interfaces
- Creating control panels or settings

**Implementation**:
- Position controls near affected elements
- Use spatial arrangement to show relationships:
  - Horizontal slider for timeline/continuous values
  - Vertical slider for volume/height
  - Grid layout for spatial relationships
- Use visual cues (lines, grouping, color) to connect controls to effects
- Provide immediate feedback when controls are manipulated

**Examples**:

Good example: Text formatting toolbar
```
[B] [I] [U] | [Left] [Center] [Right] | [Color picker]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Selected text appears here with formatting applied
```
Toolbar is directly above affected text.

Good example: Volume slider
```
🔇 ━━━━━━●━━━━ 🔊
   0        50   100
```
Horizontal mapping: left = quiet, right = loud.

**Anti-patterns**:
- Controls far from affected content
- Arbitrary control placement with no logical relationship
- Inconsistent mapping (sometimes left means more, sometimes less)
- Grouping unrelated controls together

**Related guidelines**: Mental Model (3), Signifiers (4), Position Tools Near Work Context (30)

---

## Consistency

**Core principle**: Apply consistent rules in color, shape, layout, and behavior across the interface.

**Apply when**:
- Designing multiple screens or pages
- Creating component libraries
- Establishing design systems
- Building multi-feature applications

**Implementation**:
- Use consistent visual elements:
  - Same button styles for same action types
  - Consistent color scheme
  - Uniform spacing and typography
- Apply consistent behavior patterns:
  - Same interactions produce same results
  - Navigation works the same way throughout
  - Confirmation dialogs follow same pattern
- Create and follow design system guidelines
- Use component libraries to ensure consistency

**Examples**:

Good example: Consistent button hierarchy
```javascript
// Primary actions
<Button variant="primary">Save</Button>
<Button variant="primary">Submit</Button>

// Secondary actions
<Button variant="secondary">Cancel</Button>
<Button variant="secondary">Back</Button>

// Danger actions
<Button variant="danger">Delete</Button>
<Button variant="danger">Remove</Button>
```

Good example: Consistent form patterns
```
All forms follow same pattern:
- Label above input
- Required fields marked with *
- Error messages below field in red
- Submit button bottom-right
```

**Anti-patterns**:
- Different button styles for same actions across pages
- Inconsistent terminology ("Delete" on one page, "Remove" on another)
- Navigation in different locations on different screens
- Mixing interaction patterns (sometimes click, sometimes double-click for same action)

**Related guidelines**: Graphical Tone & Manner Consistency (27), Layout Consistency (83), Platform-Specific Button Placement (70)

---

## User Control

**Core principle**: Let users control the system. Minimize forced operations and allow users to choose based on their intentions.

**Apply when**:
- Designing workflows and task flows
- Implementing automation features
- Creating onboarding experiences
- Designing modal dialogs

**Implementation**:
- Provide explicit user actions rather than automatic behaviors
- Allow users to cancel or undo operations
- Let users choose timing and sequence of actions
- Avoid auto-playing media, auto-advancing slides, or forced tutorials
- Provide clear exit options from all states

**Examples**:

Good example: Video player controls
```
[Play/Pause] [Volume] [Progress bar with scrubbing]
               [Captions] [Speed] [Fullscreen]
```
User controls all aspects: when to play, volume, position, settings.

Good example: Optional onboarding
```
Welcome! Would you like a quick tour?
[Take tour]  [Skip - I'll explore on my own]
```
User chooses whether to engage.

**Anti-patterns**:
- Auto-playing videos with hidden/tiny controls
- Forced tutorials that prevent accessing actual application
- Workflows that must be completed in exact sequence
- Automatic actions users can't prevent or undo
- Modal dialogs without clear close/cancel options

**Related guidelines**: Direct Manipulation (8), Modeless (9), Escape Hatch (59), Pace of User Work (93)

---

## Direct Manipulation

**Core principle**: Create sensation of directly touching/manipulating on-screen objects with real-time feedback.

**Apply when**:
- Designing drag-and-drop interfaces
- Creating visual editors (image, layout, diagram)
- Implementing gestures and touch interactions
- Building interactive visualizations

**Implementation**:
- Make objects directly manipulable (drag, resize, rotate)
- Provide immediate visual feedback during manipulation
- Show object state changes in real-time
- Allow reversal of actions
- Minimize intermediate dialogs or modes

**Examples**:

Good example: Drag-and-drop file upload
```javascript
// User can drag files directly onto target
<DropZone>
  Drag files here or click to browse
</DropZone>

// Files appear immediately as they're added
<FileList>
  {files.map(file => (
    <File name={file.name} onRemove={() => removeFile(file)} />
  ))}
</FileList>
```

Good example: Image crop tool
```
User drags corners of crop box
→ Image preview updates in real-time
→ Dimensions shown live
→ Can adjust until satisfied
→ Click "Crop" to apply
```

**Anti-patterns**:
- Requiring dialogs to specify dimensions instead of direct resizing
- No visual feedback during manipulation
- Delayed updates that break feeling of directness
- Indirect controls (text inputs for coordinates instead of drag handles)

**Related guidelines**: User Control (7), Responsive Interaction (63), Object State Visualization (25)

---

## Modeless

**Core principle**: Minimize modes where operations have different meanings depending on current state. Allow free-order task completion.

**Apply when**:
- Designing tool interfaces
- Creating multi-function applications
- Implementing editing interfaces
- Designing form workflows

**Implementation**:
- Avoid "modes" where same action has different effects
- If modes are necessary, make current mode extremely obvious
- Allow actions in any order when possible
- Don't lock users into specific workflows
- Provide clear indication when modes must exist

**Examples**:

Good example: Modeless editing
```
Text editor where:
- Selection and editing always available
- Formatting tools always accessible
- No "edit mode" vs "view mode"
- All actions available all the time
```

Good example: Modern graphics applications
```
Tools palette always visible
Can switch tools anytime
Previous tool settings preserved
No mode prevents accessing other features
```

**Anti-patterns**:
- "Edit mode" that must be explicitly entered
- Modes that change meaning of common actions (Enter key)
- Locked workflows requiring specific sequence
- Hidden modes that confuse users ("Why isn't this working?")
- Mode-dependent keyboard shortcuts

**Acceptable exception with clear signifier**:
```
Text field in "editing" vs "viewing" state
Clear visual distinction:
- Editing: white background, cursor visible, border highlighted
- Viewing: gray background, no cursor, plain border
```

**Related guidelines**: User Control (7), Flexibility (86), Self-Explanatory Design (87)

---

## Visual Gestalt

**Core principle**: Utilize perception's tendency to recognize whole patterns. Use proximity, similarity, and closure in layout to show relationships.

**Apply when**:
- Organizing form fields and controls
- Designing page layouts
- Creating navigation structures
- Grouping related information

**Implementation**:

**Proximity**: Elements close together are perceived as related
```css
/* Good: Related fields grouped with spacing */
.form-section {
  margin-bottom: 32px; /* Large gap between sections */
}

.field {
  margin-bottom: 8px; /* Small gap within section */
}
```

**Similarity**: Elements that look similar are perceived as related
```css
/* All primary actions use same button style */
.btn-primary { background: blue; }

/* All secondary actions use different style */
.btn-secondary { background: gray; }
```

**Closure**: People fill in gaps to create complete shapes
```
Navigation breadcrumb:
Home > Products > Electronics > Phones
(Users perceive complete hierarchy even with minimal visual elements)
```

**Common fate**: Elements moving together are perceived as related
```javascript
// When accordion opens, all section content moves together
// indicating they belong to same group
```

**Examples**:

Good example: Form layout using proximity
```html
<form>
  <!-- Personal info section (tight spacing) -->
  <div class="section">
    <h3>Personal Information</h3>
    <input name="firstName" />
    <input name="lastName" />
    <input name="email" />
  </div>

  <!-- Large gap -->

  <!-- Address section (tight spacing) -->
  <div class="section">
    <h3>Address</h3>
    <input name="street" />
    <input name="city" />
    <input name="zip" />
  </div>
</form>
```

Good example: Card design using similarity and proximity
```css
.card {
  /* All cards look similar */
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;

  /* Content inside card close together */
}

.card-title {
  margin-bottom: 4px;
}

.card-description {
  margin-bottom: 0;
}
```

**Anti-patterns**:
- Equal spacing between all elements (no grouping indicated)
- Mixing unrelated elements with similar styling
- Related elements that look different
- Inconsistent spacing that breaks perceived relationships

**Related guidelines**: Layout Consistency (83), Visual Restraint (82), Graphical Tone & Manner Consistency (27)

---

## Applying Cognitive Principles Together

These principles work in concert to create intuitive interfaces:

1. **Start with Mental Model**: Understand how users think about the domain
2. **Add Signifiers**: Make interactive elements obvious
3. **Use Mapping**: Position controls logically relative to effects
4. **Apply Consistency**: Use same patterns throughout
5. **Enable User Control**: Let users drive interactions
6. **Implement Direct Manipulation**: Remove intermediary steps where possible
7. **Minimize Modes**: Allow actions in any context
8. **Use Visual Gestalt**: Group and organize through visual design

**Example applying all principles**:

```javascript
// Email client interface demonstrating all principles

// Mental Model: Metaphor of physical mail
<Layout>
  {/* Visual Gestalt: Proximity groups related elements */}
  <Sidebar>
    {/* Consistency: All folders use same pattern */}
    <Folder icon="inbox">Inbox (5)</Folder>
    <Folder icon="sent">Sent</Folder>
    <Folder icon="trash">Trash</Folder>
  </Sidebar>

  <MessageList>
    {messages.map(msg => (
      {/* Signifiers: Unread messages visually distinct */}
      <Message
        isUnread={msg.unread}
        {/* Direct Manipulation: Drag to folder */}
        draggable
        onDragEnd={(folder) => moveToFolder(msg, folder)}
        {/* User Control: Click to read when ready */}
        onClick={() => openMessage(msg)}
      >
        {msg.subject}
      </Message>
    ))}
  </MessageList>

  {/* Modeless: All actions available anytime */}
  <Toolbar>
    {/* Mapping: Delete button near selected message */}
    <Button>Reply</Button>
    <Button>Forward</Button>
    <Button>Delete</Button>
  </Toolbar>
</Layout>
```

---

## Quick Reference

When designing interfaces, ask:

1. **Mental Model**: Does this match how users think about the task?
2. **Signifiers**: Can users tell what's interactive and what it does?
3. **Mapping**: Is the relationship between control and effect clear?
4. **Consistency**: Does this work like similar elements elsewhere?
5. **User Control**: Can users decide when and how to act?
6. **Direct Manipulation**: Can users interact with objects directly?
7. **Modeless**: Do actions work the same way regardless of context?
8. **Visual Gestalt**: Does visual grouping show relationships?
