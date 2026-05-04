---
name: mobile-ui
description: React Native conventions for screen structure, navigation, SafeAreaView, FlatList, and platform-specific patterns.
---
# mobile-ui.md

## Purpose

Define conventions for building mobile UI using React Native — covering
screen structure, navigation, platform-specific patterns, and styling.

---

## Conventions

### Folder Structure

```
/src
  /screens              # Full screen components
    /auth
      LoginScreen.tsx
      RegisterScreen.tsx
    /dashboard
      DashboardScreen.tsx
  /components
    /ui                 # Generic reusable components (Button, Input, Card)
    /features           # Feature-specific components (OrderItem, UserAvatar)
  /navigation           # Navigation stacks and tab configs
    AppNavigator.tsx
    AuthNavigator.tsx
  /hooks                # Custom hooks
  /services             # API calls, storage, device services
  /store                # Global state (Zustand or Redux)
  /types                # TypeScript type definitions
  /utils                # Helper functions, constants
  /assets               # Images, fonts, icons
```

### Screen Rules

- One screen per file — named with `Screen` suffix: `HomeScreen.tsx`
- Screens handle layout and orchestration only — no raw business logic
- Extract logic to hooks, data fetching to services
- Always handle loading, error, and empty states
- Always use `SafeAreaView` as the root wrapper for screens

### Component Rules

- Functional components only with TypeScript props interfaces
- Keep components under 150 lines — split if larger
- Platform-specific code: use `Platform.OS` or `.ios.tsx` / `.android.tsx` file extensions

### Navigation

- Use React Navigation as the standard navigation library
- Stack navigator for hierarchical flows
- Tab navigator for primary app sections
- Always type navigation props using React Navigation's TypeScript generics
- Never use raw `navigation.navigate()` without typed route params

### Styling

- Use `StyleSheet.create()` for all styles — never inline style objects
- No hardcoded pixel values — use a spacing scale from a constants file
- Use `Dimensions` or `useWindowDimensions` for responsive sizing
- Separate platform styles using `Platform.select()` when needed

### Performance

- Use `FlatList` or `SectionList` for long lists — never `ScrollView` with `.map()`
- Memoize list item components with `React.memo`
- Use `useCallback` for event handlers passed to list items
- Avoid anonymous functions in JSX for performance-critical components

---

## Anti-Patterns

- Never use `ScrollView` to render large lists — use `FlatList`
- Never hardcode platform-specific logic without using `Platform.OS`
- Never put API calls directly in screen components — use services
- Never use inline style objects — use `StyleSheet.create()`
- Never skip `SafeAreaView` on screens — content will clip on notched devices
- Never use untyped navigation params

---

## Ready-to-Use Prompt

```
Task: Build [screen or component name] for [feature description]
Skill: frontend/mobile-ui, frontend/state-management, core/security

REQUIREMENTS:
- Type: [screen / feature component / UI component]
- Platform target: [iOS / Android / both]
- Navigation: [stack / tab / modal]
- Data needed: [describe data shape or API endpoint]
- States to handle: loading, error, empty, success

CONSTRAINTS:
- React Native with TypeScript
- StyleSheet.create() for all styles — no inline objects
- SafeAreaView as root wrapper for screens
- FlatList for any list rendering
- Typed navigation params

IF BLOCKED:
- Document blocker as TODO comment
- Log reasoning for platform-specific decisions

DONE WHEN:
- Screen renders on both iOS and Android without clipping
- All states handled (loading, error, empty, success)
- Navigation typed correctly
- Code review skill applied before commit
```

