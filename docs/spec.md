# AnonymousFeedback — Feature Specification

**Version**: 0.1 (pre-implementation)
**Status**: Draft
**Last updated**: 2025

---

## 1. Problem Statement

Professional 360-degree feedback tools (Lattice, Culture Amp) exist at work but feel clinical and are locked behind corporate accounts. In social circles, honest feedback is rare — people avoid it to protect relationships. AnonymousFeedback creates a trusted, bounded space where your actual network (friends, colleagues, collaborators) can give you the honest signal that helps you grow, delivered in a way that feels like Instagram, not HR software.

---

## 2. Core Design Principles

1. **Signal over noise**: Never overwhelm. Always 3 positive + 1 negative.
2. **Anonymity is sacred**: Recipients never see who submitted. Submitters know their text is visible only to the recipient.
3. **Consent-first**: You choose when to be open. Feedback can only be submitted to open profiles.
4. **Trust network only**: Feedback comes from your direct connections — not strangers.
5. **No gamification of negativity**: One negative per session prevents pile-ons.

---

## 3. User Roles

| Role | Description |
|------|-------------|
| **Recipient** | Opens their profile to feedback; views their curated feed |
| **Submitter** | Sends anonymous feedback to a connected open profile |

_Note: A user can be both a recipient and a submitter._

---

## 4. Functional Requirements

### 4.1 Authentication
- [ ] Users sign in via magic link email or Google OAuth
- [ ] New users complete a 3-field onboarding: display name, optional avatar, optional short bio
- [ ] Sessions persist across app restarts using device secure storage
- [ ] Users can delete their account (cascades all feedback data)

### 4.2 Connections
- [ ] User A can invite User B via a shareable link or by searching username
- [ ] User B must accept before A can submit feedback to B (or vice versa)
- [ ] Both parties see each other in their connections list after acceptance
- [ ] Either party can remove or block a connection at any time
- [ ] Blocking prevents future feedback submissions and hides existing connection

### 4.3 Open/Close Toggle
- [ ] Users can toggle `is_open_for_feedback` at any time
- [ ] Default state for new users: **closed**
- [ ] When open: a "Share" button generates a link with message: "Give me honest feedback: [link]"
- [ ] When closed: submitters see "This person is not currently accepting feedback"
- [ ] Toggle state is visible on the user's public profile card

### 4.4 Feedback Submission
- [ ] Submitter can only submit if: (a) connected to target, (b) target is open, (c) within rate limit
- [ ] Feedback form: text area (max 280 chars), sentiment toggle (positive 👍 / constructive 💬)
- [ ] Rate limit: 1 submission per submitter-target pair per 24 hours
- [ ] No edit after submission — feedback is final
- [ ] Submitter receives a confirmation but no receipt of read status
- [ ] Submitter cannot see their own submission after it's sent (prevents reverse-engineering identity)

### 4.5 Feed Curation (Core Algorithm)
```
Algorithm: curate_feed(user_id) -> FeedItem[4]

1. Fetch all unread positive feedback for user → unread_pos[]
2. Fetch all unread negative feedback for user → unread_neg[]
3. Select up to 3 from unread_pos (newest first) → selected_pos[]
4. If len(selected_pos) < 3:
     pad with read positives (oldest first) until len = 3
5. If len(unread_neg) > 0:
     selected_neg = unread_neg[0]  # newest
6. Else:
     selected_neg = None
7. If selected_neg is None:
     return selected_pos[:4]  # show 4 positives
8. Return shuffle(selected_pos) + [selected_neg]
   # negative always last in the array, displayed last in feed
```

- [ ] Feed endpoint returns exactly 4 items (or fewer only if user has < 4 total feedback)
- [ ] Feed items are marked unread → read when the card is viewed (scrolled into viewport)
- [ ] Dismissed items are hidden for 30 days, then re-eligible
- [ ] "Helpful" marks increase item priority in future surfacing (v2 feature)

### 4.6 Notifications
- [ ] Push notification when new feedback arrives (batched: at most once per day)
- [ ] In-app badge count on feed tab shows unread feedback count
- [ ] No notification reveals feedback content — only "You have new feedback"

---

## 5. Non-Functional Requirements

- [ ] API response time < 300ms p95 for feed endpoint
- [ ] Mobile app cold start < 2s on a 3-year-old device
- [ ] Supabase Row Level Security (RLS) enforced on all tables — no server-side trust
- [ ] All feedback text stored encrypted at rest (Supabase default AES-256)
- [ ] Zero PII in logs — mask user IDs, truncate feedback text in any log lines
- [ ] GDPR: full data export (`GET /users/me/export`) and account deletion within 30 days

---

## 6. Data Model

