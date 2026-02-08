# AI UI Development with Vercel AI SDK

Guide for building AI-powered user interfaces in Next.js using the Vercel AI SDK UI hooks. This covers chat interfaces, text completions, streamed object generation, and tool-calling patterns.

Reference: [AI SDK UI Documentation](https://ai-sdk.dev/docs/ai-sdk-ui/overview)

## Installation

```bash
npm install ai @ai-sdk/react
```

You'll also need at least one AI provider. Example with Anthropic:

```bash
npm install @ai-sdk/anthropic
```

Set your API key in `.env.local`:

```
ANTHROPIC_API_KEY=your_key_here
```

## Three Core Hooks

The AI SDK UI provides three hooks for different use cases:

| Hook | Use Case | What It Streams |
|------|----------|-----------------|
| `useChat` | Conversational chat interfaces | Messages back and forth |
| `useCompletion` | Single text completions | Text output from a prompt |
| `useObject` | Structured data generation | Typed JSON objects |

## useChat — Chat Interfaces

The primary hook for building chatbots and conversational UIs.

### Basic Chat

Client component:

```tsx
'use client'

import { useChat } from '@ai-sdk/react'
import { DefaultChatTransport } from 'ai'
import { useState } from 'react'

export default function Chat() {
  const { messages, sendMessage, status } = useChat({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  })

  const [input, setInput] = useState('')

  return (
    <>
      {messages.map((message) => (
        <div key={message.id}>
          {message.role === 'user' ? 'User: ' : 'AI: '}
          {message.parts.map((part, index) =>
            part.type === 'text' ? <span key={index}>{part.text}</span> : null
          )}
        </div>
      ))}

      <form
        onSubmit={(e) => {
          e.preventDefault()
          if (input.trim()) {
            sendMessage({ text: input })
            setInput('')
          }
        }}
      >
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          disabled={status !== 'ready'}
          placeholder="Say something..."
        />
        <button type="submit" disabled={status !== 'ready'}>
          Send
        </button>
      </form>
    </>
  )
}
```

API route:

```typescript
// app/api/chat/route.ts
import { convertToModelMessages, streamText, UIMessage } from 'ai'

export const maxDuration = 30

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json()

  const result = streamText({
    model: 'anthropic/claude-sonnet-4.5',
    system: 'You are a helpful assistant.',
    messages: await convertToModelMessages(messages),
  })

  return result.toUIMessageStreamResponse()
}
```

### useChat Status Values

- `submitted` — Message sent, awaiting response stream
- `streaming` — Response actively streaming in
- `ready` — Response complete, can submit new message
- `error` — Request failed

### useChat Key Features

- `messages` — Array of all chat messages
- `sendMessage({ text })` — Send a new user message
- `status` — Current chat status
- `stop()` — Abort the current streaming response
- `reload()` — Regenerate the last assistant message
- `error` — Error object if request failed

## useCompletion — Text Completions

For single-prompt, single-response text generation (not conversational).

```tsx
'use client'

import { useCompletion } from '@ai-sdk/react'

export default function Completion() {
  const { completion, input, handleInputChange, handleSubmit, isLoading } =
    useCompletion({
      api: '/api/completion',
    })

  return (
    <form onSubmit={handleSubmit}>
      <input value={input} onChange={handleInputChange} />
      <button type="submit" disabled={isLoading}>
        Submit
      </button>
      <div>{completion}</div>
    </form>
  )
}
```

API route:

```typescript
// app/api/completion/route.ts
import { streamText } from 'ai'

export const maxDuration = 30

export async function POST(req: Request) {
  const { prompt }: { prompt: string } = await req.json()

  const result = streamText({
    model: 'anthropic/claude-sonnet-4.5',
    prompt,
  })

  return result.toUIMessageStreamResponse()
}
```

### useCompletion Key Features

- `completion` — The streamed text result
- `input` / `setInput` — Managed input state
- `handleSubmit` / `handleInputChange` — Form helpers
- `isLoading` — Whether a request is in progress
- `stop()` — Abort the current stream
- `error` — Error object if request failed
- `onFinish(prompt, completion)` — Callback when streaming completes

## useObject — Streamed Structured Data

For generating typed JSON objects that stream in progressively. Useful for generating lists, cards, notifications, or any structured UI data.

### Schema Definition

Define a shared schema (used on both client and server):

```typescript
// app/api/notifications/schema.ts
import { z } from 'zod'

export const notificationSchema = z.object({
  notifications: z.array(
    z.object({
      name: z.string().describe('Name of a fictional person.'),
      message: z.string().describe('Message. Do not use emojis or links.'),
    })
  ),
})
```

### Client

```tsx
'use client'

import { experimental_useObject as useObject } from '@ai-sdk/react'
import { notificationSchema } from './api/notifications/schema'

export default function Notifications() {
  const { object, submit, isLoading } = useObject({
    api: '/api/notifications',
    schema: notificationSchema,
  })

  return (
    <>
      <button onClick={() => submit('Messages during finals week.')}>
        Generate notifications
      </button>

      {object?.notifications?.map((notification, index) => (
        <div key={index}>
          <p>{notification?.name}</p>
          <p>{notification?.message}</p>
        </div>
      ))}
    </>
  )
}
```

### Server

```typescript
// app/api/notifications/route.ts
import { streamText, Output } from 'ai'
import { notificationSchema } from './schema'

export const maxDuration = 30

export async function POST(req: Request) {
  const context = await req.json()

  const result = streamText({
    model: 'anthropic/claude-sonnet-4.5',
    output: Output.object({ schema: notificationSchema }),
    prompt: `Generate 3 notifications for a messages app in this context: ${context}`,
  })

  return result.toTextStreamResponse()
}
```

Note: `useObject` results are partial as they stream in. Always use optional chaining (`?.`) when rendering.

## Tool Calling in Chat

The AI SDK supports three types of tools with `useChat`:

1. **Server-side tools** — Executed automatically on the server via `execute` function
2. **Client-side auto tools** — Executed automatically on the client via `onToolCall`
3. **User interaction tools** — Displayed in the UI for user input (e.g., confirmation dialogs)

### API Route with Tools

```typescript
// app/api/chat/route.ts
import { convertToModelMessages, streamText, UIMessage } from 'ai'
import { z } from 'zod'

export const maxDuration = 30

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json()

  const result = streamText({
    model: 'anthropic/claude-sonnet-4.5',
    messages: await convertToModelMessages(messages),
    tools: {
      // Server-side tool (auto-executed)
      getWeatherInformation: {
        description: 'Show the weather in a given city',
        inputSchema: z.object({ city: z.string() }),
        execute: async ({ city }) => {
          const options = ['sunny', 'cloudy', 'rainy', 'snowy', 'windy']
          return options[Math.floor(Math.random() * options.length)]
        },
      },
      // Client-side tool (requires user interaction)
      askForConfirmation: {
        description: 'Ask the user for confirmation.',
        inputSchema: z.object({
          message: z.string().describe('The message to ask for confirmation.'),
        }),
        // No execute = handled on client
      },
    },
  })

  return result.toUIMessageStreamResponse()
}
```

### Rendering Tool Parts in Chat UI

```tsx
{messages.map((message) => (
  <div key={message.id}>
    {message.parts.map((part, index) => {
      if (part.type === 'text') {
        return <span key={index}>{part.text}</span>
      }
      if (part.type === 'tool-invocation') {
        const { toolName, state, result } = part.toolInvocation
        if (state === 'result') {
          return <div key={index}>Tool {toolName}: {JSON.stringify(result)}</div>
        }
        return <div key={index}>Calling {toolName}...</div>
      }
      return null
    })}
  </div>
))}
```

## Common Patterns

### Throttling UI Updates

Reduce re-renders during fast streaming:

```tsx
const { messages } = useChat({
  experimental_throttle: 50, // Update UI at most every 50ms
})
```

### Custom Request Headers/Body

```tsx
const { messages } = useChat({
  transport: new DefaultChatTransport({
    api: '/api/chat',
    headers: { Authorization: 'Bearer your_token' },
    body: { user_id: '123' },
  }),
})
```

### Error Handling

```tsx
const { error } = useChat({ /* ... */ })

useEffect(() => {
  if (error) {
    toast.error(error.message)
  }
}, [error])
```

### Cancellation

```tsx
const { stop, status } = useChat({ /* ... */ })

return (
  <button onClick={stop} disabled={status !== 'streaming'}>
    Stop generating
  </button>
)
```

## When to Use Which Hook

| Scenario | Hook |
|----------|------|
| Chat / conversational UI | `useChat` |
| Single prompt → text response | `useCompletion` |
| Generate structured data (lists, cards, forms) | `useObject` |
| Chat with tool calling / function execution | `useChat` with tools |
| Classification / enum output | `useObject` with `z.enum` |

## Project Structure for AI Features

```
app/
├── api/
│   ├── chat/
│   │   └── route.ts          # Chat endpoint
│   ├── completion/
│   │   └── route.ts          # Completion endpoint
│   └── notifications/
│       ├── route.ts           # Object generation endpoint
│       └── schema.ts          # Shared Zod schema
├── components/
│   ├── chat.tsx               # Chat UI component
│   ├── completion-form.tsx    # Completion UI component
│   └── notifications.tsx      # Object generation UI
└── page.tsx
```
