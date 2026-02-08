# Accessibility Standards

Every component and page should meet WCAG AA standards. This guide covers the key practices to follow when building Next.js frontend UIs.

## Semantic HTML

Use the right element for the job — don't style a `<div>` to look like a button.

| Purpose | Use | Don't Use |
|---------|-----|-----------|
| Navigation | `<nav>` | `<div className="nav">` |
| Page sections | `<main>`, `<section>`, `<aside>` | `<div>` |
| Headings | `<h1>` through `<h6>` in order | `<p className="heading">` |
| Clickable actions | `<button>` | `<div onClick>` |
| Links to pages | `<a href>` or Next.js `<Link>` | `<button onClick={navigate}>` |
| Lists | `<ul>`, `<ol>`, `<li>` | `<div>` with styled items |
| Forms | `<form>`, `<label>`, `<input>` | Unlabeled inputs |

## ARIA Labels

Use ARIA attributes when semantic HTML alone isn't enough:

```tsx
// Icon-only button needs a label
<button aria-label="Close dialog">
  <XIcon />
</button>

// Describe the current state
<button aria-expanded={isOpen} aria-controls="menu-panel">
  Menu
</button>

// Live regions for dynamic content
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>

// Loading states announced to screen readers
<div aria-busy={isLoading} aria-live="polite">
  {isLoading ? 'Loading...' : content}
</div>
```

Rules of thumb:
- If it has no visible text, add `aria-label`
- If it toggles something, add `aria-expanded`
- If content updates dynamically, use `aria-live`
- Don't overuse ARIA — semantic HTML is always preferred

## Keyboard Navigation

Every interactive element must be reachable and operable via keyboard:

```tsx
// Focus visible states with Tailwind
<button className="focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2">
  Click me
</button>

// Trap focus in modals (shadcn Dialog handles this automatically)
<Dialog>
  <DialogContent>
    {/* Focus is trapped here until closed */}
  </DialogContent>
</Dialog>

// Custom keyboard handlers when needed
<div
  role="listbox"
  tabIndex={0}
  onKeyDown={(e) => {
    if (e.key === 'ArrowDown') selectNext()
    if (e.key === 'ArrowUp') selectPrevious()
    if (e.key === 'Enter') confirmSelection()
    if (e.key === 'Escape') close()
  }}
>
```

Key requirements:
- All interactive elements focusable via Tab
- Visible focus indicator (never `outline: none` without replacement)
- Escape closes modals, dropdowns, popovers
- Arrow keys navigate within composite widgets (menus, listboxes, tabs)
- Enter/Space activates buttons and links

## Color and Contrast

- Minimum contrast ratio of 4.5:1 for normal text (WCAG AA)
- Minimum contrast ratio of 3:1 for large text (18px+ or 14px+ bold)
- Never convey information through color alone — add icons, text, or patterns

```tsx
// Good: error state uses color AND icon AND text
<div className="text-red-600 flex items-center gap-2">
  <AlertCircle className="h-4 w-4" />
  <span>Email is required</span>
</div>

// Bad: only color indicates error
<input className="border-red-500" />
```

Dark mode considerations:
```tsx
// Ensure contrast in both modes
<p className="text-gray-900 dark:text-gray-100">Readable in both modes</p>

// Avoid low-contrast combinations
// Bad: text-gray-400 on white (fails AA)
// Good: text-gray-600 on white (passes AA)
```

## Images and Media

```tsx
// Informative images need descriptive alt text
<Image src="/chart.png" alt="Monthly revenue chart showing 20% growth in Q4" />

// Decorative images use empty alt
<Image src="/decorative-wave.svg" alt="" aria-hidden="true" />

// Next.js Image component handles lazy loading automatically
import Image from 'next/image'
<Image src="/photo.jpg" alt="Team photo at the office" width={800} height={600} />
```

## Forms

```tsx
// Every input needs a visible label
<div>
  <label htmlFor="email" className="block text-sm font-medium">
    Email address
  </label>
  <input
    id="email"
    type="email"
    aria-required="true"
    aria-invalid={!!errors.email}
    aria-describedby={errors.email ? 'email-error' : undefined}
    className="mt-1 block w-full rounded-md border"
  />
  {errors.email && (
    <p id="email-error" role="alert" className="mt-1 text-sm text-red-600">
      {errors.email.message}
    </p>
  )}
</div>

// Group related fields
<fieldset>
  <legend className="text-sm font-medium">Notification preferences</legend>
  {/* radio buttons or checkboxes */}
</fieldset>

// Submit buttons should indicate loading state
<button type="submit" disabled={isSubmitting} aria-busy={isSubmitting}>
  {isSubmitting ? 'Submitting...' : 'Submit'}
</button>
```

## shadcn Component Accessibility

shadcn components are built on Radix UI primitives, which handle most accessibility out of the box:

- **Dialog** — Focus trapping, Escape to close, aria-labelledby
- **DropdownMenu** — Arrow key navigation, typeahead, focus management
- **Tabs** — Arrow key switching, proper ARIA roles
- **Toast** — aria-live announcements
- **Select** — Keyboard navigation, screen reader support

When customizing shadcn components, don't remove:
- `role` attributes
- `aria-*` attributes
- Keyboard event handlers
- Focus management logic

## Quick Checklist

When building or reviewing a component:

- [ ] Can you Tab to every interactive element?
- [ ] Is there a visible focus indicator?
- [ ] Can you operate it with keyboard only (Enter, Space, Escape, Arrows)?
- [ ] Does every image have appropriate alt text?
- [ ] Does every form input have a visible label?
- [ ] Are error messages associated with their inputs via `aria-describedby`?
- [ ] Is dynamic content announced via `aria-live`?
- [ ] Does text meet 4.5:1 contrast ratio?
- [ ] Does it work in both light and dark mode?
- [ ] Is information conveyed by more than just color?
