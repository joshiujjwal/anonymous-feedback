# AnonymousFeedback — Task Breakdown

## How to Use This File

Workflow per task:
1. Write failing tests FIRST (red phase) — commit with `test: red - <what>`
2. Implement until tests pass (green phase) — commit with `feat: green - <what>`
3. Review diff manually before moving on
4. Commit with descriptive message referencing this task
5. Update `CLAUDE.md` or `AGENTS.md` if you discovered something non-obvious (compound loop)

**Evidence gate**: Before marking a phase ✅, all checkboxes must be checked AND tests must be green.

---

## Phase 0: Foundation ⬜

- [ ] Initialize `src/` as Node.js project (`npm init`, TypeScript config)
- [ ] Initialize `mobile/` as Expo project (`npx create-expo-app`)
- [ ] Add ESLint + Prettier for both `src/` and `mobile/`
- [ ] Set up Jest for API (`src/`) with a smoke test that passes
- [ ] Set up Jest + React Native Testing Library for `mobile/`
- [ ] Create `.env.example` with all required env vars documented
- [ ] Set up Supabase project (local dev via `supabase start`)
- [ ] Write first DB migration: `users`, `feedback`, `feedback_sessions` tables
- [ ] GitHub Actions CI: lint + test on push to `main` and PRs
- [ ] Review `CLAUDE.md`, `AGENTS.md`, `docs/spec.md` — update if stale

**Gate**: CI passes on an empty test suite ✅

---

## Phase 1: Auth & User Identity ⬜

- [ ] **[red]** Write tests for user registration/login flow
- [ ] Supabase Auth integration — magic link email + Google OAuth
- [ ] User profile model: `id`, `display_name`, `avatar_url`, `is_open_for_feedback`, `created_at`
- [ ] `POST /auth/callback` — handle Supabase auth callback
- [ ] `GET /users/me` — return current user profile
- [ ] `PATCH /users/me` — update display name, avatar
- [ ] Mobile: Auth screens (welcome, email input, magic link sent, OAuth buttons)
- [ ] Mobile: Persist session with SecureStore
- [ ] **[green]** All auth tests pass
- [ ] Manual test: full sign-in flow on iOS Simulator + Android Emulator

**Gate**: A user can sign in and their session persists across app restarts ✅

---

## Phase 2: Friend/Colleague Connections ⬜

- [ ] **[red]** Write tests for connection invite and acceptance
- [ ] `connections` table: `id`, `requester_id`, `recipient_id`, `status` (pending/accepted/blocked)
- [ ] `POST /connections/invite` — send invite via shareable link or username
- [ ] `POST /connections/accept/:id` — accept a pending connection
- [ ] `DELETE /connections/:id` — remove a connection
- [ ] `GET /connections` — list accepted connections
- [ ] Mobile: Connections screen (list, invite via share sheet, pending requests badge)
- [ ] Deep link handling for invite URLs (`anonfeedback://invite/:token`)
- [ ] **[green]** All connection tests pass
- [ ] Manual test: two test accounts can connect via invite link

**Gate**: Two users can connect and see each other in their connections list ✅

---

## Phase 3: Feedback Submission ⬜

- [ ] **[red]** Write tests for feedback submission and validation
- [ ] `feedback` table: `id`, `target_user_id`, `submitter_id`, `text`, `sentiment` (positive/negative), `created_at`, `is_read`
- [ ] Validation: submitter must be connected to target, target must be `is_open_for_feedback = true`
- [ ] `POST /feedback` — submit feedback (anonymous to recipient)
- [ ] Rate limiting: max 1 feedback per submitter-target pair per 24h
- [ ] `GET /users/:id/feedback-status` — check if user is open for feedback (public endpoint)
- [ ] Mobile: Feedback submission screen (friend profile → write feedback → positive/negative toggle → submit)
- [ ] Mobile: Character limit enforcement (280 chars) with live counter
- [ ] **[green]** All submission tests pass

**Gate**: A connected user can submit feedback to an open friend ✅

---

## Phase 4: The Feed — Core Curation Logic ⬜

- [ ] **[red]** Write unit tests for feed curation algorithm
- [ ] Feed curation service: select 3 newest unread positive + 1 newest unread negative
- [ ] If < 3 positive available: pad with oldest already-read positives
- [ ] If no negative available: show 4 positives (never show empty feed)
- [ ] `GET /feed` — returns curated `[pos, pos, pos, neg]` array (shuffled positions except neg is always last)
- [ ] `POST /feed/:id/mark-read` — mark a feedback item as read
- [ ] `POST /feed/:id/helpful` — mark as helpful (soft signal for future ranking)
- [ ] `POST /feed/:id/dismiss` — dismiss (won't surface again for 30 days)
- [ ] Mobile: Feed screen — Instagram-style card stack, swipe to dismiss, tap to expand
- [ ] Mobile: Empty state — "You have no feedback yet. Share your profile to get some."
- [ ] **[green]** All feed curation tests pass with edge cases (empty, partial, all-positive)

**Gate**: Feed correctly shows 3+1 and degrades gracefully ✅

---

## Phase 5: Open/Close Toggle ⬜

- [ ] **[red]** Write tests for open/close state transitions
- [ ] `PATCH /users/me/feedback-status` — toggle `is_open_for_feedback`
- [ ] When closing: pending unsubmitted feedback drafts from friends are discarded
- [ ] When opening: generate shareable link with UTM for distribution
- [ ] Mobile: Profile screen — prominent toggle (open = green pulse, closed = grey)
- [ ] Mobile: Share sheet when user opens for feedback (pre-filled message)
- [ ] Notification: push when you receive new feedback (batched, max 1/day)
- [ ] **[green]** All toggle tests pass

**Gate**: Toggle persists, share link works, push notification fires ✅

---

## Phase 6: Polish & Safety ⬜

- [ ] Report feedback as inappropriate (`POST /feedback/:id/report`)
- [ ] Auto-hide reported feedback pending review (threshold: 2 reports)
- [ ] Block a connection (`PATCH /connections/:id/block`) — stops future submissions
- [ ] Rate limit abuse: shadow-ban submitters who get 3 reports in 7 days
- [ ] Onboarding flow (3-screen swipe explaining the 3+1 concept)
- [ ] Empty states, loading skeletons, error toasts throughout the app
- [ ] Accessibility audit (VoiceOver / TalkBack on main flows)
- [ ] Privacy screen: "Who can see my feedback?" explainer

**Gate**: All safety paths tested; no crash on any error state ✅

---

## Phase 7: Ship ⬜

- [ ] App Store Connect setup + provisioning profiles
- [ ] Google Play Console setup
- [ ] EAS Build config (`eas.json`) for preview + production
- [ ] Railway production deploy with environment secrets
- [ ] Supabase prod project (separate from dev)
- [ ] Run full e2e test suite on physical device before submission
- [ ] TestFlight beta with 10 real users — collect feedback
- [ ] Submit to App Store + Play Store

**Gate**: Both stores approved, 10 beta users have submitted feedback ✅

---

## Parking Lot 🅿️

- Weekly digest email ("You got 4 new feedbacks this week")
- Feedback themes/tags (auto-tagged by GPT: "communication", "reliability", etc.)
- Public profile mode (shareable link shows feedback stats, not content)
- Streak system: stay open for 7 days and earn a badge
- Web app version (Next.js)
- Anonymous Q&A mode (friends can ask you questions, you answer publicly)

---

## Lessons Learned 📝

_Update this section as you build. What was harder than expected? What shortcuts backfired?_

- [ ] TODO: fill in as you go
