# MyStay — Validation & QA Plan

> **Scope:** Production readiness validation for the MyStay hotel guest experience platform.
> **Stack:** Next.js 16 App Router · Supabase (Auth + RLS + Realtime) · PWA · Web Push · next-intl (en/fr/ar)
> **Roles:** `client` (guest) · `staff` · `admin`

---

## 1. End-to-End Testing (Playwright MCP)

### 1.1 Test Organization

```
tests/
  e2e/
    auth/
    client/
    staff/
    admin/
    security/
  fixtures/
    seed.ts        # test data factories
    auth.ts        # role-based login helpers
  playwright.config.ts
```

Each test file maps to one user journey. Tests must be fully isolated — no shared state between runs.

---

### 1.2 Authentication Flows

| Scenario | Steps | Expected |
|---|---|---|
| Login — valid client | Enter valid email + password | Redirect to `/[locale]/dashboard` |
| Login — valid staff | Enter valid email + password | Redirect to `/[locale]/staff/orders` |
| Login — valid admin | Enter valid email + password | Redirect to `/[locale]/admin/operations` |
| Login — wrong password | Submit invalid credentials | Error message shown, no redirect |
| Login — unknown email | Submit nonexistent user | Error message shown |
| Forgot password | Submit registered email | Success message shown |
| Reset password | Use valid recovery link | New password accepted, redirected to login |
| Reset password — expired link | Use expired token | Error page or redirect to login |
| Logout | Click logout | Session cleared, redirect to `/login` |
| Direct URL access (unauthenticated) | Navigate to `/dashboard` | Redirect to `/login` |

---

### 1.3 Client (Guest) Journeys

#### 1.3.1 Room Service

| Scenario | Steps |
|---|---|
| Browse menu | Navigate to `/menu`, categories and items render |
| Add to cart | Click item → cart count increments |
| Adjust quantity | Increase/decrease quantity in cart |
| Remove item | Remove item from cart, cart updates |
| Place order | Checkout from cart, order confirmed |
| View order status | Navigate to `/orders`, order appears with `pending` status |
| Real-time status update | Staff updates order status → guest sees update without refresh |
| Order with note | Add special instructions, verify saved to order |
| Empty menu | Menu with no available items shows empty state |

#### 1.3.2 Service Requests

| Scenario | Steps |
|---|---|
| Submit cleaning request | Select type `cleaning`, submit | Status `pending` visible |
| Submit urgent maintenance | Set priority `urgent` | Displayed with urgency indicator |
| View request history | Navigate to `/requests` | All requests listed with status |
| Real-time status update | Staff updates request → guest sees update without refresh |
| Submit request — no active stay | Guest without active stay | Graceful error, not a crash |

#### 1.3.3 Chat

| Scenario | Steps |
|---|---|
| Send message to staff | Type and send | Message appears in thread |
| Receive staff reply | Staff sends reply → appears in real time |
| Multi-line message | Send message with line breaks | Renders correctly |
| Empty message | Submit blank message | Blocked, no API call |

#### 1.3.4 Expenses

| Scenario | Steps |
|---|---|
| View bill | Navigate to `/expenses` | All non-cancelled orders listed with totals |
| Empty bill | No orders placed | Empty state shown |

---

### 1.4 Staff Journeys

| Scenario | Steps |
|---|---|
| View incoming orders | Navigate to `/staff/orders` | All hotel orders visible |
| Update order status | Confirm → Preparing → Delivering → Delivered | Status updates, real-time on guest side |
| Cancel order | Cancel pending order | Status changes, guest notified |
| View service requests | Navigate to `/staff/requests` | All pending requests visible |
| Update request status | Move to `in_progress` → `completed` | Status changes, real-time on guest side |
| Set request priority | Mark as urgent | Priority indicator visible |
| View guest chat | Open conversation | Messages visible |
| Reply to guest | Send message | Guest sees reply in real time |
| Push notification — new order | New order placed | Staff receives push notification |
| Push notification — urgent request | Guest submits urgent request | Staff receives push notification |

---

### 1.5 Admin Journeys

| Scenario | Steps |
|---|---|
| View operations dashboard | Navigate to `/admin/operations` | Stats (active stays, revenue, orders, requests) render |
| Create menu category | Add name, save | Category appears in menu |
| Create menu item | Fill form with price, category | Item appears on guest menu |
| Edit menu item | Update name/price | Change reflected for guests |
| Toggle item availability | Disable item | Item hidden from guest menu |
| Delete menu item | Confirm delete | Item removed |
| Create stay | Select guest, room, dates | Stay created, guest can now access hotel features |
| Archive stay | Mark stay as completed | Guest loses active stay access |
| Create staff user | Enter name, email, password | Staff can log in |
| Create admin user | Enter details with admin role | Admin can log in |
| Delete staff user | Confirm delete | User loses access |
| View guest list | Navigate to `/admin/guests` | All guests for the hotel listed |

---

### 1.6 Role-Based Access Control

