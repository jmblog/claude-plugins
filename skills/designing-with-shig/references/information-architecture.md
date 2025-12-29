# Information Architecture

Information architecture principles guide how content is organized, structured, and made discoverable.

## Contents
- [Wayfinding](#wayfinding)
- [Navigation Design](#navigation-design)
- [Object-Based Design](#object-based-design)
- [Views Represent Objects](#views-represent-objects)
- [Use Nouns for Navigation](#use-nouns-for-navigation)

---

## Wayfinding

**Core principle**: Help users understand where they are, where they can go, and how to get there.

**Apply when**:
- Designing navigation systems
- Creating multi-page applications
- Building complex information spaces

**Implementation**:

Breadcrumbs for hierarchy:
```html
<nav aria-label="Breadcrumb">
  <a href="/">Home</a> >
  <a href="/products">Products</a> >
  <a href="/products/electronics">Electronics</a> >
  <span aria-current="page">Headphones</span>
</nav>
```

Active page indication:
```html
<nav>
  <a href="/dashboard" class="active">Dashboard</a>
  <a href="/projects">Projects</a>
  <a href="/settings">Settings</a>
</nav>

<style>
.active {
  font-weight: 700;
  color: #0066cc;
  border-bottom: 2px solid #0066cc;
}
</style>
```

Clear page titles:
```html
<header>
  <h1>Account Settings</h1>
  <p>Manage your profile and preferences</p>
</header>
```

**Examples**:

Good example: Multi-level wayfinding
```html
<Layout>
  <!-- Global navigation -->
  <TopNav>
    <Logo />
    <NavItem active>Dashboard</NavItem>
    <NavItem>Reports</NavItem>
  </TopNav>

  <!-- Breadcrumb -->
  <Breadcrumb>
    Home > Dashboard > Analytics
  </Breadcrumb>

  <!-- Page header -->
  <PageHeader>
    <h1>Analytics Dashboard</h1>
  </PageHeader>

  <!-- Content -->
  <Main>...</Main>
</Layout>
```

**Anti-patterns**:
- No indication of current location
- Ambiguous page titles
- Hidden navigation paths
- No way to return to previous level

**Related guidelines**: Escape Hatch (59), Navigation Hierarchy (78)

---

## Navigation Design

**Core principle**: Create clear, discoverable navigation schemes that match user mental models.

**Apply when**:
- Designing site/app navigation
- Organizing content hierarchies
- Creating menu structures

**Implementation**:

Primary navigation patterns:
```html
<!-- Horizontal top navigation for main sections -->
<nav class="primary-nav">
  <a href="/dashboard">Dashboard</a>
  <a href="/projects">Projects</a>
  <a href="/team">Team</a>
  <a href="/settings">Settings</a>
</nav>

<!-- Vertical sidebar for sub-navigation -->
<aside class="sidebar-nav">
  <h2>Projects</h2>
  <a href="/projects/active">Active</a>
  <a href="/projects/archived">Archived</a>
  <a href="/projects/templates">Templates</a>
</aside>
```

Mobile navigation:
```html
<!-- Hamburger menu for mobile -->
<button class="menu-toggle" aria-label="Menu">
  <Icon name="menu" />
</button>

<nav class="mobile-nav" hidden>
  <a href="/dashboard">Dashboard</a>
  <a href="/projects">Projects</a>
  <a href="/team">Team</a>
</nav>
```

**Examples**:

Good example: Clear hierarchy
```html
<nav>
  <!-- Level 1: Main sections -->
  <NavGroup>
    <NavItem href="/dashboard">Dashboard</NavItem>
    <NavItem href="/projects" expanded>
      Projects
      <!-- Level 2: Subsections -->
      <SubNav>
        <NavItem href="/projects/active">Active</NavItem>
        <NavItem href="/projects/archived">Archived</NavItem>
      </SubNav>
    </NavItem>
  </NavGroup>
</nav>
```

**Anti-patterns**:
- Deep nesting (>3 levels)
- Hidden essential functions
- Unclear navigation labels
- No visual hierarchy

**Related guidelines**: Wayfinding, Mental Model (3), Use Nouns for Navigation (33)

---

## Object-Based Design

**Core principle**: Design around user mental models of objects. Allow direct interaction with key objects.

**Apply when**:
- Designing CRUD applications
- Creating data-centric interfaces
- Building content management systems

**Implementation**:

Identify core objects from user tasks:
```
User tasks → Objects
- Send email → Email message
- Manage contacts → Contact
- Schedule meeting → Event/Calendar entry
- Edit document → Document
```

Design object-centric UI:
```html
<!-- Email app: Objects are messages -->
<MessageList>
  <Message
    from="user@example.com"
    subject="Meeting tomorrow"
    onSelect={() => openMessage(id)}
    onArchive={() => archiveMessage(id)}
    onDelete={() => deleteMessage(id)}
  />
</MessageList>

<!-- Not function-centric:
  [Archive] [Delete] [Move] buttons first,
  then select message -->
```

**Examples**:

Good example: Direct object manipulation
```javascript
// File manager: Files are the objects
<FileGrid>
  {files.map(file => (
    <File
      name={file.name}
      icon={file.icon}
      // Direct manipulation
      draggable
      onDragStart={() => startDrag(file)}
      onClick={() => openFile(file)}
      onRename={(name) => renameFile(file, name)}
      onDelete={() => deleteFile(file)}
    />
  ))}
</FileGrid>
```

**Anti-patterns**:
- Function-first design (tools before objects)
- Abstract organization not matching user tasks
- Requiring selection before seeing options

**Related guidelines**: Mental Model (3), Noun→Verb Operation Order (26), Direct Manipulation (8)

---

## Views Represent Objects

**Core principle**: Interface is composed of object representation views. Objects can have multiple view representations.

**Apply when**:
- Designing data displays
- Creating dashboards
- Building visualization tools

**Implementation**:

Multiple views of same object:
```javascript
// Same data, different representations
const project = {
  id: 1,
  name: 'Website Redesign',
  status: 'in-progress',
  dueDate: '2024-12-31',
  tasks: [...]
};

// List view
<ProjectListItem project={project} />

// Card view
<ProjectCard project={project} />

// Detail view
<ProjectDetail project={project} />

// Calendar view
<CalendarEvent project={project} />
```

Data binding across views:
```javascript
// When object changes, all views update
const updateProject = (updates) => {
  project = { ...project, ...updates };
  // All views automatically reflect changes
};
```

**Examples**:

Good example: Synchronized views
```html
<Dashboard>
  <!-- Table view -->
  <TaskTable data={tasks} onUpdate={updateTask} />

  <!-- Kanban view -->
  <TaskBoard data={tasks} onUpdate={updateTask} />

  <!-- Both views show same tasks, stay synchronized -->
</Dashboard>
```

**Anti-patterns**:
- Inconsistent representations of same data
- Views that don't update together
- Different object states in different views

**Related guidelines**: Object-Based Design (23), Data Binding (35)

---

## Use Nouns for Navigation

**Core principle**: Navigation items should be noun-based, describing the destination content.

**Apply when**:
- Designing navigation menus
- Creating sitemap structures
- Labeling navigation links

**Implementation**:

Noun-based labels:
```html
<!-- Good: Nouns -->
<nav>
  <a href="/dashboard">Dashboard</a>
  <a href="/projects">Projects</a>
  <a href="/team">Team</a>
  <a href="/settings">Settings</a>
</nav>

<!-- Bad: Verbs -->
<nav>
  <a href="/dashboard">View Dashboard</a>
  <a href="/projects">Manage Projects</a>
  <a href="/team">See Team</a>
  <a href="/settings">Change Settings</a>
</nav>
```

Content-focused organization:
```html
<nav>
  <!-- Organized by content type (nouns) -->
  <NavSection title="Content">
    <NavItem>Posts</NavItem>
    <NavItem>Pages</NavItem>
    <NavItem>Media</NavItem>
  </NavSection>

  <NavSection title="Users">
    <NavItem>All Users</NavItem>
    <NavItem>Roles</NavItem>
  </NavSection>
</nav>
```

**Examples**:

Good example: Clear content labels
```
Main navigation:
- Home
- Products
- About
- Contact

Not:
- Go Home
- Browse Products
- Learn About Us
- Get in Touch
```

**Anti-patterns**:
- Action-based navigation ("Create", "Edit", "Delete")
- Verb phrases in main navigation
- Unclear destination

**Related guidelines**: Object-Based Design (23), Mental Model (3)

---

## Navigation Patterns

### Hierarchical Navigation

For deep content structures:
```html
<nav>
  <!-- Level 1 -->
  <a href="/docs">Documentation</a>

  <!-- Level 2 -->
  <ul>
    <li><a href="/docs/getting-started">Getting Started</a></li>
    <li>
      <a href="/docs/guides">Guides</a>
      <!-- Level 3 -->
      <ul>
        <li><a href="/docs/guides/basics">Basics</a></li>
        <li><a href="/docs/guides/advanced">Advanced</a></li>
      </ul>
    </li>
  </ul>
</nav>
```

### Flat Navigation

For shallow structures:
```html
<nav>
  <a href="/dashboard">Dashboard</a>
  <a href="/analytics">Analytics</a>
  <a href="/reports">Reports</a>
  <a href="/settings">Settings</a>
</nav>
```

### Hub Navigation

For distinct sections:
```html
<!-- Main sections are separate "hubs" -->
<nav>
  <a href="/workspace">Workspace</a>
  <a href="/reports">Reports</a>
  <a href="/admin">Admin</a>
</nav>

<!-- Each hub has own sub-navigation -->
```

**Related guidelines**: Navigation Hierarchy (78), Wayfinding, Menu Item Positioning (71)
