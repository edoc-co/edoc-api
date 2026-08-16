# edoc — Encounter Spec

The encounter is edoc's atomic content unit. Everything — the fight screen, scoring, the mastery heatmap, spaced repetition, Campus reporting — reads from this shape. **Get it right before building anything that consumes it.**

---

## 1. Type definitions

```ts
type EncounterId = string;        // "py.core.loops.04"
type ConceptTag  = string;        // "for-loop", "range", "off-by-one"

type Difficulty = 'intro' | 'standard' | 'hard' | 'boss';

interface Encounter {
  id: EncounterId;
  language: string;               // "python"
  module: string;                 // "py.core" | "py.web"
  difficulty: Difficulty;
  conceptTags: ConceptTag[];      // drives mastery + spaced repetition

  brief: string;                  // markdown, shown above the editor
  starterCode: string;
  solutionCode: string;           // never sent to the client
  tests: Test[];
  hintCards: HintCard[];
  monster: Monster;
  failureMap: FailureRule[];      // makes "boss teaches you" possible

  artifact?: Artifact;            // boss encounters only
  estimatedMinutes: number;
}
```

### Test

```ts
interface Test {
  id: string;
  label: string;                  // "Handles an empty list" — shown to the user
  hidden: boolean;                // hidden tests only reveal pass/fail, not input
  damage: number;                 // HP removed from the monster when this passes
  conceptTags: ConceptTag[];      // which concept this test actually probes
}
```

Damage across all tests in an encounter must sum to exactly `monster.hp`. Enforce this in a content validation script — it is the most common authoring bug.

### HintCard

```ts
interface HintCard {
  id: string;
  title: string;
  body: string;                   // markdown, max ~240 chars — it's a card, not a page
  skeleton?: string;              // inserted on swipe-up. MUST contain blanks.
  conceptTags: ConceptTag[];
  cost: number;                   // shard cost in Ranked; 0 in Practice
}
```

**`skeleton` must never be a working solution.** Use `___` or `# your code here` placeholders. If the card writes the code, the learning didn't happen.

### Monster

```ts
interface Monster {
  id: string;
  name: string;                   // "The Loop King"
  hp: number;
  sprite: string;
  attacks: Attack[];
}

interface Attack {
  id: string;
  damage: number;                 // HP removed from the player
  message: string;                // in-character, but must name the real problem
  trigger: 'test-fail' | 'timeout' | 'runtime-error';
}
```

Attack `message` is flavor **plus** signal: "The Loop King coils tighter — your loop never terminates." Never flavor alone.

### FailureRule

This is the piece that makes defeat instructive rather than punishing.

```ts
interface FailureRule {
  match: {
    testId?: string;
    errorType?: string;           // "IndexError", "SyntaxError"
    pattern?: string;             // regex against stderr
  };
  concept: ConceptTag;
  lessonCardId: string;           // HintCard surfaced on defeat
  rematchVariant?: string;        // alternate params so memorizing fails
}
```

Rules are evaluated in order; first match wins. Every encounter needs a catch-all rule at the end.

### Artifact

```ts
interface Artifact {
  name: string;                   // "wordfreq CLI"
  description: string;
  files: { path: string; content: string }[];
  repoTemplate?: string;
}
```

Boss encounters only. This is what makes "resume disguised as a player card" literally true.

---

## 2. Runtime contract

The frontend talks to execution through exactly this interface. **Build the entire fight screen against a mock of it.**

```ts
interface RunRequest {
  encounterId: EncounterId;
  code: string;
  mode: 'practice' | 'ranked' | 'proctored';
}

interface RunResult {
  status: 'pass' | 'fail' | 'error' | 'timeout';
  results: TestResult[];
  stdout: string;
  stderr: string;
  durationMs: number;
}

interface TestResult {
  testId: string;
  passed: boolean;
  errorLine?: number;             // drives the in-editor damage highlight
  message?: string;
}

declare function runTests(req: RunRequest): Promise<RunResult>;
```

`errorLine` is what CodeMirror uses to flash the offending line. Without it the damage effect has nowhere to land.

**Practice** may resolve in-browser via WASM. **Ranked and Proctored must resolve server-side** — client results are never trusted for scoring.

---

## 3. Example encounter

```json
{
  "id": "py.core.loops.04",
  "language": "python",
  "module": "py.core",
  "difficulty": "standard",
  "conceptTags": ["for-loop", "accumulator", "off-by-one"],
  "brief": "Return the sum of every even number from 1 to n, inclusive.",
  "starterCode": "def sum_evens(n):\n    total = 0\n    # your code here\n    return total",
  "estimatedMinutes": 6,
  "monster": {
    "id": "loop-king", "name": "The Loop King", "hp": 100, "sprite": "loop_king",
    "attacks": [
      { "id": "coil", "damage": 15, "trigger": "test-fail",
        "message": "The Loop King coils tighter — your range stops one step short." },
      { "id": "crush", "damage": 30, "trigger": "timeout",
        "message": "Crushed. That loop never ends." }
    ]
  },
  "tests": [
    { "id": "t1", "label": "sum_evens(10) is 30", "hidden": false, "damage": 30, "conceptTags": ["for-loop"] },
    { "id": "t2", "label": "Includes n when n is even", "hidden": false, "damage": 40, "conceptTags": ["off-by-one"] },
    { "id": "t3", "label": "Returns 0 for n = 0", "hidden": true, "damage": 30, "conceptTags": ["accumulator"] }
  ],
  "hintCards": [
    { "id": "h1", "title": "range is exclusive", "body": "`range(1, 5)` yields 1,2,3,4 — the stop value is left out. To include n, use `range(1, n + 1)`.", "conceptTags": ["off-by-one"], "cost": 0 },
    { "id": "h2", "title": "Accumulator pattern", "body": "Start at zero outside the loop, add inside it.", "skeleton": "total = 0\nfor i in ___:\n    if ___:\n        total += ___", "conceptTags": ["accumulator"], "cost": 1 }
  ],
  "failureMap": [
    { "match": { "testId": "t2" }, "concept": "off-by-one", "lessonCardId": "h1", "rematchVariant": "n=14" },
    { "match": { "errorType": "TypeError" }, "concept": "accumulator", "lessonCardId": "h2" },
    { "match": {}, "concept": "for-loop", "lessonCardId": "h2" }
  ]
}
```

---

## 4. Authoring rules

1. Test damage sums to exactly `monster.hp`. Validate in CI.
2. Every encounter has at least one catch-all `failureMap` rule.
3. Test `label` describes behavior, never implementation: "Handles an empty list", not "assert f([]) == 0".
4. Hint skeletons always contain blanks.
5. `conceptTags` come from a controlled vocabulary per language. Free-text tags break the mastery heatmap.
6. Boss encounters must define an `artifact`.
7. 3–8 tests per encounter. Fewer feels arbitrary; more turns the HP bar into noise.

---

## 5. Why this shape

- `conceptTags` on both tests and encounters is what lets Campus mode show *which concept* a student is failing, not just which exercise.
- `failureMap` turns a loss into a lesson. Without it the monster can only say "wrong."
- `rematchVariant` stops players from memorizing an answer to beat a boss.
- Separating `tests[].damage` from `monster.hp` means the HP bar reflects genuine partial progress — the single strongest feedback signal in the fight loop.
