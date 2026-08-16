# edoc — Product Spec

Product context for anyone (human or agent) working on this repo. Read this before `DESIGN.md`.

---

## 1. What edoc is

A gamified platform for learning programming. You pick a language, fight your way through it, and come out with proof you can actually code.

The core loop is combat: you write code in an editor, a monster's health drops when your tests pass, and it attacks when they fail. Micro-lessons arrive as swipeable cards under the editor. Winning drops cosmetic loot. Your profile is an eSports-style player card, not a résumé.

**The one-line pitch:** guided learning with an arcade combat loop, real multiplayer, and an evidence-backed certificate.

---

## 2. Who it's for

| Audience | What they want | Primary mode |
|---|---|---|
| Total beginners | Not to quit in week one | Practice |
| Hobbyists / enthusiasts | Loot, friends, streaks | Practice + clans |
| Working developers | Sharpening, competition | Ranked |
| Job seekers | A credential with evidence behind it | Proctored |
| Universities / colleges | Assignable curriculum + integrity | Campus |

One world, four modes. Do not build four products.

---

## 3. The four modes

**Practice** — no timer, no penalties, free hints, monster acts as tutor. Where beginners live. Never gated, never walled.

**Ranked** — PvP duels, code golf, bug-race, seasonal ladder, clan raids. Scored server-side.

**Proctored** — full-screen lock, external paste blocked, timed. **This is the only place lockdown applies.** Produces certificates.

**Campus** — instructor dashboard, roster import, assignments, per-concept mastery heatmaps, LMS grade passback (LTI). B2B. Build last, but design the data model for it now — cohorts and per-concept mastery are painful to retrofit.

---

## 4. Content model

Every language splits in two:

- **Core module** — syntax, control flow, data structures, functions, error handling, OOP. Finite and completable. Same shape for every language.
- **Applied modules** — where the language plugs into a stack. `Python Web`, `Python Data`, `JS DOM`, `JS Node`. Optional.

**Tracks** (frontend, backend, data, mobile) are curated *ordered sequences* of modules across languages — not tags. Example: Backend = Python Core → Python Web → SQL Core → APIs.

A language may appear in at most **3 primary tracks** in the UI.

### Certificates

| Certificate | Earned by | Weight |
|---|---|---|
| **Language** | Complete one language's Core module | Motivational milestone |
| **Stack** | Complete a full track | The one that carries weight |

Projects are **not required** for either. But a Stack Certificate earned *with* a project is visually distinct, links to the artifact, and ranks higher in the showcase.

Every certificate displays performance data, not just a completion date: `Cleared 148 encounters · 96% first-attempt`. That's what separates it from a completion checkbox.

---

## 5. Projects (optional at every tier)

Three tiers:

1. **Boss artifacts** — every boss fight already produces something real (a CLI, a parser, a small API). Automatic; no opt-in. Ensures everyone has a portfolio even if they never choose to build.
2. **Guided builds** — step-by-step, produces a shippable thing. Unlocked as loot ("blueprint drop") rather than assigned.
3. **Free build (the Workshop)** — blank multi-file sandbox, your own idea. Publishable to the showcase feed.

Optional but *visible*: the player card shows a conspicuously empty project slot, and no project means no presence in the showcase feed.

---

## 6. What makes edoc different from Codedex et al.

1. **Multiplayer** — duels, clan raids on a shared multi-file repo, ghost replays. Every competitor is single-player.
2. **Artifacts over badges** — the certificate links to working code.
3. **Failure is the content** — debugging *is* the combat system. Bugs are the lesson, not the punishment.

---

## 7. Screen layout

Three zones:

- **Zone A — Language coaster.** Scroll-driven 3D track of languages, categorized by track. Selection floods the UI with that language's accent color.
- **Zone B — Fight.** Editor + monster + HP bar + test output. The core screen.
- **Zone C — Hint deck.** Swipeable micro-lesson cards under the editor. Left discards, right saves, up inserts a code skeleton.

Plus a global ticker (lobby only, hidden during fights) and the player card profile.

---

## 8. Technical decisions already made

**Frontend:** Next.js (App Router) + Tailwind + Zustand + CodeMirror 6.

**Editor is CodeMirror 6, not Monaco** — 200KB vs 5MB, touch-viable, and runtime-swappable theming, which the loot economy depends on.

**Code execution is hybrid:**
- Browser WASM (Pyodide / QuickJS) for Practice — instant feedback, zero server cost
- Server-side execution for anything scored (Ranked, Proctored, certificates) — client-side results are never trusted

For MVP, use a hosted execution API rather than building container infrastructure.

**Anti-cheat is scoped to Proctored only.** Blocking paste globally punishes normal developer behavior (reading docs, adapting snippets) and leaks anyway. The real integrity signal is keystroke/process replay: a cheater's transcript is one giant paste, a learner's is iteration. Passive, and stronger.

---

## 9. Build order

1. One language deep — Python, ~150 encounters with real test suites
2. Fight loop + card deck (mocked runtime first)
3. Duels and clans — retention
4. More languages + the coaster
5. Proctored mode + certificates + artifact pages
6. Campus dashboard

---

## 10. Non-negotiables

- **Never gate practice.** Loot is gated behind keys; learning never is. The most motivated user must never hit a locked door.
- **Errors are not punishment.** The monster attacks on *failing tests*, not on syntax errors mid-keystroke. Reading a stack trace is the skill being taught.
- **On defeat, the boss teaches.** Show the concept that killed you, then rematch with different parameters so memorizing the answer doesn't work.
- **Accessibility is a launch requirement,** not a polish item. Universities will ask during procurement.
