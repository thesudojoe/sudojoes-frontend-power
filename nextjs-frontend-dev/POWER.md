---
name: "nextjs-frontend-dev"
displayName: "Next.js Frontend Dev"
description: "Complete frontend development power for Next.js projects. Integrates Next.js DevTools MCP for runtime diagnostics, shadcn for UI components, and Playwright for browser testing. Includes guidance on Tailwind CSS v4, project structure, Web Vitals analytics, and best practices."
keywords: ["nextjs", "frontend", "shadcn", "playwright", "tailwind"]
author: "Kiro User"
---

# Next.js Frontend Dev

## Overview

This power streamlines frontend development with Next.js 16+ by combining three MCP servers into a single workflow: Next.js DevTools for runtime diagnostics and documentation, shadcn for UI component management, and Playwright for browser testing.

It also provides guidance on project structure conventions, Tailwind CSS v4 styling, Web Vitals analytics with `useReportWebVitals`, and frontend best practices so you can get productive quickly without researching each tool individually.

## Available Steering Files

- **project-setup** — Step-by-step guide for initializing a new Next.js project with Tailwind CSS v4, shadcn, Playwright, and analytics
- **workflows** — Common development workflows: building pages, adding components, testing, and debugging
- **ai-ui** — Building AI-powered UIs with the Vercel AI SDK (useChat, useCompletion, useObject, tool calling). Load this when creating chat interfaces, AI completions, or streamed structured data.
- **accessibility** — WCAG AA accessibility standards, semantic HTML, ARIA patterns, keyboard navigation, and a component checklist. Load this when building or reviewing UI components.
- **forms** — Form patterns with shadcn, react-hook-form, zod, and Server Actions. Covers validation, multi-step forms, search with debounce, and auth forms. Load this when building forms.

## Onboarding

### Prerequisites

- Node.js v20.19+ (LTS recommended)
- npm or pnpm
- A Next.js 16+ project (or willingness to create one)

### Quick Start

1. Start your Next.js dev server:

```bash
npm run dev
```

2. Call the `init` tool from the next-devtools MCP server at the start of every session. This loads Next.js documentation context and connects to your running dev server.

3. You're ready to go. Ask the agent to build pages, add components, check for errors, or run browser tests.

### Bundled Hook: Auto-Init Next.js DevTools

This power includes a hook that automatically calls the `init` tool at the start of every session. You don't need to remember to do it — the hook fires on your first prompt and silently connects to your running dev server, loading Next.js documentation context.

### Project Initialization

If you're starting from scratch, read the **project-setup** steering file for a complete walkthrough:

```
Call action "readSteering" with steeringFile="project-setup.md"
```

This covers:
- Creating a Next.js 16+ app with App Router
- Installing and configuring Tailwind CSS v4
- Setting up shadcn components
- Configuring Playwright for E2E testing
- Adding Web Vitals analytics

## MCP Servers

This power includes three MCP servers that work together:

### next-devtools (Next.js DevTools MCP)

Connects to your running Next.js 16+ dev server and provides runtime diagnostics.

**Key tools:**
- `init` — Initialize context and load Next.js documentation (call this first every session)
- `get_errors` — Retrieve build errors, runtime errors, and TypeScript type errors
- `get_logs` — Access dev server logs and browser console output
- `get_page_metadata` — Inspect routes, components, and rendering info for specific pages
- `get_project_metadata` — Get project structure, config, and dev server URL
- `get_server_action_by_id` — Look up Server Actions by ID

**How it works:** Next.js 16+ exposes a built-in MCP endpoint at `/_next/mcp` on your dev server. The next-devtools-mcp package discovers this endpoint automatically.

### shadcn (Component Library MCP)

Browse, search, and install UI components from shadcn registries using natural language.

**Key capabilities:**
- Browse all available components, blocks, and templates
- Search across multiple registries (shadcn/ui, private, third-party)
- Install components directly into your project
- Support for namespaced registries (`@namespace/component`)

