# Project Setup Guide

Step-by-step guide for initializing a new Next.js project with all the tools configured.

## 1. Create Next.js App

```bash
npx create-next-app@latest my-app --typescript --tailwind --eslint --app --src-dir=false --import-alias="@/*"
cd my-app
```

This creates a Next.js 16+ project with:
- TypeScript
- Tailwind CSS v4
- ESLint
- App Router
- `@/*` import alias

## 2. Initialize shadcn

```bash
npx shadcn@latest init
```

Follow the prompts. This will:
- Create `components.json` configuration
- Set up the `components/ui/` directory
- Configure path aliases
- Add the `cn()` utility to `lib/utils.ts`

Install some starter components:

```bash
npx shadcn@latest add button card input label
```

## 3. Set Up Playwright

Install Playwright:

```bash
npm init playwright@latest
```

When prompted:
- Choose TypeScript
- Put tests in `e2e/` directory
- Add a GitHub Actions workflow (optional)
- Install Playwright browsers

Update `playwright.config.ts` to work with your Next.js dev server:

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
})
```

Create a basic test:

```typescript
// e2e/home.spec.ts
import { test, expect } from '@playwright/test'

test('homepage loads correctly', async ({ page }) => {
  await page.goto('/')
  await expect(page).toHaveTitle(/Next/)
})
```

## 4. Add Web Vitals Analytics

Create the Web Vitals component:

```tsx
// app/components/web-vitals.tsx
'use client'

import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    // Log in development
    if (process.env.NODE_ENV === 'development') {
      console.log(metric)
      return
    }

    // Send to analytics in production
    const body = JSON.stringify({
      name: metric.name,
      value: metric.value,
      rating: metric.rating,
      delta: metric.delta,
      id: metric.id,
      navigationType: metric.navigationType,
    })

    if (navigator.sendBeacon) {
      navigator.sendBeacon('/api/analytics', body)
    } else {
      fetch('/api/analytics', { body, method: 'POST', keepalive: true })
    }
  })

  return null
}
```

Create the analytics API route:

```typescript
// app/api/analytics/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const metric = await request.json()

  // Store or forward metrics as needed
  // Example: log to console, send to external service, write to database
  console.log('[Web Vital]', metric.name, metric.value, metric.rating)

  return NextResponse.json({ received: true })
}
```

Add to your root layout:

```tsx
// app/layout.tsx
import type { Metadata } from 'next'
import { WebVitals } from './components/web-vitals'
import './globals.css'

export const metadata: Metadata = {
  title: 'My App',
  description: 'Built with Next.js',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
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

## 5. Configure Tailwind CSS v4

Your `globals.css` should look like:

```css
@import "tailwindcss";

@theme {
  --color-background: #ffffff;
  --color-foreground: #0a0a0a;
  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
}

@media (prefers-color-scheme: dark) {
  @theme {
    --color-background: #0a0a0a;
    --color-foreground: #ededed;
  }
}
```

Note: shadcn's `init` command will configure Tailwind CSS variables for you. The above is a minimal example if you need to customize further.

## 6. Verify Setup

1. Start the dev server: `npm run dev`
2. Call the `init` tool from next-devtools MCP
3. Ask the agent: "What errors are in my Next.js application?"
4. Ask the agent: "Add the dialog component from shadcn"
5. Run Playwright tests: `npx playwright test`

If all five steps work, your project is fully configured.

## Final Project Structure

```
my-app/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── globals.css
│   ├── components/
│   │   └── web-vitals.tsx
│   └── api/
│       └── analytics/
│           └── route.ts
├── components/
│   └── ui/
│       ├── button.tsx
│       ├── card.tsx
│       ├── input.tsx
│       └── label.tsx
├── lib/
│   └── utils.ts
├── e2e/
│   └── home.spec.ts
├── public/
├── playwright.config.ts
├── components.json
├── next.config.ts
├── package.json
└── tsconfig.json
```
