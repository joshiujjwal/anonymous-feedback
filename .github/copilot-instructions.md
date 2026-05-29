# GitHub Copilot Instructions — AnonymousFeedback

## Project Summary

Anonymous feedback app where users open themselves to honest feedback from trusted connections. Feed shows exactly **3 positive + 1 negative** pieces of feedback (Instagram card style). Two codebases: `src/` (Express API, TypeScript) and `mobile/` (React Native + Expo, TypeScript).

## Stack

- **API**: Node.js 20+, Express 4, TypeScript 5, Supabase (PostgreSQL + Auth + Realtime)
- **Mobile**: React Native 0.73+, Expo SDK 50+, Zustand, React Navigation v6
- **Testing**: Jest, Supertest (API), React Native Testing Library (mobile), Detox (e2e)
- **CI**: GitHub Actions

## Coding Conventions

### General
- TypeScript strict mode — no `any`, no untyped returns on async functions
- Named exports only, no default exports
- No `console.log` in production code — use a logger utility
- All errors go through the `AppError` typed class

### API
- Route handlers: validate → service → respond (3 lines max, delegate everything)
- DB access only in `src/models/` — no Supabase queries in routes or services
- Services are pure TypeScript logic that call models
- Return `{ data, error }` tuples from model functions (mirrors Supabase client pattern)

### Mobile
- Hooks-first: all data fetching in custom hooks under `mobile/hooks/`
- Zustand store slices in `mobile/store/` — one file per domain
- `StyleSheet.create()` always — no inline style objects
- Accessibility: every interactive element needs `accessibilityLabel`
- Use `react-native-safe-area-context` on all root screen views

## Testing Conventions

- **Write tests before implementation** — this is enforced, not optional
- Unit tests for all service functions (especially `curateFeed`)
- Integration tests use Supertest against a real Express instance with mocked Supabase
- Mobile component tests use React Native Testing Library
- Snapshot tests are discouraged — prefer behavioral assertions

## Boundaries

- **Do not refactor** files outside the scope of the current task unless explicitly asked
- **Do not remove** any existing passing test
- **Do not bypass** Supabase RLS — never use the service role key in application request handlers
- **Do not expose** `submitter_id` in any API response (anonymity invariant)
- **Do not add** new dependencies without checking if the existing stack already covers the need
- **Do not generate** placeholder TODO comments in implementation code — either implement it or leave it for a future task in TODO.md
