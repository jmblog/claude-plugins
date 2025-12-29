# Implementation Patterns

Code examples and implementation guidance for common UI patterns across React, Vue, and vanilla JavaScript.

## Contents
- [Button Patterns](#button-patterns)
- [Form Patterns](#form-patterns)
- [Modal & Dialog Patterns](#modal--dialog-patterns)
- [Navigation Patterns](#navigation-patterns)
- [Feedback Patterns](#feedback-patterns)
- [Loading Patterns](#loading-patterns)

---

## Button Patterns

### Button Hierarchy

**React**:
```jsx
// Button component with variants
const Button = ({ variant = 'primary', children, ...props }) => {
  const styles = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-white text-blue-600 border border-blue-600 hover:bg-blue-50',
    danger: 'bg-red-600 text-white hover:bg-red-700',
    tertiary: 'bg-transparent text-gray-600 hover:bg-gray-100'
  };

  return (
    <button
      className={`px-4 py-2 rounded font-medium transition ${styles[variant]}`}
      {...props}
    >
      {children}
    </button>
  );
};

// Usage
<div className="flex gap-2">
  <Button variant="primary">Save</Button>
  <Button variant="secondary">Cancel</Button>
  <Button variant="danger">Delete</Button>
</div>
```

**Vue**:
```vue
<!-- Button.vue -->
<template>
  <button
    :class="['btn', `btn-${variant}`]"
    v-bind="$attrs"
  >
    <slot />
  </button>
</template>

<script setup>
defineProps({
  variant: {
    type: String,
    default: 'primary',
    validator: (v) => ['primary', 'secondary', 'danger'].includes(v)
  }
});
</script>

<style scoped>
.btn {
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  font-weight: 500;
  transition: all 0.2s;
}

.btn-primary {
  background: #0066cc;
  color: white;
}

.btn-primary:hover {
  background: #0052a3;
}
</style>

<!-- Usage -->
<template>
  <div class="flex gap-2">
    <Button variant="primary">Save</Button>
    <Button variant="secondary">Cancel</Button>
  </div>
</template>
```

**Vanilla JS**:
```javascript
// Button factory
function createButton(text, variant = 'primary', onClick) {
  const button = document.createElement('button');
  button.textContent = text;
  button.className = `btn btn-${variant}`;

  if (onClick) {
    button.addEventListener('click', onClick);
  }

  return button;
}

// CSS
const styles = `
.btn {
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-primary {
  background: #0066cc;
  color: white;
}

.btn-primary:hover {
  background: #0052a3;
}
`;

// Usage
const saveButton = createButton('Save', 'primary', () => {
  console.log('Saving...');
});
document.body.appendChild(saveButton);
```

---

## Form Patterns

### Validated Input Field

**React**:
```jsx
const FormField = ({ label, error, required, children, id }) => (
  <div className="form-field">
    <label htmlFor={id} className="font-medium">
      {label}
      {required && <span className="text-red-600" aria-label="required">*</span>}
    </label>
    {children}
    {error && (
      <p className="text-red-600 text-sm mt-1" role="alert">
        {error}
      </p>
    )}
  </div>
);

const ValidatedInput = ({ label, value, onChange, validate, required, ...props }) => {
  const [error, setError] = useState('');
  const id = useId();

  const handleBlur = () => {
    if (validate) {
      const err = validate(value);
      setError(err || '');
    }
  };

  return (
    <FormField label={label} error={error} required={required} id={id}>
      <input
        id={id}
        value={value}
        onChange={(e) => {
          onChange(e.target.value);
          if (error) setError(''); // Clear error on change
        }}
        onBlur={handleBlur}
        aria-invalid={error ? 'true' : 'false'}
        aria-describedby={error ? `${id}-error` : undefined}
        className={`w-full px-3 py-2 border rounded ${
          error ? 'border-red-600' : 'border-gray-300'
        }`}
        {...props}
      />
    </FormField>
  );
};

// Usage
<ValidatedInput
  label="Email"
  value={email}
  onChange={setEmail}
  validate={(v) => {
    if (!v) return 'Email is required';
    if (!/\S+@\S+\.\S+/.test(v)) return 'Invalid email format';
    return '';
  }}
  required
/>
```

**Vue**:
```vue
<!-- ValidatedInput.vue -->
<template>
  <div class="form-field">
    <label :for="id" class="font-medium">
      {{ label }}
      <span v-if="required" class="text-red-600" aria-label="required">*</span>
    </label>
    <input
      :id="id"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      @blur="handleBlur"
      :aria-invalid="error ? 'true' : 'false'"
      :class="['input', { 'error': error }]"
      v-bind="$attrs"
    />
    <p v-if="error" class="error-message" role="alert">
      {{ error }}
    </p>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  label: String,
  modelValue: String,
  validate: Function,
  required: Boolean
});

const emit = defineEmits(['update:modelValue']);

const error = ref('');
const id = computed(() => `input-${Math.random().toString(36).substr(2, 9)}`);

const handleBlur = () => {
  if (props.validate) {
    error.value = props.validate(props.modelValue) || '';
  }
};
</script>

<style scoped>
.input {
  width: 100%;
  padding: 0.5rem 0.75rem;
  border: 1px solid #d1d5db;
  border-radius: 0.25rem;
}

.input.error {
  border-color: #dc2626;
}

.error-message {
  color: #dc2626;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}
</style>
```

---

## Modal & Dialog Patterns

### Accessible Modal

**React**:
```jsx
const Modal = ({ isOpen, onClose, title, children }) => {
  const modalRef = useRef(null);

  useEffect(() => {
    if (isOpen) {
      const lastFocused = document.activeElement;

      // Focus trap
      const handleTab = (e) => {
        if (e.key === 'Tab') {
          const focusableElements = modalRef.current.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
          );
          const firstElement = focusableElements[0];
          const lastElement = focusableElements[focusableElements.length - 1];

          if (e.shiftKey && document.activeElement === firstElement) {
            lastElement.focus();
            e.preventDefault();
          } else if (!e.shiftKey && document.activeElement === lastElement) {
            firstElement.focus();
            e.preventDefault();
          }
        }
      };

      // Close on Escape
      const handleEscape = (e) => {
        if (e.key === 'Escape') onClose();
      };

      document.addEventListener('keydown', handleTab);
      document.addEventListener('keydown', handleEscape);

      modalRef.current?.focus();

      return () => {
        document.removeEventListener('keydown', handleTab);
        document.removeEventListener('keydown', handleEscape);
        lastFocused?.focus();
      };
    }
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={modalRef}
        className="modal"
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        onClick={(e) => e.stopPropagation()}
      >
        <div className="modal-header">
          <h2 id="modal-title">{title}</h2>
          <button onClick={onClose} aria-label="Close dialog">×</button>
        </div>
        <div className="modal-content">{children}</div>
      </div>
    </div>
  );
};

// CSS
const styles = `
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  background: white;
  border-radius: 0.5rem;
  max-width: 500px;
  width: 90%;
  max-height: 90vh;
  overflow: auto;
}
`;
```

---

## Navigation Patterns

### Responsive Navigation

**React**:
```jsx
const Navigation = () => {
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);
  const isMobile = useMediaQuery('(max-width: 768px)');

  return (
    <nav className="navbar">
      {isMobile ? (
        <>
          <button
            onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
            aria-label="Menu"
            aria-expanded={mobileMenuOpen}
          >
            ☰
          </button>
          {mobileMenuOpen && (
            <div className="mobile-menu">
              <NavLink href="/">Home</NavLink>
              <NavLink href="/projects">Projects</NavLink>
              <NavLink href="/about">About</NavLink>
            </div>
          )}
        </>
      ) : (
        <div className="desktop-menu">
          <NavLink href="/">Home</NavLink>
          <NavLink href="/projects">Projects</NavLink>
          <NavLink href="/about">About</NavLink>
        </div>
      )}
    </nav>
  );
};

const NavLink = ({ href, children }) => {
  const isActive = useMatch(href);

  return (
    <a
      href={href}
      className={isActive ? 'nav-link active' : 'nav-link'}
      aria-current={isActive ? 'page' : undefined}
    >
      {children}
    </a>
  );
};
```

---

## Feedback Patterns

### Toast Notifications

**React**:
```jsx
// Toast context
const ToastContext = createContext();

export const ToastProvider = ({ children }) => {
  const [toasts, setToasts] = useState([]);

  const showToast = (message, type = 'info', duration = 3000) => {
    const id = Date.now();
    setToasts(prev => [...prev, { id, message, type }]);

    setTimeout(() => {
      setToasts(prev => prev.filter(t => t.id !== id));
    }, duration);
  };

  return (
    <ToastContext.Provider value={{ showToast }}>
      {children}
      <div className="toast-container">
        {toasts.map(toast => (
          <Toast
            key={toast.id}
            message={toast.message}
            type={toast.type}
            onClose={() => setToasts(prev => prev.filter(t => t.id !== toast.id))}
          />
        ))}
      </div>
    </ToastContext.Provider>
  );
};

const Toast = ({ message, type, onClose }) => (
  <div className={`toast toast-${type}`} role="status" aria-live="polite">
    {type === 'success' && '✓'}
    {type === 'error' && '✗'}
    {message}
    <button onClick={onClose} aria-label="Close">×</button>
  </div>
);

// Usage
const MyComponent = () => {
  const { showToast } = useContext(ToastContext);

  return (
    <button onClick={() => showToast('Saved successfully!', 'success')}>
      Save
    </button>
  );
};
```

---

## Loading Patterns

### Skeleton Screen

**React**:
```jsx
const Skeleton = ({ width, height, className }) => (
  <div
    className={`skeleton ${className}`}
    style={{ width, height }}
    aria-hidden="true"
  />
);

const SkeletonCard = () => (
  <div className="card">
    <Skeleton width="100%" height="200px" />
    <div className="card-body">
      <Skeleton width="70%" height="24px" />
      <Skeleton width="100%" height="16px" />
      <Skeleton width="100%" height="16px" />
      <Skeleton width="40%" height="16px" />
    </div>
  </div>
);

// CSS
const styles = `
.skeleton {
  background: linear-gradient(
    90deg,
    #f0f0f0 25%,
    #e0e0e0 50%,
    #f0f0f0 75%
  );
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
  border-radius: 4px;
}

@keyframes loading {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
`;

// Usage with data
const PostList = () => {
  const { data, loading } = useQuery();

  if (loading) {
    return (
      <div>
        <SkeletonCard />
        <SkeletonCard />
        <SkeletonCard />
      </div>
    );
  }

  return data.map(post => <PostCard key={post.id} post={post} />);
};
```

### Progress Indicator

**React**:
```jsx
const ProgressBar = ({ value, max = 100, label }) => {
  const percentage = (value / max) * 100;

  return (
    <div className="progress-container">
      {label && <label>{label}</label>}
      <div
        className="progress-bar"
        role="progressbar"
        aria-valuenow={value}
        aria-valuemin={0}
        aria-valuemax={max}
        aria-label={label}
      >
        <div
          className="progress-fill"
          style={{ width: `${percentage}%` }}
        />
      </div>
      <span className="progress-text">{Math.round(percentage)}%</span>
    </div>
  );
};

// Indeterminate spinner
const Spinner = ({ size = 'md' }) => {
  const sizes = {
    sm: '16px',
    md: '24px',
    lg: '40px'
  };

  return (
    <div
      className="spinner"
      style={{ width: sizes[size], height: sizes[size] }}
      role="status"
      aria-label="Loading"
    >
      <span className="sr-only">Loading...</span>
    </div>
  );
};
```

---

## Quick Reference

**Component patterns**:
- Always include accessibility attributes (ARIA labels, roles)
- Provide keyboard navigation
- Include loading and error states
- Use semantic HTML
- Handle focus management

**Styling patterns**:
- Use CSS custom properties for theming
- Implement responsive breakpoints
- Provide hover/active/focus states
- Use transitions for smooth interactions

**State management**:
- Handle loading states
- Show errors constructively
- Provide immediate feedback
- Maintain UI responsiveness

**Framework-specific best practices**:
- **React**: Use hooks for state, refs for DOM access, context for global state
- **Vue**: Use composition API, reactive refs, computed properties
- **Vanilla**: Progressive enhancement, event delegation, modern APIs

For more implementation examples, see framework documentation and the guidelines referenced throughout this skill.
