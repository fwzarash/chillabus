# Architecture Context

> Build-phase plan. This round is UI-only (Canva), so none of this is built yet.

## Stack

| Layer     | Technology                          | Role                                                        |
| --------- | ----------------------------------- | ----------------------------------------------------------- |
| Framework | Next.js (App Router) + TypeScript   | Frontend, API, and the assistant loop in one repo           |
| Hosting   | Vercel                              | Deploy, HTTPS by default, PWA delivery                      |
| UI        | Mobile-first PWA (React)     | Phone-first screens; component/styling choice open          |
| Auth      | Clerk                               | Sign-in, prebuilt components, Vercel-native                 |
| Database  | Neon (Postgres) + Drizzle ORM       | Users, tasks, check-ins, area weights                       |
| AI        | OpenAI API (tool calling)           | Assistant that runs the feature tools, server-side          |
| Voice     | Web Speech API + text fallback      | Speech-to-text input; server STT is the upgrade path        |

## System Boundaries

- `app/` - Next.js routes: pages and the assistant route handlers.
- `app/api/` - server-only endpoints, including the OpenAI agent loop (key never reaches the client).
- `lib/` - capacity computation, tool implementations (add/adjust task, rebalance, suggest recovery).
- `db/` - Drizzle schema and queries against Neon.

## Storage Model

- **Database (Neon Postgres)**: users, tasks/commitments (area, effort, priority, due), daily check-ins, per-area weights. Source of truth for capacity.
- No blob or file storage needed yet.

## Auth and Access Model

- Every user signs in via Clerk.
- The Clerk user id is stored as a foreign key on the user's rows in Neon (auth and data are separate vendors, so we join them ourselves).
- A user reads and writes only their own tasks, check-ins, and weights.

## Invariants

1. The OpenAI key lives server-side only. The browser never calls OpenAI directly.
2. Capacity is always derived from stored check-ins and tasks, never hand-set.
3. Voice always has a working text fallback (iOS Safari speech support is unreliable).
4. No guilt mechanics: no streaks, no broken-chain penalties, nothing that adds load to the user.
