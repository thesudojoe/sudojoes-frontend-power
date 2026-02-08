# Next.js Frontend Dev — Kiro Power

A [Kiro](https://kiro.dev) power that streamlines frontend development with Next.js 16+ by bundling three MCP servers, agent hooks, and steering guides into a single workflow.

## What's Included

### MCP Servers

| Server | Purpose |
|--------|---------|
| **next-devtools** | Connects to your running Next.js dev server for runtime diagnostics, error retrieval, and documentation context |
| **shadcn** | Browse, search, and install UI components from shadcn registries via natural language |
| **playwright** | Automate browser interactions for E2E testing and visual verification |

### Steering Files

Contextual guides that load into the agent when relevant:

- **project-setup** — Initialize a new Next.js project with Tailwind CSS v4, shadcn, Playwright, and Web Vitals analytics
- **workflows** — Day-to-day patterns: building pages, adding components, debugging, and testing
- **ai-ui** — Build AI-powered UIs with the Vercel AI SDK (`useChat`, `useCompletion`, `useObject`, tool calling)
- **accessibility** — WCAG AA standards, semantic HTML, ARIA patterns, keyboard navigation, and a component checklist
- **forms** — Form patterns with shadcn, react-hook-form, zod, and Server Actions

### Agent Hooks

| Hook | Trigger | What It Does |
|------|---------|--------------|
| **Next.js DevTools Init** | `promptSubmit` | Auto-calls the `init` tool at session start to load Next.js docs context and connect to the dev server |
| **Reuse Existing Components** | `promptSubmit` | Scans `components/` before creating new ones to avoid duplicates |
| **Suggest Component Tests** | `userTriggered` | Identifies components and pages missing Playwright E2E test coverage |

## Prerequisites

- Node.js v20.19+ (LTS recommended)
- A Next.js 16+ project (or willingness to create one)
- npm or pnpm

## Quick Start

1. Install this power in Kiro.
2. Start your Next.js dev server:
   ```bash
   npm run dev
   ```
3. Send any prompt — the auto-init hook connects to your dev server and loads Next.js documentation context automatically.
4. Start building. Ask the agent to create pages, add shadcn components, check for errors, or run browser tests.

If you're starting a new project from scratch, load the **project-setup** steering file for a full walkthrough.

## Project Structure

The power follows standard Next.js App Router conventions:

```
my-app/
├── app/                    # Pages, layouts, API routes
│   ├── layout.tsx
│   ├── page.tsx
│   ├── globals.css
│   └── components/
├── components/
│   └── ui/                 # shadcn components
├── lib/
│   └── utils.ts            # Utilities (cn helper)
├── e2e/                    # Playwright E2E tests
├── playwright.config.ts
├── components.json         # shadcn config
└── next.config.ts
```

## Key Topics Covered

- **Tailwind CSS v4** — CSS-first configuration with `@theme`, zero-config content detection, and shadcn integration
- **Web Vitals** — Built-in performance monitoring with `useReportWebVitals` (LCP, FID, CLS, FCP, TTFB, INP)
- **Server Components** — Server-first by default, `'use client'` only when needed
- **Server Actions** — Preferred over API routes for form submissions and mutations
- **Accessibility** — Semantic HTML, ARIA labels, keyboard navigation, 4.5:1 contrast ratios

## Troubleshooting

- **next-devtools not connecting** — Ensure Next.js 16+, dev server running, and `/_next/mcp` endpoint accessible
- **shadcn components not installing** — Run `npx shadcn@latest init` if `components.json` is missing
- **Playwright tests failing** — Run `npx playwright install` to install browsers
- **Tailwind v4 styles not applying** — Verify `@import "tailwindcss"` in your CSS file and clear `.next` cache

## License

ISC
