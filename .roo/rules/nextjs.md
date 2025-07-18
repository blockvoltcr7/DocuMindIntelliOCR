---
description: 
globs: 
alwaysApply: true
---
# Roo Code Rules for Next.js Health Coach AI Project

You are an expert Next.js 15.2.4 and React developer working on the Health Coach AI platform. Follow these rules strictly:

## Core Architecture Rules

### Next.js App Router (REQUIRED)
- Use App Router exclusively (`app/` directory structure)
- Server Components by default, Client Components only when necessary
- Use `'use client'` directive ONLY when component needs:
  - Browser APIs (window, document, localStorage)
  - Event handlers (onClick, onChange, etc.)
  - React hooks (useState, useEffect, useContext, etc.)
  - Third-party libraries requiring client-side execution

### TypeScript Standards
- Use TypeScript for ALL files (.ts, .tsx)
- Define interfaces for all props, state, and API responses
- Use proper typing for Supabase database schemas
- Prefer `type` over `interface` for simple object shapes
- Use `interface` for extensible object definitions

## File Structure & Naming

### Directory Structure example
```
app/
├── (auth)/
│   ├── login/
│   └── signup/
├── dashboard/
├── api/
├── globals.css
└── layout.tsx

components/
├── ui/              # shadcn/ui components
├── auth/            # Authentication components
└── dashboard/       # Dashboard-specific components

lib/
├── supabase/        # Supabase client configuration
├── types/           # TypeScript type definitions
├── utils/           # Utility functions
└── validation/      # Zod schemas
```

### Naming Conventions
- Components: PascalCase (`UserProfile.tsx`)
- Files/folders: kebab-case (`user-profile.tsx`)
- Variables/functions: camelCase (`getUserProfile`)
- Constants: SCREAMING_SNAKE_CASE (`API_BASE_URL`)

## React & Component Rules

### Component Structure
```tsx
'use client' // Only if client-side features needed

import { useState } from 'react' // React imports first
import { Button } from '@/components/ui/button' // UI components
import { createClient } from '@/lib/supabase/client' // Internal imports
import type { ComponentProps } from '@/lib/types' // Type imports last

interface Props {
  // Define props interface
}

export function ComponentName({ prop1, prop2 }: Props) {
  // Component logic
  
  return (
    <div>
      {/* JSX */}
    </div>
  )
}
```

### React Context Rules
- **ALWAYS** use `'use client'` directive for context providers
- Create custom hooks for context consumption
- Provide proper TypeScript typing
- Handle null context gracefully

```tsx
'use client'

import { createContext, useContext } from 'react'

interface ContextType {
  // Define context shape
}

const Context = createContext<ContextType | null>(null)

export function useCustomContext() {
  const context = useContext(Context)
  if (!context) {
    throw new Error('useCustomContext must be used within Provider')
  }
  return context
}

export function ContextProvider({ children }: { children: React.ReactNode }) {
  return <Context.Provider value={{}}>{children}</Context.Provider>
}
```

### State Management
- Use `useState` for local component state
- Use React Context for cross-component state
- Prefer server state over client state when possible
- Use Supabase real-time subscriptions for live data

## Supabase Integration Rules

### Database Operations
- **NEVER** use database functions - use SSR Supabase client only
- Always use Row Level Security (RLS) policies
- Use Server Actions for mutations
- Use Server Components for data fetching when possible

### Authentication
```tsx
// Server Component - check auth
import { createClient } from '@/lib/supabase/server'

export default async function Page() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    redirect('/login')
  }
  
  return <div>Protected content</div>
}
```

### Client-side Supabase
```tsx
'use client'

import { createClient } from '@/lib/supabase/client'

export function ClientComponent() {
  const supabase = createClient()
  // Client-side operations
}
```

## Data Fetching & Server Actions

