# Bug Tracker

Bugs found via targeted scan of `interview-built-by-a-clanker/`. Ordered by real-world impact within each tier.

Legend: `[ ]` = open · `[x]` = fixed

---

## P0 — Build / TypeScript Errors

| # | Status | Description | File | Line |
|---|--------|-------------|------|------|
| 1 | [x] | `SearchParams` interface not exported — breaks `tsc` on frontend (`routeTree.gen.ts` TS4023) | `apps/web/src/routes/index.tsx` | 9 |

---

## P1 — Critical: Data Integrity / Security / Broken Core Flows

| # | Status | Description | File | Line |
|---|--------|-------------|------|------|
| 2 | [x] | Auth middleware is a no-op by default (`ENFORCE_AUTH` env not set) — skips JWT verification, `request.user` is `undefined`, crashes ALL protected routes with 500 instead of 401 | `apps/api/src/middleware/auth.ts` | 3–11 |
| 3 | [x] | Login response omits `username` — register works, login broken; users get an incomplete session, navbar shows `undefined` after login | `apps/api/src/routes/auth.ts` | 62–65 |
| 4 | [x] | Checkout does not clear cart after order placed — `db.cart.clearForUser()` exists but is never called; users can re-checkout the same items, leading to duplicate orders | `apps/api/src/routes/checkout.ts` | 50–52 |
| 5 | [x] | Favorite toggle logic inverted — calls `DELETE` when NOT favorited, `POST` when already favorited; actively corrupts favorites in the DB | `apps/web/src/routes/personas/$personaId.tsx` | 42–44 |
| 6 | [x] | `DELETE /cart/:itemId` missing ownership check — any authenticated user can delete any other user's cart item by ID (IDOR, DB corruption) | `apps/api/src/routes/cart.ts` | 73–85 |
| 7 | [x] | `PersonaCard` multiplies price by 100 — displays `$4999.00` instead of `$49.99` on browse and favorites pages; storefront is unusable | `apps/web/src/components/PersonaCard.tsx` | 63 |

---

## P2 — High: Significant UX / Security Issues (No DB Corruption)

| # | Status | Description | File | Line |
|---|--------|-------------|------|------|
| 8 | [x] | Two queries share key `["favorites"]` but return different shapes (`string[]` vs `{ favorites: Persona[] }`) — cache collision causes `isFavorited` to always be `false` depending on page visit order | `apps/web/src/routes/personas/$personaId.tsx` line 22 vs `apps/web/src/routes/favorites.tsx` line 16 | — |
| 9 | [x] | `logout()` does not remove token from localStorage — on page refresh after logout, old token is re-read and user is silently re-authenticated | `apps/web/src/lib/auth.tsx` | 55–58 |
| 10 | [ ] | React Query key `["personas"]` is static — changing any filter/search param does NOT trigger a refetch; all filtering appears broken to the user | `apps/web/src/routes/index.tsx` | 43 |
| 11 | [ ] | CartItem decrement button has no `disabled` prop at `quantity === 1` — sends `quantity: 0` to API which rejects with 400; button appears clickable but errors silently | `apps/web/src/components/CartItem.tsx` | 41 |
| 12 | [ ] | Cart count in navbar uses `queryKey: ["cart-count"]` but cart mutations invalidate `["cart"]` — nav badge is perpetually stale after any cart change | `apps/web/src/routes/__root.tsx` | 16 |

---

## P3 — Medium: Correctness / UX Issues

| # | Status | Description | File | Line |
|---|--------|-------------|------|------|
| 13 | [ ] | `minPrice` filter uses `<=` instead of `>=` — returns cheap items when expensive ones are requested; display/filter only, no DB impact | `apps/api/src/db.ts` | 365 |
| 14 | [ ] | React Query cache not cleared on logout — next user to log in on same tab briefly sees prior user's `cart` and `favorites` data | `apps/web/src/lib/auth.tsx` + `apps/web/src/routes/__root.tsx` | 55–58 / 84–87 |
| 15 | [ ] | `q=` search does not include `specialty` field — `?q=security` misses Compliance Carl; display/filter only, no DB impact | `apps/api/src/db.ts` | 345–354 |

---

## P4 — Low: Code Quality / Security Hygiene

| # | Status | Description | File | Line |
|---|--------|-------------|------|------|
| 16 | [ ] | `GET /favorites` handler bypasses `db` abstraction and reads raw Map stores directly — inconsistent with all other route handlers | `apps/api/src/routes/favorites.ts` | 3, 11–18 |
| 17 | [ ] | Cart and order IDs are sequential integers (`cart-1`, `cart-2`) — trivially enumerable, amplifies IDOR risk of bug #6 | `apps/api/src/db.ts` | 20–25 |

---

## Stats
- Total bugs: 17
- P0: 1 · P1: 6 · P2: 5 · P3: 3 · P4: 2
- Fixed: 9 / 17
