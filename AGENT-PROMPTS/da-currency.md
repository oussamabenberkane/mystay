# Agent Task: Replace All Currencies with DA (Algerian Dinar)

## Context

This is a Next.js 16 hotel management app (MyStay). Currently currency is displayed inconsistently — some places use USD, some EUR. The requirement is to standardize everything to **DA (Algerian Dinar, ISO 4217 code: `DZD`)** with the symbol `DA` displayed before the amount (e.g., `DA 1,500`).

## Your Goal

Replace every currency formatting call across the entire codebase so that all monetary values display in DA.

## The Standard Format

Use this single shared utility (already exists at `src/lib/utils/format.ts`):

```ts
export function formatCurrency(amount: number): string {
  return `DA ${new Intl.NumberFormat('fr-DZ', {
    minimumFractionDigits: 0,
    maximumFractionDigits: 2,
  }).format(amount)}`
}
```

Update `src/lib/utils/format.ts` to use this exact implementation. Remove the `currency` parameter — it should always be DZD.

## Files to Update

Every file that has its own local `formatCurrency`, `formatPrice`, or inline currency formatting:

### 1. `src/lib/utils/format.ts`
- Change default from `'EUR'` to the new DA format above
- Remove the `currency` parameter entirely

### 2. `src/app/[locale]/(guest)/dashboard/page.tsx`
- Has local `formatCurrency` using `currency: 'USD'`
- Delete the local function, import from `@/lib/utils/format`

### 3. `src/app/[locale]/(guest)/expenses/page.tsx`
- Has local `formatCurrency` using `currency: 'USD'`
- Delete the local function, import from `@/lib/utils/format`

### 4. `src/app/[locale]/(admin)/admin/operations/page.tsx`
- Has local `formatCurrency` using `currency: 'USD'`
- Delete the local function, import from `@/lib/utils/format`

### 5. `src/app/[locale]/(admin)/admin/menu/_components/menu-client.tsx`
- Has local `formatPrice` using `currency: 'USD'`, and a label `"Price (USD)"`
- Delete local function, import `formatCurrency` from `@/lib/utils/format`
- Change the label `"Price (USD)"` to `"Price (DA)"`

### 6. `src/app/[locale]/(guest)/orders/page.tsx`
- Uses inline `` €${order.total_amount.toFixed(2)} `` and `` €${item.unit_price.toFixed(2)} ``
- Replace with `formatCurrency(order.total_amount)` / `formatCurrency(item.unit_price)`
- Import `formatCurrency` from `@/lib/utils/format`

## Search for Any Missed Occurrences

After the above, run a search for any remaining:
- `currency: 'USD'`
- `currency: 'EUR'`
- `` `€${ ``
- `` `${ `` followed by `.toFixed(2)}` near a price variable
- `style: 'currency'` in `Intl.NumberFormat`

Fix any found occurrences using the shared utility.

## Rules

1. Never introduce a `currency` parameter — the format is always DA.
2. The shared utility in `src/lib/utils/format.ts` is the single source of truth.
3. Do not change any database values, seed data, or stored amounts — only the display formatting.
4. Do not change any business logic, only string formatting.

## Verify

After changes:
1. `pnpm tsc --noEmit` — no TypeScript errors
2. Check guest dashboard, orders, expenses pages — all show `DA 1,500` style amounts
3. Check admin operations and menu pages — same format
