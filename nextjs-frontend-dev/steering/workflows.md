# Common Development Workflows

Practical workflows for day-to-day frontend development with this power.

## Starting a Development Session

Every time you open your project:

1. Start the dev server: `npm run dev`
2. Call the `init` tool from next-devtools MCP
3. You're ready to build

The `init` tool loads Next.js documentation context and connects to your running dev server so the agent can access runtime diagnostics.

## Building a New Page

### 1. Create the page file

Ask the agent to create a new page. Provide context about what the page should do:

"Create a pricing page at /pricing with a header, three pricing tiers using shadcn Card components, and a FAQ section"

The agent will:
- Create `app/pricing/page.tsx` as a Server Component
- Use shadcn components (installing any that are missing)
- Apply Tailwind CSS v4 utility classes
- Follow App Router conventions

### 2. Add interactivity where needed

If parts of the page need client-side interactivity (dropdowns, modals, form validation), the agent will add `'use client'` only to those specific components, keeping the page itself as a Server Component.

### 3. Verify in browser

Ask the agent to use Playwright to verify the page:

"Use Playwright to navigate to /pricing and verify the page loads with three pricing cards"

## Adding UI Components

### Browse available components

"Show me all available components in the shadcn registry"

### Install specific components

"Add the dialog, dropdown-menu, and toast components to my project"

shadcn components are copied into `components/ui/` — they're your code to customize.

### Use components in pages

"Build a settings page using the tabs, form, input, and button components from shadcn"

The agent knows the installed components and will import them correctly.

## Debugging Errors

### Check for errors

"What errors are in my Next.js application?"

The agent calls `get_errors` to retrieve:
- Build errors
- Runtime errors
- TypeScript type errors

### Check dev server logs

"Show me the development server logs"

The agent calls `get_logs` to access server output and browser console logs.

### Inspect a specific page

"Show me the metadata and route info for the /dashboard page"

The agent calls `get_page_metadata` to inspect routes, components, and rendering details.

## Writing E2E Tests

### Generate tests for a page

"Write a Playwright E2E test for the login page that tests successful login and invalid credentials"

The agent will create a test file in `e2e/` following Playwright best practices.

### Run tests with Playwright MCP

"Use Playwright to test the checkout flow: add item to cart, go to checkout, fill in details, and submit"

The Playwright MCP server automates the browser directly, navigating pages and interacting with elements via accessibility snapshots.

### Run the test suite

Tests can be run via:

```bash
npx playwright test
```

Or for a specific test:

```bash
npx playwright test e2e/login.spec.ts
```

## Working with Layouts

### Create a layout

"Create a dashboard layout with a sidebar navigation and top header bar"

The agent creates `app/dashboard/layout.tsx` that wraps all pages under `/dashboard/`.

### Use route groups

"Organize the marketing pages (about, pricing, contact) into a route group with a shared marketing layout"

The agent creates:
- `app/(marketing)/layout.tsx` — shared layout
- `app/(marketing)/about/page.tsx`
- `app/(marketing)/pricing/page.tsx`
- `app/(marketing)/contact/page.tsx`

Route groups `(name)` don't affect the URL — `/about` still works, not `/(marketing)/about`.

## API Routes

### Create an API endpoint

"Create a POST API route at /api/contact that validates email and message fields and returns a success response"

The agent creates `app/api/contact/route.ts` using Next.js Route Handlers.

### Server Actions

For form submissions that don't need a separate API route:

"Create a Server Action for the contact form that validates and saves the submission"

Server Actions run on the server and can be called directly from Client Components.

## Performance Monitoring

### Check Web Vitals

If you've set up the Web Vitals component (see project-setup steering file), metrics are automatically reported.

In development, metrics are logged to the console. In production, they're sent to your `/api/analytics` endpoint.

### Key metrics to watch

- **LCP > 2.5s** — Optimize images, reduce server response time, minimize render-blocking resources
- **CLS > 0.1** — Add explicit dimensions to images/videos, avoid inserting content above existing content
- **INP > 200ms** — Reduce JavaScript execution time, break up long tasks, use `startTransition` for non-urgent updates

### Using next-devtools for performance

"Show me the project metadata and configuration"

The agent calls `get_project_metadata` to review your Next.js config, which can reveal performance-related settings.

## Upgrading Next.js

"Help me upgrade my Next.js app to the latest version"

The next-devtools MCP includes migration and upgrade tools with automated codemods for handling breaking changes.

## Common Patterns

### Loading states

Create `loading.tsx` alongside any `page.tsx` to show a loading UI while the page loads:

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div className="animate-pulse">Loading...</div>
}
```

### Error boundaries

Create `error.tsx` to handle errors gracefully:

```tsx
// app/dashboard/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

### Not found pages

Create `not-found.tsx` for custom 404 pages:

```tsx
// app/not-found.tsx
import Link from 'next/link'

export default function NotFound() {
  return (
    <div>
      <h2>Page Not Found</h2>
      <Link href="/">Go home</Link>
    </div>
  )
}
```
