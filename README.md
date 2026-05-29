# AnonymousFeedback

> Get work-like feedback in social settings — from people who actually know you.

![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)
![License](https://img.shields.io/badge/license-MIT-blue)

## What It Does

AnonymousFeedback lets you open yourself up for honest, structured feedback from friends and colleagues — like a 360 review, but for your life. The feed shows exactly **3 positive** and **1 negative** piece of feedback at a time, Instagram-style. No floods. No noise. Just signal.

## Tech Stack

| Layer | Choice |
|-------|--------|
| Mobile | React Native (Expo) |
| Backend API | Node.js + Express |
| Database | PostgreSQL |
| Auth | Supabase Auth (magic link + social) |
| Real-time | Supabase Realtime |
| Notifications | Expo Push Notifications |
| Hosting | Railway (API) + Supabase (DB/Auth) |

## Getting Started

```bash
# Clone
git clone https://github.com/<your-username>/anonymous-feedback.git
cd anonymous-feedback

# Install API dependencies
cd src && npm install

# Install mobile dependencies
cd ../mobile && npm install

# Set up environment variables
cp .env.example .env
# Edit .env with your Supabase project URL and anon key

# Start API dev server
cd ../src && npm run dev

# Start mobile app
cd ../mobile && npx expo start
```

## Running Tests

```bash
# API unit + integration tests
cd src && npm test

# Watch mode
cd src && npm run test:watch

# Mobile tests
cd mobile && npm test
```

## Project Structure

```
anonymous-feedback/
├── src/                    # Express API
│   ├── api/                # Route handlers
│   ├── models/             # DB models (Supabase queries)
│   ├── services/           # Business logic (feedback curation, etc.)
│   ├── middleware/         # Auth, rate limiting, validation
│   └── utils/              # Shared helpers
├── mobile/                 # React Native (Expo) app
│   ├── screens/            # Screen components
│   ├── components/         # Reusable UI components
│   ├── hooks/              # Custom hooks (useFeedback, useProfile)
│   ├── navigation/         # React Navigation config
│   └── store/              # Zustand state management
├── tests/
│   ├── unit/               # Pure function tests
│   ├── integration/        # API endpoint tests (supertest)
│   └── e2e/                # Detox mobile e2e tests
├── docs/
│   ├── spec.md             # Feature specification
│   └── adr/                # Architecture decision records
├── .github/
│   └── copilot-instructions.md
├── README.md
├── TODO.md
├── CLAUDE.md
└── AGENTS.md
```

## Core Concept

- Users **open** themselves to feedback (toggle on/off)
- Friends/colleagues submit anonymous text feedback
- The feed surfaces **3 positive + 1 negative** per session — no more, no less
- Feedback is displayed without attribution
- Users can mark feedback as helpful or dismiss it

## Contributing

1. **Write tests first** (red phase) before any implementation
2. Run existing tests before touching any code: `npm test`
3. Keep PRs small and focused — one feature or fix per PR
4. PR descriptions must include evidence: what tests pass, what you manually verified
5. Never remove a passing test without explicit discussion
