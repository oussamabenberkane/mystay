# Agent Task: Wire French i18n into All Components

## Context

This is a Next.js 16 App Router project using **next-intl** for i18n. Locale routing is set up — routes are prefixed `/{locale}/...` (en, fr, ar). Translation files exist at:

- `messages/en.json` — source of truth, all keys populated
- `messages/fr.json` — fully translated French counterpart, matching key structure

**Problem**: No component currently uses `useTranslations`. All UI text is hardcoded English strings. The task is to wire French translations in without breaking English.

## Your Goal

Replace every hardcoded user-visible string across all guest, staff, admin, and auth pages/components with `t('key')` calls sourced from the existing translation files. Do **not** add or rename any keys in the JSON files — only use keys that already exist.

## Rules

1. **Server Components** that need translations: use `import { getTranslations } from 'next-intl/server'` and `const t = await getTranslations('Namespace')`.
2. **Client Components**: use `import { useTranslations } from 'next-intl'` and `const t = useTranslations('Namespace')`.
3. **Do not create new translation keys** — only use keys already in `messages/en.json` and `messages/fr.json`.
4. If a hardcoded string has no match in the JSON files, leave it hardcoded and add a `// TODO: i18n` comment.
5. **Arabic (`ar`) is intentionally non-functional** — do not add `messages/ar.json` keys or test Arabic.
6. Make sure `next-intl` middleware and `i18n/config.ts` already support FR routing — verify before changing config.
7. Preserve all existing functionality. Only change text strings, not logic.

## Translation Key Namespaces (from messages/en.json)

```json
{
  "nav": { ... },         // bottom nav labels
  "auth": { ... },        // login/signup pages
  "guest": { ... },       // guest dashboard, orders, requests, expenses, chat
  "staff": { ... },       // staff orders, chat pages
  "admin": { ... },       // admin menu, operations, users, stays pages
  "common": { ... },      // shared labels (Save, Cancel, Loading, etc.)
  "status": { ... }       // order/request status labels
}
```

## Files to Update

Work through these in order:

### Auth Pages (Client Components)
- `src/app/[locale]/(auth)/login/page.tsx`
- `src/app/[locale]/(auth)/signup/page.tsx`

### Guest Pages
- `src/app/[locale]/(guest)/dashboard/page.tsx` (Server Component)
- `src/app/[locale]/(guest)/orders/page.tsx` (Client Component)
- `src/app/[locale]/(guest)/menu/page.tsx`
- `src/app/[locale]/(guest)/requests/page.tsx`
- `src/app/[locale]/(guest)/expenses/page.tsx` (Server Component)
- `src/app/[locale]/(guest)/chat/page.tsx` (Client Component)
- `src/app/[locale]/(guest)/_components/` — all components inside

### Staff Pages
- `src/app/[locale]/(staff)/staff/orders/page.tsx`
- `src/app/[locale]/(staff)/staff/chat/page.tsx`
- `src/components/staff/` — ConversationList, ChatPanel, etc.

### Admin Pages
- `src/app/[locale]/(admin)/admin/menu/page.tsx`
- `src/app/[locale]/(admin)/admin/operations/page.tsx`
- `src/app/[locale]/(admin)/admin/users/page.tsx`
- `src/app/[locale]/(admin)/admin/stays/page.tsx`
- `src/app/[locale]/(admin)/admin/menu/_components/`
- `src/app/[locale]/(admin)/admin/operations/_components/`
- `src/components/admin/`

### Shared Components
- `src/components/shared/` — MessageBubble, MessageInput, BottomNav, etc.

## How to Verify

After each file, check that:
1. TypeScript compiles without errors (`pnpm tsc --noEmit`)
2. The English text still shows correctly at `http://localhost:3001/en/...`
3. The French text shows at `http://localhost:3001/fr/...`

## Example Pattern

Before:
```tsx
// Server Component
export default async function Page() {
  return <h1>Welcome back</h1>
}
```

After:
```tsx
import { getTranslations } from 'next-intl/server'

export default async function Page() {
  const t = await getTranslations('guest')
  return <h1>{t('welcomeBack')}</h1>
}
```

Before (Client):
```tsx
'use client'
export default function Component() {
  return <button>Sign In</button>
}
```

After:
```tsx
'use client'
import { useTranslations } from 'next-intl'

export default function Component() {
  const t = useTranslations('auth')
  return <button>{t('signIn')}</button>
}
```