### Server Actions Pattern
```tsx
'use server'

import { createClient } from '@/lib/supabase/server'
import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createUser(formData: FormData) {
  const supabase = await createClient()
  
  // Extract and validate data
  const email = formData.get('email') as string
  
  // Perform operation
  const { error } = await supabase.from('users').insert({ email })
  
  if (error) {
    return { error: error.message }
  }
  
  revalidatePath('/dashboard')
  redirect('/dashboard')
}
```

### Error Handling
- Always handle Supabase errors gracefully
- Use try/catch blocks for async operations
- Provide user-friendly error messages
- Log detailed errors for debugging

## Styling & UI

### Tailwind CSS
- Use Tailwind utility classes exclusively
- Follow mobile-first responsive design
- Use consistent spacing scale (4, 8, 12, 16, 24, 32, 48, 64)
- Prefer semantic color names (`text-slate-600` over `text-gray-600`)

### shadcn/ui Components
- Use shadcn/ui components as base building blocks
- Customize via Tailwind classes, not CSS files
- Follow the existing component patterns in the project

### Design Tokens
```tsx
// Consistent color palette
const colors = {
  primary: 'teal', // Health Coach AI brand color
  neutral: 'slate',
  success: 'green',
  warning: 'yellow',
  error: 'red'
}
```

## Form Handling

### Form Validation
- Use Zod for schema validation
- Validate on both client and server
- Use `react-hook-form` for complex forms
- Provide real-time validation feedback

```tsx
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8)
})

type FormData = z.infer<typeof schema>
```

### Form Submission
- Use Server Actions for form handling
- Handle loading states properly
- Show validation errors clearly
- Prevent double submissions

## Performance Rules

### Code Splitting
- Use dynamic imports for large components
- Implement proper loading states
- Use React.Suspense for async components

### Optimization
- Minimize client-side JavaScript
- Use Server Components when possible
- Implement proper caching strategies
- Optimize images with next/image

## Security Rules

### Authentication & Authorization
- Always validate user permissions
- Use middleware for route protection
- Implement proper session management
- Never expose sensitive data in client code

### Data Protection
- Sanitize user inputs
- Use parameterized queries
- Implement rate limiting
- Follow GDPR compliance for EU users

## Testing Guidelines

### Component Testing
- Test user interactions, not implementation details
- Use realistic test data
- Mock external dependencies properly
- Test error states and edge cases

### Integration Testing
- Test complete user flows
- Verify database operations
- Test authentication flows
- Validate form submissions

## AI Memory Integration

### mem0 + Graphiti Setup
- Use mem0 for conversational memory
- Implement Graphiti for temporal knowledge graphs
- Store conversation context properly
- Handle memory retrieval efficiently

### Context Management
```tsx
interface MemoryContext {
  conversationId: string
  userId: string
  memories: Memory[]
}
```

## Error Boundaries & Monitoring

### Error Handling
```tsx
'use client'

import { ErrorBoundary } from 'react-error-boundary'

function ErrorFallback({ error }: { error: Error }) {
  return (
    <div role="alert">
      <h2>Something went wrong:</h2>
      <pre>{error.message}</pre>
    </div>
  )
}

export function AppErrorBoundary({ children }: { children: React.ReactNode }) {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      {children}
    </ErrorBoundary>
  )
}
```

## Code Quality Rules

### Comments & Documentation
- Write self-documenting code
- Add comments for complex business logic
- Document component props and usage
- Keep README files updated

### Code Organization
- Keep components under 200 lines
- Extract custom hooks for reusable logic
- Use consistent import ordering
- Remove unused imports and variables

### Git Practices
- Use conventional commit messages
- Create feature branches for new work
- Keep commits focused and atomic
- Write descriptive commit messages

## Deployment & Environment

### Environment Variables
- Use `.env.local` for local development
- Never commit sensitive environment variables
- Use Vercel environment variables for production
- Validate environment variables at startup


---

**Remember**: This is a health coaching platform where user data privacy and security are paramount. Always prioritize user experience, data protection, and code maintainability.