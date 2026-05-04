---
name: web-ui
description: React and Next.js App Router conventions including component structure, Tailwind styling, and data fetching patterns.
---
# web-ui.md

## Purpose

Define conventions for building web UI using React and Next.js — covering
component structure, routing patterns, styling, and file organization.

---

## Conventions

### Folder Structure

```
/src
  /app                  # Next.js app router pages
    /dashboard
      page.tsx
      layout.tsx
    /api                # API routes (Next.js)
  /components
    /ui                 # Generic reusable components (Button, Input, Modal)
    /features           # Feature-specific components (UserCard, OrderTable)
    /layouts            # Page layout wrappers (Sidebar, Header, Footer)
  /hooks                # Custom React hooks
  /lib                  # Utility functions, constants, config
  /types                # TypeScript type definitions
  /styles               # Global styles, theme tokens
```

### Component Rules

- One component per file — filename matches component name
- Use functional components only — no class components
- Always type props with TypeScript interfaces
- Keep components small — if it exceeds 150 lines, split it
- Separate UI components from logic — use custom hooks for logic
- Always provide a default export for page components
- Named exports for reusable UI components

### Naming Conventions

- Components: `PascalCase` — `UserCard.tsx`
- Hooks: `camelCase` prefixed with `use` — `useOrderData.ts`
- Utilities: `camelCase` — `formatCurrency.ts`
- Types: `PascalCase` with suffix — `UserProps`, `OrderType`
- Pages (Next.js): `page.tsx` inside named folder

### Routing (Next.js App Router)

- Use App Router — not Pages Router
- Dynamic routes: `/app/orders/[id]/page.tsx`
- Layouts wrap shared UI per route segment
- Loading states: `loading.tsx` per route segment
- Error boundaries: `error.tsx` per route segment
- Always use `next/link` for internal navigation — never `<a>` tags

### Styling

- Use Tailwind CSS utility classes as primary styling method
- No inline styles unless dynamically computed
- Use `cn()` utility for conditional class merging
- Global styles only for base resets and theme tokens
- Mobile-first responsive design — always start with small screen styles

### Data Fetching

- Server components fetch data directly — no useEffect for initial data
- Client components use SWR or React Query for client-side fetching
- Never fetch data inside a component render without caching
- Always handle loading, error, and empty states explicitly

### Performance

- Use `next/image` for all images — never raw `<img>` tags
- Use `next/font` for fonts
- Lazy load heavy components with `dynamic()` import
- Memoize expensive computations with `useMemo`
- Memoize stable callbacks with `useCallback`

---

## Anti-Patterns

- Never put business logic inside JSX — extract to hooks or utils
- Never use `any` type in TypeScript
- Never use inline styles for static values
- Never fetch data with `useEffect` on mount for initial page data in Next.js
- Never create deeply nested component trees — flatten with composition
- Never hardcode colors, spacing, or breakpoints — use Tailwind tokens
- Never use `<a>` for internal links — use `next/link`

---

## Ready-to-Use Prompt

```
Task: Build [component or page name] for [feature description]
Skill: frontend/web-ui, core/api-design, core/security

REQUIREMENTS:
- Component type: [page / feature component / UI component / layout]
- Data needed: [describe data shape or API endpoint]
- States to handle: loading, error, empty, success
- Responsive: mobile-first

CONSTRAINTS:
- Use Next.js App Router conventions
- Functional components with TypeScript
- Tailwind CSS for styling — no inline styles
- Server component if data fetching, client component if interactive
- Follow folder structure from web-ui.md
- No business logic in JSX — extract to hooks

IF BLOCKED:
- Document blocker as TODO comment
- Log reasoning for non-obvious decisions

DONE WHEN:
- Component renders correctly with all states handled
- TypeScript types defined for all props
- Follows folder and naming conventions
- Code review skill applied before commit
```