### `users`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
email         text UNIQUE NOT NULL
display_name  text NOT NULL
avatar_url    text
bio           text
is_open_for_feedback  boolean DEFAULT false
created_at    timestamptz DEFAULT now()
updated_at    timestamptz DEFAULT now()
```

### `connections`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
requester_id  uuid REFERENCES users(id) ON DELETE CASCADE
recipient_id  uuid REFERENCES users(id) ON DELETE CASCADE
status        text CHECK (status IN ('pending', 'accepted', 'blocked'))
created_at    timestamptz DEFAULT now()
UNIQUE (requester_id, recipient_id)
```

### `feedback`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
target_user_id   uuid REFERENCES users(id) ON DELETE CASCADE
submitter_id     uuid REFERENCES users(id) ON DELETE SET NULL
text             text NOT NULL CHECK (char_length(text) <= 280)
sentiment        text CHECK (sentiment IN ('positive', 'negative'))
is_read          boolean DEFAULT false
is_dismissed     boolean DEFAULT false
dismissed_until  timestamptz
report_count     int DEFAULT 0
is_hidden        boolean DEFAULT false
created_at       timestamptz DEFAULT now()
UNIQUE (submitter_id, target_user_id, DATE(created_at))  -- rate limit
```

### `feedback_sessions`
```sql
id              uuid PRIMARY KEY DEFAULT gen_random_uuid()
user_id         uuid REFERENCES users(id) ON DELETE CASCADE
feedback_ids    uuid[]  -- the 4 items surfaced in this session
created_at      timestamptz DEFAULT now()
```

---

## 7. API Design

### Auth
| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/callback` | Supabase auth redirect handler |
| DELETE | `/auth/session` | Sign out |

### Users
| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/me` | Current user profile |
| PATCH | `/users/me` | Update profile |
| DELETE | `/users/me` | Delete account (GDPR) |
| GET | `/users/me/export` | Export all user data |
| PATCH | `/users/me/feedback-status` | Toggle open/closed |
| GET | `/users/:id` | Public profile (display name, avatar, open status) |

### Connections
| Method | Path | Description |
|--------|------|-------------|
| GET | `/connections` | List accepted connections |
| POST | `/connections/invite` | Create invite link |
| POST | `/connections/accept/:token` | Accept invite |
| DELETE | `/connections/:id` | Remove connection |
| PATCH | `/connections/:id/block` | Block connection |

### Feedback
| Method | Path | Description |
|--------|------|-------------|
| POST | `/feedback` | Submit feedback to a connected user |
| POST | `/feedback/:id/report` | Report inappropriate content |

### Feed
| Method | Path | Description |
|--------|------|-------------|
| GET | `/feed` | Get curated 3+1 feed |
| POST | `/feed/:id/mark-read` | Mark item as read |
| POST | `/feed/:id/helpful` | Mark as helpful |
| POST | `/feed/:id/dismiss` | Dismiss for 30 days |

---

## 8. Mobile Screen Map

```
App
├── Auth
│   ├── WelcomeScreen
│   ├── EmailInputScreen
│   ├── MagicLinkSentScreen
│   └── OnboardingScreen (3 swipe steps)
└── Main (Tab Navigator)
    ├── FeedTab
    │   ├── FeedScreen (card stack, 3+1)
    │   └── FeedbackDetailScreen (expanded card)
    ├── ConnectionsTab
    │   ├── ConnectionsScreen (list)
    │   ├── PendingRequestsScreen
    │   └── FriendProfileScreen (submit feedback here)
    └── ProfileTab
        ├── MyProfileScreen (toggle, share button, stats)
        └── SettingsScreen (notifications, account deletion)
```

---

## 9. Test Plan

### Unit Tests (src/)
- `curateFeed()` — all edge cases: empty, partial, all-positive, all-negative
- `validateFeedback()` — text length, rate limit check, connection check, open status check
- `generateInviteToken()` / `validateInviteToken()` — expiry, single-use
- `Connection.canSubmitFeedback(submitter, target)` — all boolean combinations

### Integration Tests (API)
- Full auth flow (mock Supabase)
- `POST /feedback` — valid, rate-limited, closed target, unconnected submitter
- `GET /feed` — correct 3+1 composition with seeded DB data
- `PATCH /users/me/feedback-status` — toggle persists, share link generated

### Mobile Tests
- Feed card swipe interaction (jest + RNTL)
- Feedback submission form validation (char count, sentiment required)
- Open/close toggle UI state

### E2E (Detox)
- Full user journey: sign in → connect with friend → friend submits feedback → toggle open → view feed

---

## 10. Open Questions

- [ ] Should we show *how many* unread feedback items a user has, or just "new feedback"? (Showing counts might create anxiety)
- [ ] What happens to feedback when a connection is removed? Archive it or delete it?
- [ ] Should submitters be able to *update* their feedback within a grace period (e.g., 1 hour)?
- [ ] Minimum connection age before feedback allowed? (prevents burner account abuse)
- [ ] Should the negative-always-last rule be user-configurable?
- [ ] App name trademark check needed before public launch
