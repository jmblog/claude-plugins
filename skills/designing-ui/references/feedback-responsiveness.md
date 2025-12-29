# Feedback & Responsiveness

Feedback and responsiveness principles ensure users understand system state and receive timely responses to actions.

## Contents
- [Responsive Interaction](#responsive-interaction)
- [Contextual Feedback](#contextual-feedback)
- [Transition Animations](#transition-animations)
- [Loading States](#loading-states)
- [Silent Execution](#silent-execution)

---

## Responsive Interaction

**Core principle**: Respond to user actions within 0.1 seconds. Show progress for longer operations.

**Apply when**:
- Handling user input
- Processing operations
- Loading data
- Performing async tasks

**Implementation**:

Immediate feedback (<0.1s):
```javascript
// Button press feedback
<Button
  onMouseDown={() => setPressed(true)}
  onMouseUp={() => setPressed(false)}
  className={pressed ? 'pressed' : ''}
>
  Click me
</Button>

<style>
.button.pressed {
  transform: scale(0.98);
  transition: transform 0.05s;
}
</style>
```

Short operations (0.1-1s):
```javascript
// Show spinner for operations >0.1s
const [loading, setLoading] = useState(false);

const handleSubmit = async () => {
  setLoading(true);
  try {
    await saveData();
  } finally {
    setLoading(false);
  }
};

<Button disabled={loading}>
  {loading ? <Spinner /> : 'Save'}
</Button>
```

Long operations (>1s):
```javascript
// Show progress for long operations
const [progress, setProgress] = useState(0);

const uploadFile = async (file) => {
  const xhr = new XMLHttpRequest();
  xhr.upload.addEventListener('progress', (e) => {
    setProgress((e.loaded / e.total) * 100);
  });
  // ... upload logic
};

<ProgressBar value={progress} max={100} />
```

**Examples**:

Good example: Immediate visual feedback
```javascript
<Input
  onChange={(e) => {
    // Update immediately as user types
    setValue(e.target.value);
  }}
  value={value} // Shows change instantly
/>
```

**Anti-patterns**:
- No feedback during async operations
- Delayed response to clicks/taps
- Frozen UI during processing
- No progress indication for long tasks

**Related guidelines**: Contextual Feedback (64), UI Responsiveness (88)

---

## Contextual Feedback

**Core principle**: Provide feedback near the point of interaction.

**Apply when**:
- Showing validation errors
- Displaying success messages
- Indicating loading states
- Providing tooltips or hints

**Implementation**:

Inline validation:
```html
<FormField>
  <Label>Email</Label>
  <Input
    type="email"
    value={email}
    onChange={handleChange}
  />
  <!-- Error appears next to field -->
  {error && (
    <ErrorMessage>
      <Icon name="alert" />
      {error}
    </ErrorMessage>
  )}
</FormField>
```

Contextual success:
```javascript
// Show success near action
<Button onClick={handleSave}>
  Save
</Button>
{saved && (
  <SuccessMessage>
    <Icon name="check" />
    Saved successfully
  </SuccessMessage>
)}
```

Toast notifications:
```javascript
// For actions affecting whole page
const showToast = (message) => {
  toast.success(message, {
    position: 'top-right', // Non-blocking position
    autoClose: 3000
  });
};

<Button onClick={() => {
  deleteItem();
  showToast('Item deleted');
}}>
  Delete
</Button>
```

**Examples**:

Good example: Loading state near action
```javascript
<Card>
  <h3>{post.title}</h3>
  <Button onClick={likePost} disabled={liking}>
    {liking ? (
      <>
        <Spinner size="sm" />
        Liking...
      </>
    ) : (
      <>
        <Icon name="heart" />
        Like
      </>
    )}
  </Button>
</Card>
```

**Anti-patterns**:
- Global error messages for field-specific errors
- Success messages far from action
- Modal dialogs for non-critical feedback
- No visual indication during loading

**Related guidelines**: Responsive Interaction (63), Constructive Error Messaging (55)

---

## Transition Animations

**Core principle**: Use 0.1-0.5 second animations for state changes. Animate both forward and reverse.

**Apply when**:
- Showing/hiding elements
- Changing between states
- Moving elements
- Indicating progress

**Implementation**:

Standard transitions:
```css
/* Default: 200ms for most transitions */
.element {
  transition: all 0.2s ease-in-out;
}

/* Quick: 100ms for immediate feedback */
.button {
  transition: transform 0.1s;
}

.button:active {
  transform: scale(0.98);
}

/* Moderate: 300ms for complex changes */
.modal {
  transition: opacity 0.3s, transform 0.3s;
}

.modal.entering {
  opacity: 1;
  transform: scale(1);
}

.modal.exiting {
  opacity: 0;
  transform: scale(0.95);
}
```

Easing functions:
```css
/* Use appropriate easing */
.slide-in {
  transition: transform 0.3s ease-out; /* Decelerates */
}

.slide-out {
  transition: transform 0.3s ease-in; /* Accelerates */
}

.bounce {
  transition: transform 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
}
```

**Examples**:

Good example: Smooth expand/collapse
```javascript
<Accordion>
  <AccordionItem>
    <AccordionHeader onClick={toggle}>
      Section Title
      <Icon name="chevron" className={open ? 'rotate-180' : ''} />
    </AccordionHeader>
    <AccordionContent
      style={{
        maxHeight: open ? contentHeight : 0,
        transition: 'max-height 0.3s ease-in-out'
      }}
    >
      Content here
    </AccordionContent>
  </AccordionItem>
</Accordion>
```

**Anti-patterns**:
- No animation (jarring state changes)
- Too slow (>0.5s feels sluggish)
- Too fast (<0.1s feels abrupt)
- Inconsistent animation speeds
- One-way animations (no reverse)

**Related guidelines**: Responsive Interaction (63), Visual Hierarchy

---

## Loading States

**Core principle**: Show appropriate loading indicators based on operation duration.

**Apply when**:
- Fetching data
- Submitting forms
- Loading pages
- Processing operations

**Implementation**:

Component-level loading:
```javascript
// Skeleton screens for initial load
<Card>
  {loading ? (
    <Skeleton>
      <SkeletonText lines={3} />
      <SkeletonImage />
    </Skeleton>
  ) : (
    <Content data={data} />
  )}
</Card>
```

Page-level loading:
```javascript
// Loading overlay for full page
{loading && (
  <LoadingOverlay>
    <Spinner size="lg" />
    <p>Loading...</p>
  </LoadingOverlay>
)}
```

Progressive loading:
```javascript
// Load content as it becomes available
<Feed>
  {posts.map(post => (
    <Post key={post.id} data={post} />
  ))}
  {hasMore && (
    <LoadMoreButton onClick={loadMore} loading={loadingMore}>
      {loadingMore ? 'Loading...' : 'Load More'}
    </LoadMoreButton>
  )}
</Feed>
```

**Examples**:

Good example: Optimistic UI
```javascript
const likePost = async (postId) => {
  // Optimistically update UI
  setPosts(posts.map(p =>
    p.id === postId ? { ...p, liked: true, likes: p.likes + 1 } : p
  ));

  try {
    await api.likePost(postId);
  } catch (error) {
    // Revert on error
    setPosts(posts.map(p =>
      p.id === postId ? { ...p, liked: false, likes: p.likes - 1 } : p
    ));
    showError('Failed to like post');
  }
};
```

**Anti-patterns**:
- Blank screen during load
- Unclear that loading is happening
- No indication of progress
- Blocking entire UI for partial updates

**Related guidelines**: Responsive Interaction (63), Progressive Disclosure (65)

---

## Silent Execution

**Core principle**: Provide feedback through state changes. Confirm only critical, irreversible actions.

**Apply when**:
- Performing reversible actions
- Auto-saving content
- Updating settings
- Non-destructive operations

**Implementation**:

Auto-save without confirmation:
```javascript
// Silently save as user types
const [content, setContent] = useState('');
const [saving, setSaving] = useState(false);

useDebounce(() => {
  if (content) {
    setSaving(true);
    saveContent(content).then(() => {
      setSaving(false);
    });
  }
}, 1000, [content]);

<Editor
  value={content}
  onChange={setContent}
/>
{saving && <SaveIndicator>Saving...</SaveIndicator>}
```

State change as confirmation:
```javascript
// Toggle state shows action completed
<Toggle
  checked={notifications}
  onChange={(checked) => {
    setNotifications(checked);
    updateSettings({ notifications: checked });
    // No "Settings saved" modal needed
  }}
>
  Enable notifications
</Toggle>
```

Confirm only destructive actions:
```javascript
// Reversible: No confirmation
<Button onClick={() => archiveItem(id)}>
  Archive
</Button>
<Toast>Item archived. <a onClick={undo}>Undo</a></Toast>

// Irreversible: Confirm
<Button onClick={() => {
  if (confirm('Delete this item permanently?')) {
    deleteItem(id);
  }
}}>
  Delete
</Button>
```

**Examples**:

Good example: Implicit feedback
```javascript
// Star button shows state through appearance
<Button
  onClick={toggleStar}
  className={starred ? 'starred' : ''}
>
  <Icon name={starred ? 'star-filled' : 'star-outline'} />
</Button>
// No "Added to favorites" confirmation needed
```

**Anti-patterns**:
- Confirmation for every action
- "Are you sure?" for reversible operations
- Success modals blocking workflow
- No feedback at all

**Related guidelines**: Fail-Safe over Fool-Proof (54), User Control (7)

---

## Feedback Patterns

### Success Feedback

```javascript
// Minimal: State change only
<Button onClick={save} className={saved ? 'success' : ''}>
  {saved ? 'Saved ✓' : 'Save'}
</Button>

// Moderate: Inline message
{saved && (
  <SuccessMessage>
    Changes saved successfully
  </SuccessMessage>
)}

// Prominent: Toast for important actions
toast.success('Project published successfully');
```

### Error Feedback

```javascript
// Field-level: Inline
<Input error="Email is required" />

// Form-level: Summary
<ErrorSummary>
  Please fix the following errors:
  <ul>
    <li>Email is required</li>
    <li>Password must be at least 8 characters</li>
  </ul>
</ErrorSummary>

// System-level: Toast or banner
<ErrorBanner>
  Unable to save. Please check your connection.
</ErrorBanner>
```

### Progress Feedback

```javascript
// Determinate: Known duration
<ProgressBar value={progress} max={100} />
<p>{progress}% complete</p>

// Indeterminate: Unknown duration
<Spinner />
<p>Processing...</p>

// Step indicator: Multi-step process
<Steps current={2}>
  <Step>Details</Step>
  <Step>Review</Step>
  <Step>Confirm</Step>
</Steps>
```

**Related guidelines**: Contextual Feedback (64), Constructive Error Messaging (55)