**Example prompts:**
- "Show me all available components in the shadcn registry"
- "Add the button, dialog, and card components to my project"
- "Create a contact form using shadcn components"
- "Install the sidebar block from shadcn"

**Configuration:** Components are configured via `components.json` in your project root. The default shadcn/ui registry works out of the box.

### playwright (Browser Testing MCP)

Automate browser interactions for E2E testing and visual verification.

**Key features:**
- Uses Playwright's accessibility tree (no vision models needed)
- Navigate pages, click elements, fill forms, take screenshots
- Deterministic tool application via structured accessibility snapshots
- Works with Chromium, Firefox, and WebKit

**Common uses:**
- Verify pages render correctly after changes
- Test user flows (login, form submission, navigation)
- Take screenshots for visual review
- Debug layout and interaction issues

## CSS Framework: Tailwind CSS v4

This power recommends Tailwind CSS v4 as the styling solution. It pairs naturally with shadcn components and Next.js.

### Why Tailwind CSS v4

- Up to 100x faster incremental builds (5ms vs 44ms in v3)
- Zero-config setup with automatic content detection
- CSS-first configuration using `@theme` directive instead of `tailwind.config.js`
- Native cascade layers, `@property`, and `color-mix()` support
- Built-in container queries and 3D transform utilities

### Tailwind v4 Key Changes from v3

- Configuration moves from `tailwind.config.js` to CSS using `@theme` blocks
- Import with `@import "tailwindcss"` instead of `@tailwind` directives
- Custom values use `@theme` instead of `theme.extend`
- Automatic content detection (no `content` array needed)

**Example CSS configuration (v4):**
```css
@import "tailwindcss";

@theme {
  --color-primary: #3b82f6;
  --color-secondary: #10b981;
  --font-sans: "Inter", sans-serif;
}
```

### shadcn + Tailwind CSS v4

shadcn components are built on Tailwind CSS. When you install shadcn components, they come with Tailwind utility classes. shadcn handles the Tailwind integration automatically during `npx shadcn@latest init`.

## Web Vitals Analytics

Next.js provides built-in performance monitoring through the `useReportWebVitals` hook.

### Setup

Create a Web Vitals component:

```tsx
// app/components/web-vitals.tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    // Send to your analytics endpoint
    console.log(metric)

    // Example: send to custom endpoint
    const body = JSON.stringify({
      name: metric.name,
      value: metric.value,
      rating: metric.rating,
      delta: metric.delta,
      id: metric.id,
    })

    // Use sendBeacon for reliability
    if (navigator.sendBeacon) {
      navigator.sendBeacon('/api/analytics', body)
    } else {
      fetch('/api/analytics', { body, method: 'POST', keepalive: true })
    }
  })

  return null
}
```

Add it to your root layout:

```tsx
// app/layout.tsx
import { WebVitals } from './components/web-vitals'

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <WebVitals />
        {children}
      </body>
    </html>
  )
}
```

### Metrics Tracked

| Metric | What It Measures | Good Threshold |
|--------|-----------------|----------------|
| LCP | Largest Contentful Paint — loading performance | < 2.5s |
| FID | First Input Delay — interactivity | < 100ms |
| CLS | Cumulative Layout Shift — visual stability | < 0.1 |
| FCP | First Contentful Paint — first render | < 1.8s |
| TTFB | Time to First Byte — server response | < 800ms |
| INP | Interaction to Next Paint — responsiveness | < 200ms |

## Project Structure

Follow the Next.js App Router conventions:

