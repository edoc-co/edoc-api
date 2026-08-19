# edoc — Product Spec

Product context. Read this before `DESIGN.md` or `SPEC-encounter.md`.

---

## 1. What edoc is

A platform for learning to code, built as a space exploration game.

You board a ship. You pick a character. The galaxy map is a solar system where **every planet is a programming language**. You choose a destination and fly there — and the journey is the learning. Obstacles en route (meteor fields, hull breaches, hostile ships) are coding encounters: you write code in the ship's console, working code fires, broken code takes damage. Clear the approach and you land, then explore the planet's surface, where **each natural feature of that world is a module**.

Your companion robot delivers hints. Your progress is a pilot profile. Completing a language earns a certificate backed by real performance data and working code.

**One line:** learning to code as a space voyage — with real multiplayer, real artifacts, and a credential that means something.

---

## 2. Who it's for

| Audience | Wants | Mode |
|---|---|---|
| Beginners | Not to quit in week one | Practice |
| Hobbyists | Loot, crew, streaks | Practice + crews |
| Working developers | Sharpening, competition | Ranked |
| Job seekers | A credential with evidence | Proctored |
| Universities | Assignable curriculum + integrity | Campus |

One universe, four modes. Not four products.

---

## 3. The four modes

**Practice** — no timer, no penalties, free hints, companion acts as tutor. Never gated, never walled.

**Ranked** — ship duels, crew battles, seasonal ladder. Scored server-side.

**Proctored** — fullscreen lock, external paste blocked, timed, exit penalties. **The only place lockdown applies.** Produces certificates. This is what universities pay for.

**Campus** — instructor dashboard, rosters, assignments, per-concept mastery heatmaps, LMS grade passback. Build last; design the data model now.

---

## 4. The journey structure

```
Universe map  →  choose a planet (language)
     ↓
Approach      →  1–2 encounters as obstacles (meteors, hostiles, breakdowns)
     ↓
Landing       →  cinematic, skippable
     ↓
Surface       →  planet features = modules; explore or go straight to work
     ↓
Complete      →  language certificate, next planet unlocks
```

### Planets are languages

| Language | World | Feel |
|---|---|---|
| Python | Temperate, canyon-carved | Approachable, warm |
| JavaScript | Volcanic, restless | Fast, chaotic |
| TypeScript | Crystalline, ordered | Structured, cold |
| Go | Ice world, geysers | Clean, efficient |
| Rust | Iron desert, storms | Harsh, precise |
| SQL | Ocean world, deep trenches | Vast, layered |

**Planet features are specific to the world, never generic.** A gas giant's modules are storm bands and moons. An ice world's are crevasses, geysers, and sub-surface caverns. A volcanic world's are lava tubes and ash plains. A ringed world's are ring segments and shepherd moons. Do not apply Earth features (trees, fruit, flowers) to worlds that would not have them — that was an illustrative example only.

Each planet needs its own feature vocabulary mapped to its modules, defined alongside its content.

---

## 5. Battles

### Solo
Alien or environmental hazard. Write code against a timer. Working code fires; failing tests are counter-attacks that damage your hull.

### Duel (1v1)
Ship versus ship. Two formats:
- **Race** — one problem, first to solve wins
- **Gauntlet** — 2–3 problems, with **live opponent progress visible** ("they're on 2, you're on 1"), plus chat and quick reactions. BGMI-style awareness is the point.

### Crew (team)
4 players per ship, 8 per match; scales to 25 crews per 100 players. The crew shares a ship and splits the problem set. Boarding is a scene — crew take stations, can talk, tour the ship, or skip straight in.

---

## 6. Characters and the companion

**Character:** chosen at first run — base explorer models (boy/girl). Cosmetics unlock through play: helmets, suits, visors, patches, colourways. Premium characters unlock at higher tiers.

**All characters must be original.** No licensed IP — no Marvel, Star Wars, or any existing franchise character, as reference tone or as an unlockable. Guardians of the Galaxy and Star Wars work as *mood* references only: scrappy crew, humour, worn-in retro-futurism.

**Companion robot:** earned after a points threshold. Delivers hints, explanations, and code skeletons from a fixed spot on the console. Has a name and a personality. This replaces the earlier "hint card deck" — same function, character attached.

