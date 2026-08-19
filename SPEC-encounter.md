# edoc — Encounter Spec

The encounter is edoc's atomic content unit. The fight screen, scoring, mastery heatmap, spaced repetition, and Campus reporting all read from this shape. **Shared contract between frontend and backend — neither side changes it alone.**

---

## 1. Types

```ts
type EncounterId = string;   // "py.core.loops.04"
type ConceptTag  = string;   // "for-loop", "off-by-one"
type Difficulty  = 'intro' | 'standard' | 'hard' | 'boss';
type Phase       = 'approach' | 'surface';

interface Encounter {
  id: EncounterId;
  language: string;              // "python"
  planet: string;                // "python-prime"
  module: string;                // "py.core" | "py.web"
  phase: Phase;                  // approach = journey obstacle; surface = planet module
  difficulty: Difficulty;
  conceptTags: ConceptTag[];

  brief: string;                 // markdown, shown above the console
  starterCode: string;
  solutionCode: string;          // never sent to the client
  tests: Test[];
  hints: Hint[];                 // delivered by the companion robot
  hostile: Hostile;
  failureMap: FailureRule[];

  artifact?: Artifact;           // boss encounters only
  estimatedMinutes: number;
}
```

### Test
```ts
interface Test {
  id: string;
  label: string;        // "Handles an empty list" — behaviour, never implementation
  hidden: boolean;      // hidden tests reveal pass/fail only
  damage: number;       // integrity removed from the hostile when this passes
  conceptTags: ConceptTag[];
}
```

Damage across all tests must sum to exactly `hostile.integrity`. Enforce in CI — most common authoring bug.

### Hint
Delivered by the companion robot, not as a card deck.
```ts
interface Hint {
  id: string;
  title: string;
  body: string;          // max ~240 chars — spoken by the companion
  skeleton?: string;     // inserted into the console. MUST contain blanks.
  conceptTags: ConceptTag[];
  cost: number;          // shards in Ranked; 0 in Practice
}
```

**`skeleton` must never be a working solution.** Use `___` or `# your code here`. If the hint writes the code, the learning didn't happen.

### Hostile
```ts
interface Hostile {
  id: string;
  name: string;          // "Drift Wraith"
  kind: 'alien' | 'ship' | 'hazard';
  integrity: number;
  sprite: string;        // Rive state machine: idle | hit | attack | destroyed
  attacks: Attack[];
}

interface Attack {
  id: string;
  damage: number;        // hull integrity removed from the player
  message: string;       // in character, but names the real problem
  trigger: 'test-fail' | 'timeout' | 'runtime-error';
}
```

Attack `message` is flavour **plus** signal: "The Wraith coils tighter — your range stops one step short." Never flavour alone.

### FailureRule
Turns defeat into a lesson.
```ts
interface FailureRule {
  match: { testId?: string; errorType?: string; pattern?: string };
  concept: ConceptTag;
  hintId: string;          // surfaced by the companion on defeat
  rematchVariant?: string; // alternate params so memorising fails
}
```
Evaluated in order, first match wins. Every encounter needs a catch-all last.

### Artifact
```ts
interface Artifact {
  name: string;          // "wordfreq CLI"
  description: string;
  files: { path: string; content: string }[];
  repoTemplate?: string;
}
```
Boss encounters only. This is what makes the certificate evidence-backed.

---

## 2. Planet & module structure

```ts
interface Planet {
  id: string;            // "python-prime"
  language: string;
  name: string;          // "Python Prime"
  worldType: 'temperate' | 'volcanic' | 'ice' | 'gas-giant' | 'ocean' | 'desert' | 'crystalline';
  accent: string;        // token key, not a hex
  tracks: string[];      // max 3
  features: PlanetFeature[];
}

interface PlanetFeature {
  id: string;
  moduleId: string;      // the module this feature represents
  label: string;         // "Northern Geyser Field"
  hotspot: { x: number; y: number };  // % coords on the surface scene
}
```

**Features must match the world type.** Ice worlds have geysers, crevasses, sub-surface caverns. Gas giants have storm bands and moons. Volcanic worlds have lava tubes and ash plains. Never apply Earth features to worlds that would not have them.

---

## 3. Runtime contract

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
  errorLine?: number;   // drives the in-console damage highlight
  message?: string;
}

declare function runTests(req: RunRequest): Promise<RunResult>;
```

`errorLine` is what CodeMirror uses to flash the offending line — without it the damage effect has nowhere to land.

**Practice** may resolve in-browser via WASM. **Ranked and Proctored must resolve server-side.** Client results are never trusted for scoring.

---

## 4. Authoring rules

1. Test damage sums to exactly `hostile.integrity`. Validate in CI.
2. Every encounter has a catch-all `failureMap` rule.
3. Test `label` describes behaviour, never implementation.
4. Hint skeletons always contain blanks.
5. `conceptTags` come from a controlled vocabulary per language — free text breaks the mastery heatmap.
6. Boss encounters define an `artifact`.
7. 3–8 tests per encounter.
8. Attack messages name the real problem, not just flavour.
9. `phase: 'approach'` encounters are the 1–2 obstacles before landing; everything else is `'surface'`.

---

## 5. Why this shape

- `conceptTags` on both tests and encounters lets Campus show *which concept* a student is failing, not just which exercise.
- `failureMap` turns a loss into a lesson — without it the hostile can only say "wrong."
- `rematchVariant` stops players memorising an answer.
- Separating `tests[].damage` from `hostile.integrity` means the bar reflects genuine partial progress, the strongest feedback signal in the loop.
- `phase` and `PlanetFeature` let the same encounter data drive both the journey and the surface map without forking content.