| Attempt | Expected |
|---|---|
| Client navigates to `/staff/orders` | Redirected to `/dashboard` |
| Client navigates to `/admin/operations` | Redirected to `/dashboard` |
| Staff navigates to `/admin/menu` | Redirected to `/staff/orders` |
| Staff navigates to `/dashboard` | Redirected to `/staff/orders` |
| Admin navigates to `/dashboard` | Redirected to `/admin/operations` |
| Unauthenticated user navigates to any protected route | Redirected to `/login` |

---

### 1.7 Multi-Tenant Isolation

| Scenario | Expected |
|---|---|
| Guest A (Hotel 1) cannot see Hotel 2 menu | Only Hotel 1 items returned |
| Staff A (Hotel 1) cannot see Hotel 2 orders | RLS blocks cross-hotel queries |
| Admin A (Hotel 1) cannot manage Hotel 2 users | Supabase RLS enforced |
| Direct API call with Hotel 1 token to Hotel 2 resource | 0 rows returned or error |

---

### 1.8 Internationalization

| Scenario | Expected |
|---|---|
| Navigate to `/fr/dashboard` | UI renders in French |
| Navigate to `/ar/dashboard` | UI renders in Arabic, RTL layout applied |
| Switch language | Preference persisted |
| Missing translation key | Fallback to English, no visible key string |

---

## 2. Edge Cases & Reliability

### 2.1 Input Validation

| Case | Simulate | Expected |
|---|---|---|
| Empty order checkout | Submit cart with 0 items | Button disabled or error shown |
| Order with 0 quantity | Manually set quantity to 0 | Blocked before submission |
| Service request — empty description | Submit blank description | Validation error inline |
| Signup — invalid email format | Enter `notanemail` | Field-level error |
| Signup — weak password | Enter `123` | Rejection with policy message |
| Admin creates user — duplicate email | Use existing email | Error message, no crash |
| Menu item — negative price | Enter `-5` | Rejected by form validation |
| Long input strings | 1000-char item name | Truncated or rejected gracefully |

### 2.2 Network & Latency

| Case | Simulate | Expected |
|---|---|---|
| Order submission on slow network | Throttle to 3G in DevTools | Loading spinner shown, no double-submission |
| Chat send on slow network | Throttle network | Optimistic UI or clear pending state |
| Supabase realtime disconnect | Kill WS in DevTools | Reconnects automatically or shows offline indicator |
| Server action timeout | Use `setTimeout` mock to delay | Error feedback shown, no silent failure |

### 2.3 Race Conditions

| Case | Simulate | Expected |
|---|---|---|
| Staff + admin update same order simultaneously | Open two browser tabs, update concurrently | Last write wins; no crash; UI reflects final state |
| Guest places order while staff views orders | New order while staff has orders open | Staff list updates via realtime |
| Cart modified while checkout in flight | Add item during submission | Graceful handling; no double-charge |

### 2.4 Authorization Bypass

| Attempt | Expected |
|---|---|
| Craft request to another hotel's order ID | Supabase RLS returns empty result |
| Modify JWT role claim client-side | Server ignores client claims; uses DB role |
| Access admin server actions as staff | Action returns unauthorized error |
| Replay expired session token | Session rejected, redirected to login |

### 2.5 Empty & Degraded States

| Scenario | Expected |
|---|---|
| Guest with no orders | `/orders` shows empty state component, not blank page |
| Guest with no active stay | Service request & menu features show clear "no active stay" message |
| Hotel with no menu categories | Menu page shows empty state |
| Staff with no pending orders | `/staff/orders` shows empty state |
| Admin operations with zero activity | Stats show `0`, not `null` or crash |
| Push subscription fails | Feature degrades silently; core flow unaffected |

---

## 3. UX & UI Quality Checklist

Run this checklist manually against each page before release.

### 3.1 Interactive States

- [ ] All buttons show `cursor-pointer`; disabled buttons show `cursor-not-allowed` and are visually dimmed
- [ ] Buttons in loading state are disabled and show spinner — no double-submit possible
- [ ] Links have `:hover` styles (underline or color change)
- [ ] Form inputs have visible `:focus` ring (not just browser default)
- [ ] Destructive actions (delete, cancel) require confirmation before executing

### 3.2 Feedback & Communication

- [ ] Success actions show toast/confirmation (order placed, request submitted, user created)
- [ ] Error states show actionable messages — not raw error strings or blank screens
- [ ] Form validation errors appear inline next to the relevant field
- [ ] Loading states are shown for all async operations (fetch, submit, realtime connect)
- [ ] Skeleton loaders match the layout of the loaded content

### 3.3 Realtime UX

- [ ] New messages/orders appear without page refresh
- [ ] Status badge updates in place without layout shift
- [ ] Realtime updates do not cause scroll position to jump
- [ ] Notification badge increments for unread items

### 3.4 Accessibility