---

## 7. Content model

Every language splits in two:

- **Core module** — syntax, control flow, data structures, functions, error handling, OOP. Finite and completable, same shape for every language.
- **Applied modules** — where the language plugs into a stack (`Python Web`, `JS DOM`). Optional.

**Tracks** (frontend, backend, data, mobile) are curated ordered sequences across languages, not tags. A language appears in at most 3 primary tracks.

### Certificates

| Certificate | Earned by | Weight |
|---|---|---|
| **Language** | Complete one language's Core | Motivational milestone |
| **Stack** | Complete a full track | Carries real weight |

Projects are **not required** for either, but a Stack Certificate earned with a project is visually distinct, links to the artifact, and ranks higher in the showcase.

Every certificate shows performance data: `Cleared 148 encounters · 96% first-attempt`.

---

## 8. Projects (optional at every tier)

1. **Salvage artifacts** — every boss encounter produces something real automatically. No opt-in required, so everyone has a portfolio.
2. **Guided builds** — step-by-step, unlocked as loot ("blueprint recovered"), never assigned.
3. **Free build (the Workshop)** — blank multi-file sandbox, publishable to the showcase.

Optional but visible: the pilot profile shows a conspicuously empty project slot, and no project means no presence in the showcase.

---

## 9. Sound

Present throughout with a **global on/off toggle**, defaulting to off with a single prompt to enable.

Ship hum, console clicks, hull impacts, engine burns, docking, loot chimes. Typed-text sequences carry keyboard sound.

**Silent while coding.** No audio while the console has focus. Same principle as the visual workspace rule.

---

## 10. What makes edoc different

1. **Multiplayer** — duels, crew battles, live opponent awareness, ghost replays. Every competitor is single-player.
2. **Artifacts over badges** — the certificate links to working code.
3. **Failure is the content** — debugging *is* the combat system.
4. **The journey is the curriculum** — progression is spatial and legible, not a progress bar.

---

## 11. Technical decisions

**Frontend:** Next.js (App Router) + Tailwind + Zustand + CodeMirror 6.

**Editor is CodeMirror 6, not Monaco** — 200KB vs 5MB, touch-viable, runtime-swappable theming.

**The universe is 2.5D, not real-time 3D.** This is the single most important scoping decision:

| Feels like | Actually is |
|---|---|
| 3D cockpit | 4–5 parallax layers on mouse and scroll |
| Flying to a planet | Pre-rendered video, skippable |
| Boarding, taking stations | Pre-rendered cutscene, skippable |
| Explorable surface | Illustrated scene with hotspots |
| Character customization | 2D character with layered sprites |
| Companion robot | Rive state machine |

Real-time 3D buys free camera movement, which this product does not need. Parallax runs at 60fps anywhere; WebGL does not.

**Code execution is hybrid:** browser WASM for Practice (instant, free), server-side for anything scored. Client results are never trusted. Use a managed execution service (Piston/Judge0) for MVP — do not build sandboxing in-house.

**Anti-cheat is Proctored-only.** Fullscreen API plus Page Visibility API detect exits; violations are logged and penalised. Practice and Ranked stay completely open — blocking paste globally punishes normal developer behaviour and leaks anyway. The stronger integrity signal is keystroke replay: a cheater's transcript is one paste, a learner's is iteration.

---

## 12. Non-negotiables

- **Never gate practice.** Loot is gated; learning never is.
- **Errors are not punishment.** Damage comes from *failing tests*, not syntax errors mid-keystroke.
- **On defeat, teach.** Show the concept that killed you, then rematch with different parameters.
- **The console stays calm.** When someone is writing code, the ship goes quiet — visually and audibly.
- **All assets original.** No licensed characters, ships, or worlds.
- **Accessibility is a launch requirement.** Universities ask during procurement.

---

## 13. Scope warning

The full vision — real-time 3D, character models, explorable surfaces, cinematic sequences, team multiplayer — is a game studio project: 18–24 months, 8–12 people including 3D artists and a gameplay engineer.

The 2.5D approach in §11 brings a credible v1 into range, but this is still meaningfully larger than a website. Any timeline given to a client must reflect the scope actually being built, and content authoring (~150 encounters per language, 8–10 weeks each) remains the critical path regardless of engineering speed.
