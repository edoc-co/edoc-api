# CLAUDE.md

Backend API for **edoc**, a gamified programming-learning platform. Read `PROJECT.md` for product context and `SPEC-encounter.md` for the data contract before writing anything.

There is a separate frontend repo (`edoc-web`) built against the contract in `SPEC-encounter.md`. That file is shared and authoritative.

## Stack

Node 20 · TypeScript · Fastify · Postgres · Drizzle ORM · Zod · Vitest

## Layout

```
src/
  routes/         HTTP handlers, thin — no business logic
  services/       business logic
  db/             schema, migrations, queries
  runtime/        execution provider adapter
  auth/           sessions, OAuth
  lib/            shared utilities
tests/
```

## The contract — do not break it

`POST /run` must accept `RunRequest` and return `RunResult` exactly as defined in `SPEC-encounter.md` — same field names, same shapes, same status values. The frontend is built against a mock of this.

**Never change `SPEC-encounter.md` unilaterally.** If a field is needed, say so and stop; it gets agreed with the frontend first, then both copies update together.

## Security rules

- **Never build sandboxing in-house.** `/run` delegates to a managed execution service (Piston or Judge0) behind an adapter in `src/runtime/`. The rest of the codebase must not know which provider is in use.
- **Never trust client-reported results.** Practice mode may execute in the browser, but anything scored — Ranked, Proctored, certificates, mastery — is recomputed server-side from a server-side run.
- **Never send `solutionCode` to the client.** Strip it, plus hidden test inputs and expected outputs, in the serializer. Hidden tests return pass/fail and label only.
- Rate limit `/run` per user. Enforce timeout and memory caps on every execution.
- Validate every request body with Zod at the route boundary.
- Secrets from environment only. Never commit `.env`.

## Data model notes

- Progress is per user, per encounter: attempts, pass/fail, first-attempt success, duration.
- **Mastery is derived per `conceptTag`, not per encounter.** This is what Campus mode reports on later — model it now, it is painful to retrofit.
- Design for cohorts and assignments from the start even though Campus ships last. A nullable `cohortId` now costs nothing.
- Certificates are issued from verified server-side runs only, and store the performance snapshot at issue time.

## Conventions

- All timestamps UTC, stored as `timestamptz`.
- IDs are UUIDs except encounter IDs, which are the string keys from `SPEC-encounter.md` (`py.core.loops.04`).
- Errors return `{ error: { code, message } }`. Codes are stable strings; messages are for humans.
- Every migration is reversible. Never edit an applied migration.
- Tests for every service function and every route. Run `npm test` before saying a task is done.

## Working style

- One task at a time. Stop and report when done; do not roll into the next thing.
- Prefer editing existing files over creating new ones.
- No new dependencies without asking.
- No README, no summary docs, no comments explaining obvious code.
- If a request conflicts with `SPEC-encounter.md` or the security rules above, say so before proceeding.