- [ ] All interactive elements reachable via keyboard (`Tab` / `Shift+Tab`)
- [ ] Modals trap focus and close on `Escape`
- [ ] Images and icons have descriptive `alt` text or `aria-label`
- [ ] Color is not the only signal (e.g., error states also use icon + text)
- [ ] Page has a correct `<h1>` hierarchy on every route

### 3.5 Mobile & Responsiveness

- [ ] All pages usable on 375px (iPhone SE) viewport without horizontal scroll
- [ ] Touch targets are at minimum 44×44px
- [ ] Navigation/drawer opens and closes correctly on mobile
- [ ] Cart and checkout work correctly on touch devices
- [ ] RTL layout (`/ar`) renders correctly at all breakpoints

### 3.6 RTL Specifics (Arabic)

- [ ] Text alignment is right-to-left
- [ ] Flex direction and padding/margin are mirrored correctly
- [ ] Icons with directional meaning (arrows, chevrons) are flipped
- [ ] Number formatting matches locale

---

## 4. Regression Strategy

### 4.1 Critical Path — Must Always Pass

These flows must never regress. They form the minimal regression suite.

1. **Login** → correct role redirect (all 3 roles)
2. **Guest places order** → order visible in staff panel
3. **Staff updates order status** → guest sees update in real time
4. **Admin creates stay** → guest gains access
5. **Menu CRUD** → changes reflected on guest menu immediately
6. **Push notification on new order** (staff subscription active)
7. **RLS isolation** — guest A cannot access hotel B data
8. **Locale routing** — `/fr`, `/ar` render without errors

### 4.2 Regression Triggers

Run the full critical path suite when:

- Any changes to `src/middleware.ts`
- Any changes to `src/lib/actions/`
- Any new Supabase migration
- Any changes to RLS policies
- Any dependency upgrade (Next.js, Supabase client, next-intl)

### 4.3 Preventing Regressions

- Each new feature must come with its own Playwright scenario added before merge
- RLS policy changes must include an isolation test
- Breaking changes to server actions must be paired with a regression test run

---

## 5. Test Execution Strategy

### 5.1 Local Execution

```bash
# Run full suite
npx playwright test

# Run specific role suite
npx playwright test tests/e2e/client/
npx playwright test tests/e2e/staff/
npx playwright test tests/e2e/admin/

# Run critical path only
npx playwright test --grep @critical

# Debug single test
npx playwright test --debug tests/e2e/auth/login.spec.ts
```

Tag critical path tests with `@critical` annotation for fast selective runs.

### 5.2 CI Integration (GitHub Actions)

- **Trigger:** Every push to `main` and every PR targeting `main`
- **Run:** Critical path suite (`@critical`) on every PR; full suite on merge to `main`
- **Parallelism:** Split by role group (client / staff / admin) across workers
- **Artifacts:** Upload HTML report + screenshots on failure

### 5.3 Test Data Strategy

| Concern | Approach |
|---|---|
| Isolation | Each test run creates its own hotel via the `hotels` table and signs up fresh users — no shared fixture accounts |
| Seeding | `fixtures/seed.ts` creates: 1 hotel, 1 admin, 1 staff, 1 client, 1 room, 1 active stay, and a small menu |
| Teardown | Delete the test hotel at the end of each suite (cascades to all related rows) |
| Credentials | Stored in `.env.test` (never committed); CI reads from repo secrets |
| Realtime | Use Supabase test project (not production) with realtime enabled |

### 5.4 Environment Setup

- Maintain a dedicated **test Supabase project** separate from development and production
- Use `NEXT_PUBLIC_SUPABASE_URL` / `SUPABASE_SERVICE_ROLE_KEY` in `.env.test` pointing to the test project
- Run `next dev` (or `next start` after build) against the test environment before Playwright
- Playwright `baseURL` set to `http://localhost:3000`
- Push notifications: use a test VAPID key pair; skip delivery assertion, only assert subscription was saved

### 5.5 Flakiness & Performance

| Issue | Mitigation |
|---|---|
| Realtime timing | Use `waitForResponse` or poll until element appears (max 5s); avoid fixed `sleep` |
| Auth state leaking | Use separate browser contexts per test, never share `storageState` across roles |
| Slow DB queries | Seed only minimum required data; avoid full-table scans in fixtures |
| Locale prefix | Parameterize critical tests across `en`, `fr`, `ar` for locale coverage |
| Order of operations | Each test must set up its own prerequisites — no dependency on test execution order |

---

## 6. Known Risk Areas

These areas require extra scrutiny during QA:

| Area | Risk |
|---|---|
| Supabase RLS policies | Cross-hotel data leaks if a policy has a gap |
| Realtime subscriptions | Memory leaks if channels not cleaned up on unmount |
| Cart state (Zustand) | State persists across navigation; could leak between guest sessions |
| Server actions without auth check | Possible unauthorized mutation if RLS is bypassed at the action level |
| Push notification fallback | Browsers without push support must not break the core flow |
| `get_active_stay` RPC | Returns null for guests without a stay — all callers must handle null safely |

---

*Last updated: 2026-04-20*
