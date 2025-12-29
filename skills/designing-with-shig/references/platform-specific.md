# Platform-Specific Design

Platform-specific principles guide design decisions for different devices, platforms, and interaction methods.

## Contents
- [Mobile vs Desktop](#mobile-vs-desktop)
- [Touch Interface Design](#touch-interface-design)
- [Gesture Design](#gesture-design)
- [Platform Conventions](#platform-conventions)
- [Responsive Design](#responsive-design)

---

## Mobile vs Desktop

**Core principle**: Desktop provides comprehensive overview; mobile uses hierarchical, focused presentation.

**Apply when**:
- Designing responsive layouts
- Planning information architecture
- Prioritizing features
- Creating navigation systems

**Implementation**:

Content prioritization:
```javascript
// Desktop: Show more simultaneously
<DesktopLayout>
  <Sidebar>
    <Navigation />
    <Filters />
    <RecentItems />
  </Sidebar>
  <Main>
    <Content />
  </Main>
  <Aside>
    <RelatedItems />
    <Suggestions />
  </Aside>
</DesktopLayout>

// Mobile: Show sequentially
<MobileLayout>
  <Header>
    <MenuButton />
    <Title />
  </Header>
  <Main>
    <Content /> {/* Primary focus */}
  </Main>
  <BottomNav>
    <NavItem>Home</NavItem>
    <NavItem>Search</NavItem>
    <NavItem>Profile</NavItem>
  </BottomNav>
</MobileLayout>
```

Interaction patterns:
```css
/* Desktop: Hover states */
@media (hover: hover) {
  .button:hover {
    background: #0052a3;
  }

  .link:hover {
    text-decoration: underline;
  }
}

/* Mobile: No hover, larger touch targets */
@media (hover: none) {
  .button {
    min-height: 44px;
    min-width: 44px;
  }

  .button:active {
    background: #0052a3;
  }
}
```

**Examples**:

Good example: Adaptive navigation
```javascript
// Desktop: Horizontal menu
<nav className="desktop-nav">
  <NavItem>Dashboard</NavItem>
  <NavItem>Projects</NavItem>
  <NavItem>Team</NavItem>
  <NavItem>Settings</NavItem>
</nav>

// Mobile: Hamburger menu
<nav className="mobile-nav">
  <MenuButton onClick={toggleMenu} />
  <Drawer open={menuOpen}>
    <NavItem>Dashboard</NavItem>
    <NavItem>Projects</NavItem>
    <NavItem>Team</NavItem>
    <NavItem>Settings</NavItem>
  </Drawer>
</nav>
```

**Anti-patterns**:
- Identical layouts for mobile and desktop
- Small touch targets on mobile
- Hidden functionality on mobile without good reason
- Desktop-only hover interactions without mobile alternative

**Related guidelines**: Progressive Disclosure (65), Navigation Hierarchy (78)

---

## Touch Interface Design

**Core principle**: Touch targets should be 44x44 pixels minimum (7-10mm). Space elements to prevent accidental touches.

**Apply when**:
- Designing mobile interfaces
- Creating tablet layouts
- Building touch-optimized controls
- Planning interactive elements

**Implementation**:

Touch target sizing:
```css
/* Minimum touch target */
button, a, input[type="checkbox"], input[type="radio"] {
  min-height: 44px;
  min-width: 44px;
  padding: 12px;
}

/* Comfortable size */
.primary-button {
  height: 48px;
  padding: 14px 24px;
}

/* Extra spacing for adjacent targets */
.button-group {
  display: flex;
  gap: 16px; /* Prevent fat finger errors */
}
```

Touch states:
```css
/* Show active state on touch */
.button:active {
  background: #0052a3;
  transform: scale(0.98);
}

/* Long press indication */
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

.button.long-pressing {
  animation: pulse 1s infinite;
}
```

**Examples**:

Good example: Touch-friendly list
```html
<List>
  <ListItem>
    <Checkbox size={44} /> <!-- Large enough to tap -->
    <Label>Task name</Label>
    <IconButton size={44} aria-label="More options">
      <Icon name="more-vertical" />
    </IconButton>
  </ListItem>
</List>

<style>
.list-item {
  min-height: 56px;
  padding: 8px 16px;
  display: flex;
  align-items: center;
  gap: 16px;
}
</style>
```

**Anti-patterns**:
- Buttons smaller than 44x44px
- Adjacent buttons with no spacing
- Relying on hover states
- Small text links in paragraphs

**Related guidelines**: Fitts's Law (16), Interaction Targets (91)

---

## Gesture Design

**Core principle**: Use direct manipulation gestures. Avoid arbitrary command-based gestures.

**Apply when**:
- Designing mobile interactions
- Creating touch-optimized features
- Implementing swipe actions
- Building gesture controls

**Implementation**:

Natural gestures:
```javascript
// Swipe to delete (matches physical action)
<SwipeableListItem
  onSwipeLeft={() => deleteItem()}
  leftAction={<DeleteAction />}
>
  <ItemContent />
</SwipeableListItem>

// Pull to refresh (natural pulling motion)
<PullToRefresh onRefresh={refreshData}>
  <Content />
</PullToRefresh>

// Pinch to zoom (direct manipulation)
<Zoomable
  minScale={1}
  maxScale={4}
  onZoom={(scale) => setZoom(scale)}
>
  <Image src={photo} />
</Zoomable>
```

Discoverable gestures:
```javascript
// Show hints for non-obvious gestures
const [showHint, setShowHint] = useState(true);

<Card>
  {showHint && (
    <GestureHint>
      Swipe left to delete
      <Button onClick={() => setShowHint(false)}>Got it</Button>
    </GestureHint>
  )}
  <SwipeableContent />
</Card>
```

**Examples**:

Good example: Swipe actions
```javascript
<SwipeableRow
  leftActions={[
    { icon: 'archive', color: 'blue', onPress: archive },
    { icon: 'delete', color: 'red', onPress: delete }
  ]}
  rightActions={[
    { icon: 'star', color: 'yellow', onPress: star }
  ]}
>
  <EmailPreview />
</SwipeableRow>
```

**Anti-patterns**:
- Complex multi-finger gestures
- Hidden gestures without hints
- Arbitrary gestures (e.g., draw a circle to...)
- Conflicting gestures

**Related guidelines**: Direct Manipulation (8), Signifiers (4)

---

## Platform Conventions

**Core principle**: Follow platform-specific patterns and conventions.

**Apply when**:
- Building for specific platforms (iOS, Android, Web)
- Implementing navigation
- Positioning buttons
- Creating dialogs

**Implementation**:

iOS conventions:
```javascript
// iOS: Back button top-left, action top-right
<IOSNavBar>
  <BackButton>< Back</BackButton>
  <Title>Screen Title</Title>
  <ActionButton>Edit</ActionButton>
</IOSNavBar>

// iOS: Bottom tab bar
<IOSTabBar>
  <Tab icon="home">Home</Tab>
  <Tab icon="search">Search</Tab>
  <Tab icon="profile">Profile</Tab>
</IOSTabBar>

// iOS: Action sheet from bottom
<ActionSheet>
  <Action>Edit</Action>
  <Action destructive>Delete</Action>
  <Action cancel>Cancel</Action>
</ActionSheet>
```

Android conventions:
```javascript
// Android: Floating action button (FAB)
<FAB onClick={createNew}>
  <Icon name="plus" />
</FAB>

// Android: Navigation drawer from left
<NavigationDrawer>
  <DrawerItem icon="home">Home</DrawerItem>
  <DrawerItem icon="settings">Settings</DrawerItem>
</NavigationDrawer>

// Android: Snackbar for feedback
<Snackbar
  message="Item deleted"
  action="Undo"
  onAction={undoDelete}
/>
```

Button placement:
```css
/* iOS: Positive action on right */
.ios-dialog-actions {
  display: flex;
  justify-content: flex-end;
}

.ios-dialog-actions .cancel {
  order: 1;
}

.ios-dialog-actions .confirm {
  order: 2;
}

/* Android: Positive action on right */
.android-dialog-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
}
```

**Examples**:

Good example: Platform-adaptive dialog
```javascript
const Dialog = ({ platform }) => (
  <DialogContainer>
    <DialogTitle>Delete item?</DialogTitle>
    <DialogContent>This cannot be undone.</DialogContent>

    {platform === 'ios' ? (
      <IOSActions>
        <Button>Cancel</Button>
        <Button color="red">Delete</Button>
      </IOSActions>
    ) : (
      <AndroidActions>
        <Button>CANCEL</Button>
        <Button color="red">DELETE</Button>
      </AndroidActions>
    )}
  </DialogContainer>
);
```

**Related guidelines**: Consistency (6), Conventional over Intuitive (69)

---

## Responsive Design

**Core principle**: Adapt layout and content to screen size while maintaining functionality.

**Apply when**:
- Designing for multiple devices
- Creating flexible layouts
- Planning breakpoints
- Optimizing content display

**Implementation**:

Breakpoint strategy:
```css
/* Mobile first approach */
.container {
  padding: 16px;
}

/* Tablet: 768px+ */
@media (min-width: 768px) {
  .container {
    padding: 24px;
    max-width: 720px;
    margin: 0 auto;
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop: 1024px+ */
@media (min-width: 1024px) {
  .container {
    max-width: 960px;
  }

  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* Large desktop: 1280px+ */
@media (min-width: 1280px) {
  .container {
    max-width: 1200px;
  }

  .grid {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

Adaptive components:
```javascript
// Show different components based on screen size
const useMediaQuery = (query) => {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);
    setMatches(media.matches);

    const listener = (e) => setMatches(e.matches);
    media.addEventListener('change', listener);
    return () => media.removeEventListener('change', listener);
  }, [query]);

  return matches;
};

const Navigation = () => {
  const isMobile = useMediaQuery('(max-width: 768px)');

  return isMobile ? (
    <MobileNav />
  ) : (
    <DesktopNav />
  );
};
```

**Examples**:

Good example: Responsive table
```javascript
// Desktop: Full table
// Mobile: Card layout

const DataTable = ({ data }) => {
  const isMobile = useMediaQuery('(max-width: 768px)');

  if (isMobile) {
    return (
      <CardList>
        {data.map(item => (
          <Card key={item.id}>
            <CardRow>
              <Label>Name:</Label>
              <Value>{item.name}</Value>
            </CardRow>
            <CardRow>
              <Label>Email:</Label>
              <Value>{item.email}</Value>
            </CardRow>
            <CardRow>
              <Label>Status:</Label>
              <Value>{item.status}</Value>
            </CardRow>
          </Card>
        ))}
      </CardList>
    );
  }

  return (
    <Table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        {data.map(item => (
          <tr key={item.id}>
            <td>{item.name}</td>
            <td>{item.email}</td>
            <td>{item.status}</td>
          </tr>
        ))}
      </tbody>
    </Table>
  );
};
```

**Anti-patterns**:
- Desktop-only designs
- Horizontal scrolling on mobile
- Tiny text on mobile
- Hidden content with no way to access

**Related guidelines**: Mobile vs Desktop (80), Layout Consistency (83)

---

## Platform-Specific Checklist

**Mobile**:
- [ ] Touch targets minimum 44x44px
- [ ] Adequate spacing between interactive elements
- [ ] No hover-dependent functionality
- [ ] Bottom navigation for primary actions
- [ ] Thumb-friendly zones utilized

**Desktop**:
- [ ] Hover states for interactive elements
- [ ] Keyboard shortcuts available
- [ ] Mouse interactions optimized
- [ ] Multi-column layouts utilized
- [ ] Tooltips on hover

**Touch Gestures**:
- [ ] Swipe gestures feel natural
- [ ] Gestures are discoverable
- [ ] Direct manipulation where possible
- [ ] Long-press actions clearly indicated

**Platform Conventions**:
- [ ] Navigation matches platform (iOS/Android/Web)
- [ ] Button placement follows platform standards
- [ ] Dialogs match platform style
- [ ] Feedback mechanisms platform-appropriate

**Related guidelines**: Touch Interface Design (76), Fitts's Law (16)
