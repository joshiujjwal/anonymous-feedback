# AGENTS.md — AnonymousFeedback

## Setup

```bash
# 1. Install API dependencies
cd src && npm install

# 2. Install mobile dependencies
cd ../mobile && npm install

# 3. Start local Supabase (requires Docker)
supabase start

# 4. Copy env and fill in local values from `supabase status`
cp .env.example .env

# 5. Run DB migrations
cd src && npm run db:migrate

# 6. Seed test data
npm run db:seed

# 7. Verify everything works
npm test
```

## Running Tests

```bash
cd src && npm test                  # all tests
cd src && npm run test:unit         # unit only (fast, no DB)
cd src && npm run test:int          # integration (needs supabase running)
cd mobile && npm test               # mobile component tests
```

**Always run tests before making changes.** If tests are red before you start, stop and fix them first.

## TDD Workflow

1. **Red**: Write a failing test that describes the desired behavior
   - Commit: `test: red - <behavior being tested>`
2. **Green**: Write the minimum code to make it pass
   - Commit: `feat: green - <what was implemented>`
3. **Refactor** (optional): Clean up without breaking tests
   - Commit: `refactor: <what was cleaned up>`

Never skip the red phase. Never write code without a corresponding test.

## Code Style

### TypeScript (API + Mobile)
- Strict mode enabled (`"strict": true` in tsconfig)
- No `any` — use `unknown` with type guards
- All async functions must have explicit return types
- Error handling: always use typed `AppError` class, never throw plain strings
- Named exports only — no default exports (makes refactoring easier)

### API (`src/`)
- Route handlers are thin: validate input → call service → return response
- Business logic lives in `services/`, DB access in `models/`
- HTTP status codes: 200 success, 201 created, 400 bad input, 401 unauth, 403 forbidden, 404 not found, 429 rate limited
- All model functions return `{ data, error }` (Supabase pattern)

### Mobile (`mobile/`)
- Screens use named function declarations (`function FeedScreen()`)
- Custom hooks for all data fetching — no raw Supabase calls in components
- Zustand store: one slice per domain (auth, feed, connections)
- StyleSheet.create() for all styles — no inline style objects
- Components must have a `.test.tsx` counterpart in the same directory

### Naming
- Files: `camelCase.ts`, screens: `PascalCaseScreen.tsx`, components: `PascalCase.tsx`
- DB columns: `snake_case`, TS variables: `camelCase`
- Boolean props/fields: prefix with `is` or `has`

## Critical Invariants (Never Break)

1. `submitter_id` must **never** appear in any API response — strip in model layer
2. Feedback can only be submitted to users who are `is_open_for_feedback = true`
3. Submitter must be an accepted connection of the target
4. `curateFeed()` must always return 3 positive + 1 negative (negative last), unless total feedback < 4
5. RLS policies must be in place before any new table is used in production

## PR Instructions

Every PR must include in the description:
- **What tests were written** (list the test names)
- **What tests pass** (`npm test` output or screenshot)
- **What was manually tested** (device/emulator, specific steps)
- **Which TODO.md item** this closes

PRs with failing tests will not be merged. PRs that delete tests without justification will not be merged.
