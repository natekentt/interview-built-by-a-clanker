# CLAUDE.md — ECI Final Interview

## Project Location
All code lives in `interview-built-by-a-clanker/` — a Turborepo + pnpm monorepo.

## Architecture Overview

```
interview-built-by-a-clanker/
├── apps/api/          # Fastify 5 REST API (backend, port 3001)
├── apps/web/          # React 19 SPA (frontend, port 5173)
├── packages/shared/   # Shared Zod schemas + TypeScript types
├── turbo.json         # Task orchestration (build order, caching)
├── pnpm-workspace.yaml
└── tsconfig.base.json
```

**Shared package** (`@acme/shared`) must be built before apps — Turborepo handles this via `"dependsOn": ["^build"]`.

## How to Build & Run

```bash
cd interview-built-by-a-clanker
pnpm install
pnpm build     # builds shared → api → web (in order)
pnpm dev       # starts all packages concurrently
```

- Frontend: http://localhost:5173
- API: http://localhost:3001

Per-package scripts via `pnpm --filter <name> <script>`, e.g.:
```bash
pnpm --filter @acme/api dev
pnpm --filter @acme/web build
```

Turborepo tasks: `build`, `dev`, `lint`, `typecheck`, `clean`.

## Frontend (`apps/web`)

| Tool | Version |
|------|---------|
| React | 19 |
| TanStack Router | 1.92 (file-based routing) |
| TanStack Query | 5.62 (server state) |
| Tailwind CSS | 4.0 (Vite plugin) |
| Vite | 6.0 |
| TypeScript | 5.7 |

**Routing**: File-based via `src/routes/`. `routeTree.gen.ts` is auto-generated — do not edit.

**Path alias**: `~/*` → `src/*`

**Key files:**
- `src/lib/api.ts` — fetch wrapper that injects `Authorization: Bearer` header
- `src/lib/auth.tsx` — `AuthProvider` / `useAuth` context (JWT stored in localStorage)
- `src/lib/queryClient.ts` — React Query client config
- `src/routes/__root.tsx` — root layout with nav

**Routes:** `/` (personas list), `/login`, `/register`, `/cart`, `/checkout`, `/favorites`, `/personas/$personaId`

## Backend (`apps/api`)

| Tool | Version |
|------|---------|
| Fastify | 5.2 |
| @fastify/jwt | 9.0 |
| @fastify/cors | 10.0 |
| TypeScript | 5.7 |

**Database**: In-memory only (`src/db.ts`) — Maps/Sets, no external DB, seeded with 15 personas.

**JWT secret**: `agentic-personas-dev-secret` (hardcoded for dev)

**CORS origin**: `http://localhost:5173`

**Auth enforcement**: `ENFORCE_AUTH=true` env var (defaults false) — enables JWT requirement on protected routes.

**Route files:**
- `src/routes/auth.ts` — `POST /auth/register`, `POST /auth/login`, `GET /auth/me`
- `src/routes/personas.ts` — `GET /personas`, `GET /personas/:id`
- `src/routes/cart.ts` — `GET/POST /cart`, `PUT/DELETE /cart/:itemId`
- `src/routes/favorites.ts` — `GET/POST /favorites`, `DELETE /favorites/:personaId`
- `src/routes/checkout.ts` — `POST /checkout`
- `src/middleware/auth.ts` — JWT middleware

**Dev**: `tsx watch src/index.ts` | **Prod**: `tsc` then `node dist/index.js`

## Shared Package (`packages/shared`)

Zod schemas + inferred TypeScript types. Validated by both frontend and backend.

```
src/schemas/
├── auth.ts      # registerSchema, loginSchema, userSchema
├── persona.ts   # personaSchema, personaFilterSchema
├── cart.ts      # cartItemSchema, addToCartSchema
└── order.ts     # checkoutSchema, orderSchema
```

Must run `pnpm build` after changes so apps pick up new `dist/`.

## TypeScript Configuration

**Strict mode is ON** across all packages. Base config at `tsconfig.base.json`:
- `target: ES2022`, `module: ESNext`, `moduleResolution: bundler`
- `strict: true`, `isolatedModules: true`, `esModuleInterop: true`
- `declaration: true`, `declarationMap: true`, `sourceMap: true`

Web overrides: `jsx: "react-jsx"`, `noEmit: true`

Run typechecks: `pnpm typecheck` (via Turborepo, respects build order)

## Linting / Formatting

**No ESLint, Prettier, or Biome configured.** TypeScript strict mode is the primary code quality gate.

## No Testing Framework

No Jest, Vitest, or testing-library configured.

## No Docker / No External Services

No Dockerfiles, no docker-compose, no external database, no CI/CD config.

## Data Model Highlights

**Persona**: `{ id, name, tagline, description, avatarUrl, specialty, capabilities[], price, rating, reviewCount, tier }`
- Specialties: Engineering, Design, Data, Security, DevOps, Product
- Tiers: Starter | Pro | Enterprise

**User**: `{ id, email, name }` + hashed password stored separately

**Cart**: `Map<cartItemId, { userId, personaId, quantity }>`

**Orders**: `Map<orderId, { userId, items[], total, customerInfo, createdAt }>`

## Bug Tracking

All discovered bugs are tracked in `./bugs.md` (next to this file).

**When to write to bugs.md:**
- Any time a new bug is discovered (during code review, testing, or fixing)
- When a bug is fixed — update its status from `[ ]` to `[x]`
- Bugs are prioritized P0–P4 (P0 = build errors, P1 = critical, P2 = high, P3 = medium, P4 = low/hygiene)
- Always include: description, file path, line number(s), and status

## Context: Purpose of This Repo

Debugging assessment — bugs were intentionally introduced across frontend, backend, and shared packages. The app is a storefront for AI "Agentic Personas."