```
my-app/
├── app/
│   ├── layout.tsx              # Root layout
│   ├── page.tsx                # Home page
│   ├── globals.css             # Global styles + Tailwind imports
│   ├── components/
│   │   └── web-vitals.tsx      # Web Vitals reporter
│   ├── api/
│   │   └── analytics/
│   │       └── route.ts        # Analytics API endpoint
│   ├── (marketing)/            # Route group
│   │   ├── about/
│   │   │   └── page.tsx
│   │   └── pricing/
│   │       └── page.tsx
│   └── dashboard/
│       ├── layout.tsx          # Dashboard layout
│       └── page.tsx
├── components/
│   └── ui/                     # shadcn components installed here
│       ├── button.tsx
│       ├── card.tsx
│       └── dialog.tsx
├── lib/
│   └── utils.ts                # Utility functions (cn helper)
├── public/                     # Static assets
├── e2e/                        # Playwright E2E tests
│   └── home.spec.ts
├── playwright.config.ts        # Playwright configuration
├── components.json             # shadcn configuration
├── next.config.ts              # Next.js configuration
├── package.json
└── tsconfig.json
```

### Key Conventions

- **App Router** — Use `app/` directory with file-based routing
- **Route Groups** — Use `(groupName)/` for logical grouping without affecting URL
- **Layouts** — Use `layout.tsx` for shared UI across routes
- **Loading States** — Use `loading.tsx` for Suspense boundaries
- **Error Handling** — Use `error.tsx` for error boundaries
- **Server Components** — Components are Server Components by default; add `'use client'` only when needed
- **shadcn components** — Installed to `components/ui/` by default
- **E2E tests** — Keep in `e2e/` directory at project root

## Troubleshooting

### next-devtools-mcp not connecting

1. Verify you're running Next.js 16+: check `package.json` for `"next": "^16"`
2. Ensure dev server is running: `npm run dev`
3. Check the MCP endpoint is accessible: visit `http://localhost:3000/_next/mcp`
4. Restart the dev server if it was already running before MCP was configured
5. Call the `init` tool to re-establish the connection

### shadcn components not installing

1. Verify `components.json` exists in project root
2. Run `npx shadcn@latest init` if not yet initialized
3. Check that Tailwind CSS is properly configured
4. For private registries, ensure auth tokens are set in `.env.local`

### Playwright tests failing to start

1. Install browsers: `npx playwright install`
2. Verify `playwright.config.ts` exists
3. Ensure the dev server is running before tests (or configure `webServer` in playwright config)
4. Check Node.js version is 18+

### Tailwind CSS v4 styles not applying

1. Verify your CSS file has `@import "tailwindcss"`
2. Check that `postcss.config.js` includes `@tailwindcss/postcss` (if using PostCSS)
3. For Vite-based setups, use `@tailwindcss/vite` plugin instead
4. Clear `.next` cache: `rm -rf .next` and restart dev server

## Design Philosophy

This power encourages modern, clean design patterns:

- **Minimalist UI** — Subtle shadows (`shadow-sm`, `shadow-md`), ample whitespace, clean typography. Avoid heavy drop shadows or cluttered layouts.
- **Purposeful color** — Neutral base palette with strategic color accents for CTAs and emphasis. Don't color everything.
- **Micro-interactions** — Hover states (`hover:shadow-lg transition-shadow`), smooth transitions (`transition-all duration-200`), loading skeletons during data fetches.
- **Mobile-first responsive** — Start with mobile layout, scale up with `md:` and `lg:` breakpoints. Use `max-w-7xl mx-auto` for content containers.
- **Accessibility by default** — Semantic HTML, visible focus states, ARIA labels, 4.5:1 contrast ratios. See the accessibility steering file for details.
- **Dark mode support** — Use Tailwind's `dark:` prefix. shadcn components support dark mode out of the box.

## Best Practices

- Call the `init` tool at the start of every Next.js development session
- Use Server Components by default; only add `'use client'` when you need browser APIs or interactivity
- Install shadcn components as needed rather than all at once — they're copied into your project, not imported from a package
- Write Playwright E2E tests for critical user flows (login, checkout, form submission)
- Monitor Web Vitals in production to catch performance regressions early
- Use route groups `(groupName)` to organize related pages without affecting URLs
- Keep API routes in `app/api/` for backend logic
- Use `loading.tsx` and `error.tsx` files for better UX during loading and error states
- Prefer Tailwind CSS v4's `@theme` directive over `tailwind.config.js` for customization

---

**MCP Servers:** next-devtools, shadcn, playwright
