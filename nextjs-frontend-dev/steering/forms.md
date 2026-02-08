# Form Patterns

Guide for building forms in Next.js with shadcn, react-hook-form, zod validation, and Server Actions.

## Stack

- **shadcn Form component** — Wraps react-hook-form with accessible, styled form fields
- **react-hook-form** — Form state management with minimal re-renders
- **zod** — Schema validation shared between client and server
- **Server Actions** — Preferred over API routes for form submissions

## Installation

```bash
npx shadcn@latest add form input label button textarea select
npm install zod
```

shadcn's form component includes react-hook-form as a dependency.

## Basic Form with Validation

### 1. Define the Schema

```typescript
// lib/schemas/contact.ts
import { z } from 'zod'

export const contactSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Please enter a valid email'),
  message: z.string().min(10, 'Message must be at least 10 characters'),
})

export type ContactFormData = z.infer<typeof contactSchema>
```

### 2. Create the Server Action

```typescript
// app/actions/contact.ts
'use server'

import { contactSchema, ContactFormData } from '@/lib/schemas/contact'

export async function submitContact(data: ContactFormData) {
  // Validate on server (never trust client-only validation)
  const parsed = contactSchema.safeParse(data)

  if (!parsed.success) {
    return { success: false, error: 'Invalid form data' }
  }

  // Process the form (save to DB, send email, etc.)
  // await db.contacts.create({ data: parsed.data })

  return { success: true }
}
```

### 3. Build the Form Component

```tsx
// app/components/contact-form.tsx
'use client'

import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { contactSchema, ContactFormData } from '@/lib/schemas/contact'
import { submitContact } from '@/app/actions/contact'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form'
import { useState } from 'react'

export function ContactForm() {
  const [submitted, setSubmitted] = useState(false)

  const form = useForm<ContactFormData>({
    resolver: zodResolver(contactSchema),
    defaultValues: {
      name: '',
      email: '',
      message: '',
    },
  })

  async function onSubmit(data: ContactFormData) {
    const result = await submitContact(data)
    if (result.success) {
      setSubmitted(true)
      form.reset()
    }
  }

  if (submitted) {
    return <p>Thanks for reaching out. We'll get back to you soon.</p>
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input placeholder="Your name" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input type="email" placeholder="you@example.com" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="message"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Message</FormLabel>
              <FormControl>
                <Textarea placeholder="Your message..." {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit" disabled={form.formState.isSubmitting}>
          {form.formState.isSubmitting ? 'Sending...' : 'Send message'}
        </Button>
      </form>
    </Form>
  )
}
```

## Server Actions vs API Routes

Prefer Server Actions for form submissions:

| | Server Actions | API Routes |
|---|---|---|
| Boilerplate | Minimal — just `'use server'` | Need route file, request parsing |
| Type safety | End-to-end with zod | Manual type alignment |
| Progressive enhancement | Works without JS | Requires JS |
| Use when | Form submissions, mutations | External API consumers, webhooks |

```typescript
// Server Action — preferred for forms
'use server'
export async function createPost(data: PostFormData) {
  const parsed = postSchema.safeParse(data)
  if (!parsed.success) return { error: 'Invalid data' }
  await db.posts.create({ data: parsed.data })
  revalidatePath('/posts')
  return { success: true }
}

// API Route — use when you need an HTTP endpoint
// app/api/posts/route.ts
export async function POST(req: Request) {
  const data = await req.json()
  // ...
}
```

## Validation Patterns

### Real-time Validation

Use `mode: 'onChange'` for instant feedback:

```tsx
const form = useForm<FormData>({
  resolver: zodResolver(schema),
  mode: 'onChange', // Validate on every change
})
```

### Debounced Validation

For expensive checks (e.g., username availability):

```tsx
const form = useForm<FormData>({
  resolver: zodResolver(schema),
  mode: 'onChange',
  delayError: 500, // Wait 500ms before showing errors
})
```

### Server-side Validation Errors

Return field-specific errors from Server Actions:

```typescript
'use server'
export async function register(data: RegisterData) {
  const parsed = registerSchema.safeParse(data)
  if (!parsed.success) {
    return { success: false, errors: parsed.error.flatten().fieldErrors }
  }

  const existing = await db.users.findByEmail(parsed.data.email)
  if (existing) {
    return { success: false, errors: { email: ['Email already registered'] } }
  }

  // Create user...
  return { success: true }
}
```

Display server errors on the client:

```tsx
async function onSubmit(data: RegisterData) {
  const result = await register(data)
  if (!result.success && result.errors) {
    Object.entries(result.errors).forEach(([field, messages]) => {
      form.setError(field as keyof RegisterData, {
        message: messages?.[0],
      })
    })
  }
}
```

## Common Form Types

### Authentication (Login)

```typescript
const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8, 'Password must be at least 8 characters'),
})
```

### Search with Debounce

```tsx
'use client'

import { useSearchParams, useRouter, usePathname } from 'next/navigation'
import { useDebouncedCallback } from 'use-debounce'
import { Input } from '@/components/ui/input'

export function SearchForm() {
  const searchParams = useSearchParams()
  const pathname = usePathname()
  const { replace } = useRouter()

  const handleSearch = useDebouncedCallback((term: string) => {
    const params = new URLSearchParams(searchParams)
    if (term) {
      params.set('q', term)
    } else {
      params.delete('q')
    }
    replace(`${pathname}?${params.toString()}`)
  }, 300)

  return (
    <Input
      placeholder="Search..."
      defaultValue={searchParams.get('q') ?? ''}
      onChange={(e) => handleSearch(e.target.value)}
    />
  )
}
```

### Multi-step Form

```tsx
'use client'

import { useState } from 'react'
import { Button } from '@/components/ui/button'

const steps = ['Account', 'Profile', 'Review']

export function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(0)

  return (
    <div>
      {/* Progress indicator */}
      <div className="flex gap-2 mb-6">
        {steps.map((step, index) => (
          <div
            key={step}
            className={`flex-1 h-2 rounded-full ${
              index <= currentStep ? 'bg-blue-500' : 'bg-gray-200'
            }`}
          />
        ))}
      </div>

      {/* Step content */}
      {currentStep === 0 && <AccountStep />}
      {currentStep === 1 && <ProfileStep />}
      {currentStep === 2 && <ReviewStep />}

      {/* Navigation */}
      <div className="flex justify-between mt-6">
        <Button
          variant="outline"
          onClick={() => setCurrentStep((s) => s - 1)}
          disabled={currentStep === 0}
        >
          Back
        </Button>
        <Button
          onClick={() => {
            if (currentStep < steps.length - 1) {
              setCurrentStep((s) => s + 1)
            } else {
              // Submit
            }
          }}
        >
          {currentStep === steps.length - 1 ? 'Submit' : 'Next'}
        </Button>
      </div>
    </div>
  )
}
```

## Tips

- Always validate on both client and server — share the zod schema
- Use `revalidatePath()` or `revalidateTag()` after mutations to refresh data
- Show loading state on submit buttons (`form.formState.isSubmitting`)
- Use `toast` from shadcn for success/error notifications after submission
- For file uploads, use `FormData` with Server Actions instead of JSON
