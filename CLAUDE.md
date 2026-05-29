# CLAUDE.md — AnonymousFeedback

## Orientation

Mobile-first anonymous feedback app. Instagram-like feed that shows **3 positive + 1 negative** pieces of feedback from your trusted network. Two codebases in one repo: `src/` (Node/Express API) and `mobile/` (React Native + Expo).

## Commands

### API (`src/`)
```bash
npm run dev          # Start dev server (nodemon, port 3001)
npm test             # Run all tests (Jest)
npm run test:watch   # Watch mode
npm run test:unit    # Unit tests only (tests/unit/)
npm run test:int     # Integration tests only (tests/integration/)
npm run lint         # ESLint
npm run lint:fix     # ESLint --fix
npm run build        # tsc compile to dist/
npm run db:migrate   # Run pending Supabase migrations
npm run db:reset     # Reset local Supabase DB (destroys data)
npm run db:seed      # Seed with test users + fake feedback
```

### Mobile (`mobile/`)
```bash
npx expo start       # Start Expo dev server
npx expo start --ios # iOS Simulator
npx expo start --android # Android Emulator
npm test             # Jest + RNTL
npx eas build --profile preview  # EAS preview build
```

### Supabase Local Dev
```bash
supabase start       # Start local Supabase (Docker required)
supabase stop        # Stop
supabase status      # Get local URLs + anon key
supabase db diff     # Show uncommitted schema changes
```

## Directory Map

```
src/
  api/            Route handlers (thin — delegate to services)
  models/         Supabase query wrappers (no raw SQL elsewhere)
  services/       Business logic — feed curation, invite tokens, rate limiting
  middleware/     Auth (verify Supabase JWT), rate limit, input validation
  utils/          Pure helpers — token generation, date math, sanitization

mobile/
  screens/        One file per screen (see spec.md §8 for screen map)
  components/     Reusable UI — FeedbackCard, ProfileToggle, ConnectionItem
  hooks/          useFeed, useProfile, useConnections, useAuth
  navigation/     React Navigation stack + tab config
  store/          Zustand slices: auth, feed, connections

tests/
  unit/           Fast, no DB — test pure functions and services
  integration/    Supertest against real Express app, mocked Supabase
  e2e/            Detox — real device/simulator flows

docs/
  spec.md         Source of truth for features and data model
  adr/            Architecture decisions (0001 = template)
```

## Key Conventions

### Feed Curation
- The `curateFeed()` function in `src/services/feedService.ts` is **the** most important piece of logic
- It is unit-tested exhaustively — do not change its behavior without updating tests first
- Rule: 3 positive + 1 negative. Negative always last. See `docs/spec.md §4.5` for the full algorithm

### Supabase RLS
- **Never** bypass RLS with the service role key in application code
- Service role is only for migrations and admin scripts
- All app DB access uses the anon key + user JWT

### Anonymity Invariant
- The `submitter_id` column exists in `feedback` for moderation only
- **Never** expose `submitter_id` in any API response — strip it in the model layer
- Test: `GET /feed` response must not contain any `submitter_id` field

### Rate Limiting
- Two layers: Express `express-rate-limit` middleware (IP-based) + DB unique constraint
- The DB constraint `UNIQUE (submitter_id, target_user_id, DATE(created_at))` is the source of truth
- Don't rely on middleware alone

### Environment Variables
```
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=   # never in app code, migrations only
PORT=3001
NODE_ENV=development
```

## Workflow

1. **Before writing any code**: `npm test` — all tests must pass first
2. Write the failing test (red) → commit `test: red - <what>`
3. Write minimum code to pass (green) → commit `feat: green - <what>`
4. Run `npm run lint` — fix before committing
5. Update this file if you discover a non-obvious convention
6. Check `TODO.md` for what phase you're in — don't skip ahead

## Gotchas

- Expo Go doesn't support all native modules — use a dev build for push notifications
- Supabase magic link redirects require a custom URI scheme: `anonfeedback://auth/callback`
- The `DATE(created_at)` rate limit is UTC-based — a user at 11pm can submit again at midnight
- React Navigation v6 + Expo Router can conflict — pick one, we use React Navigation
- Jest and Supabase client don't play well — always mock `@supabase/supabase-js` in tests
