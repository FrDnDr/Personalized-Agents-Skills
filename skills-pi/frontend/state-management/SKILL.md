---
name: state-management
description: State decision tree for choosing between useState, Zustand, React Query, and React Hook Form based on state type.
---
# state-management.md

## Purpose

Define when to use local vs global state, which libraries to use,
and how to structure state across web and mobile frontends.

---

## Conventions

### State Decision Tree

Before reaching for a state library, answer these questions:

```
1. Is this state used by only one component?
   → Use useState or useReducer locally. Stop here.

2. Is this state shared between a parent and a few children?
   → Pass as props or use useContext. Stop here.

3. Is this state shared across many unrelated components?
   → Use Zustand (preferred) or Redux Toolkit.

4. Is this state server data (API responses, cached data)?
   → Use React Query (web) or SWR (web) or React Query (mobile).
   → Never duplicate server state into global store.

5. Is this form state?
   → Use React Hook Form. Never manage form state manually.
```

### Local State — `useState` / `useReducer`

- Default choice for component-scoped state
- Use `useReducer` when state has multiple sub-values or complex transitions
- Never lift state higher than necessary

### Global State — Zustand (Preferred)

- Use for: auth state, user preferences, UI state shared across routes
- Never store server/API data in Zustand — that belongs in React Query
- One store file per domain: `useAuthStore.ts`, `useCartStore.ts`
- Always define TypeScript types for the store shape
- Keep store actions inside the store definition — not in components

```typescript
// Example Zustand store
import { create } from "zustand";

interface AuthStore {
  user: User | null;
  token: string | null;
  setUser: (user: User) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  token: null,
  setUser: (user) => set({ user }),
  logout: () => set({ user: null, token: null }),
}));
```

### Server State — React Query

- Use for all API data: fetching, caching, refetching, mutations
- Always define query keys as constants in a separate file
- Always handle `isLoading`, `isError`, and `data` states in UI
- Use `useMutation` for POST/PUT/PATCH/DELETE operations
- Invalidate related queries after mutations

```typescript
// Query keys file
export const queryKeys = {
  users: ["users"] as const,
  user: (id: string) => ["users", id] as const,
  orders: ["orders"] as const,
};
```

### Form State — React Hook Form

- Use for all forms — never manage form state with useState manually
- Always define a Zod schema for form validation
- Use `zodResolver` to connect Zod schema to React Hook Form

---

## Anti-Patterns

- Never store API response data in Zustand or Redux — use React Query
- Never use Context for frequently updated state — it causes unnecessary re-renders
- Never manage form state manually with useState
- Never put derived state in the store — compute it with selectors instead
- Never use Redux for new projects unless the team already uses it
- Never skip TypeScript types for store shape

---

## Ready-to-Use Prompt

```
Task: Implement state management for [feature name]
Skill: frontend/state-management, frontend/web-ui or frontend/mobile-ui

REQUIREMENTS:
- State type: [local / global UI state / server data / form]
- Shared across: [single component / parent-child / multiple routes]
- Data source: [local only / API endpoint]

DECISION:
- Apply state decision tree from state-management.md
- Select: useState / useReducer / Zustand / React Query / React Hook Form

CONSTRAINTS:
- Never store server data in Zustand
- Never manage form state with useState
- Always type store shape with TypeScript
- Query keys must be defined as constants

DONE WHEN:
- State is managed at the correct level
- TypeScript types defined
- Loading, error, and empty states handled for server data
- Code review skill applied before commit
```

