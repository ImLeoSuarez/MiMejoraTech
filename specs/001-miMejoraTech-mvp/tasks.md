# Tasks — MiMejoraTech MVP (Spec 001)

## Phase 1: Project Setup

- [x] **T01** Initialize npm project with `package.json` (name, version, scripts, engines)
  - FRs: NFR-4, NFR-5
  - Done when: `npm install` succeeds and `package.json` has all required scripts (dev, build, preview, typecheck, lint, format, format:check, test, test:watch, test:e2e)

- [x] **T02** Configure TypeScript (`tsconfig.json`) with strict mode, ES2022, bundler module resolution, DOM types
  - FRs: Constitution §3, NFR-3
  - Done when: `npm run typecheck` passes on empty project

- [x] **T03** Configure Vite (`vite.config.ts`) for vanilla TS, Sass, dev server, build output
  - FRs: NFR-4, NFR-5
  - Done when: `npm run dev` starts server; `npm run build` produces `dist/`

- [x] **T04** Configure ESLint (`eslint.config.js`) with TypeScript, recommended rules
  - FRs: Constitution §8, §12
  - Done when: `npm run lint` passes on empty project

- [x] **T05** Configure Prettier (`.prettierrc`) with 2-space indent, single quotes, semicolons, 100-char width
  - FRs: Constitution §8, §12
  - Done when: `npm run format:check` passes on empty project

- [x] **T06** Configure Vitest (`vitest.config.ts`) for unit/integration tests, coverage
  - FRs: NFR-6, Constitution §10, §12
  - Done when: `npm run test` runs (0 tests, passes)

- [x] **T07** Configure Playwright (`playwright.config.ts`) for Chromium, Firefox, WebKit; mobile + desktop viewports
  - FRs: Constitution §11, §12, NFR-1, NFR-2
  - Done when: `npm run test:e2e` runs (0 tests, passes)

- [x] **T08** Create directory structure per plan (src/, data/, tests/, styles/)
  - FRs: Architecture
  - Done when: All directories exist and are tracked by git

---

## Phase 2: Data Model & Schema

- [ ] **T09** Create `src/features/questionnaire/types.ts` with Question, Option, Service, Benefit, ContactCard, QuestionnaireData interfaces
  - FRs: FR-1, FR-2, FR-3, FR-4, FR-5, FR-6
  - Done when: TypeScript compiles; types match plan §2

- [ ] **T10** Create `src/features/questionnaire/schema.ts` with JSON Schema (or Zod/Arktype) for QuestionnaireData validation
  - FRs: FR-7
  - Done when: Schema validates example from plan §2; rejects missing required fields

- [ ] **T11** Create `data/questionnaire.json` with seed content from spec Appendix (5 questions, 13 services, 12 benefits, mappings, contact card)
  - FRs: FR-1, FR-5, FR-6, FR-9
  - Done when: File exists; `npm run typecheck` passes; JSON is valid

- [ ] **T12** Create `src/features/questionnaire/loader.ts` — async load + schema validation; throws typed error on failure
  - FRs: FR-1, FR-7
  - Done when: Loads `questionnaire.json`; returns typed `QuestionnaireData`; throws on missing/malformed file

- [ ] **T13** Unit tests: `loader.test.ts` — valid load, missing file, schema violation, malformed JSON
  - FRs: FR-1, FR-7
  - Done when: `npm run test` passes; 4+ tests covering success + error cases

---

## Phase 3: Assessment Engine (Pure TS, DOM-independent)

- [ ] **T14** Create `src/features/assessment/types.ts` with Service, Benefit, Mapping, AssessmentResult types
  - FRs: FR-14, FR-15, FR-16, FR-19
  - Done when: Types match plan §3; TypeScript compiles

- [ ] **T15** Create `src/features/assessment/deduplication.ts` — `unionUnique`, `deduplicateById` helpers using Set
  - FRs: FR-16
  - Done when: Functions export; unit tests pass (see T16)

- [ ] **T16** Unit tests: `deduplication.test.ts` — union, deduplication, stable ordering
  - FRs: FR-16
  - Done when: `npm run test` passes; edge cases covered (empty, duplicates, order)

- [ ] **T17** Create `src/features/assessment/engine.ts` — `computeAssessment(selectedOptions, data)` per plan §3 pseudocode
  - FRs: FR-14, FR-15, FR-16, FR-19
  - Done when: Function exports; pure (no DOM, no side effects); returns AssessmentResult

- [ ] **T18** Unit tests: `engine.test.ts` — mapping union, benefit derivation, deduplication, empty result fallback
  - FRs: FR-14, FR-15, FR-16, FR-19
  - Done when: `npm run test` passes; 5+ tests with fixtures covering all branches

---

## Phase 4: State & Router

- [ ] **T19** Create `src/app/state.ts` — `QuestionnaireState` class: answers Map, currentIndex, sessionStorage sync (save/load/clear)
  - FRs: FR-12, US-8, FR-13
  - Done when: Methods: `getAnswer`, `setAnswer`, `clear`, `save`, `load`; persists to sessionStorage

- [ ] **T20** Unit tests: `state.test.ts` — answer storage, retrieval, clear, sessionStorage round-trip
  - FRs: FR-12, US-8
  - Done when: `npm run test` passes; sessionStorage mock verified

- [ ] **T21** Create `src/app/router.ts` — `Router` class: `next()`, `prev()`, `restart()`, `goTo(index)`, `getCurrentQuestion()`, `canGoNext()`, `canGoPrev()`
  - FRs: FR-8, FR-12, FR-13
  - Done when: Navigation respects bounds; `restart` clears state; index management correct

- [ ] **T22** Unit tests: `router.test.ts` — next/prev/restart, answer preservation, index bounds, validation gating
  - FRs: FR-8, FR-10, FR-11, FR-12, FR-13
  - Done when: `npm run test` passes; integrates with State + Validation

---

## Phase 5: UI Components

- [ ] **T23** Create `src/utils/dom.ts` — safe DOM helpers: `createElement`, `queryRequired`, `on`, `off`, `toggleClass`
  - FRs: NFR-4
  - Done when: Helpers typed; no `any`; used by components

- [ ] **T24** Create `src/utils/i18n.ts` — Spanish string constants for validation, fallback, UI labels
  - FRs: FR-20, Constitution §9
  - Done when: All user-facing strings centralized; no hardcoded Spanish in components

- [ ] **T25** Create `src/components/error-message.ts` — renders friendly Spanish error from loader failure
  - FRs: FR-7, FR-20
  - Done when: Component mounts; displays message; accessible (role=alert)

- [ ] **T26** Create `src/features/questionnaire/validation.ts` — `validateAnswer(question, selectedOptions)` returns `{ valid: boolean; message?: string }`
  - FRs: FR-10, FR-11
  - Done when: Single-select required → invalid if empty; multi-select → valid if empty; message in Spanish

- [ ] **T27** Unit tests: `validation.test.ts` — single required empty, single answered, multi empty, multi answered
  - FRs: FR-10, FR-11
  - Done when: `npm run test` passes; 4+ cases

- [ ] **T28** Create `src/components/progress-bar.ts` — renders progress (current/total), updates on index change
  - FRs: FR-9
  - Done when: Component renders; `update(current, total)` works; accessible (aria-valuenow)

- [ ] **T29** Create `src/components/question-screen.ts` — renders question text, options (radio/checkbox), validation message, continue/back buttons
  - FRs: FR-8, FR-10, FR-11
  - Done when: Single-select uses radio; multi uses checkbox; validation blocks continue; back enabled after Q1

- [ ] **T30** Create `src/components/contact-card.ts` — renders name, email, WhatsApp from data
  - FRs: FR-6, FR-18
  - Done when: Component renders all 3 fields; links use mailto:/https://wa.me/

- [ ] **T31** Create `src/components/result-screen.ts` — renders service list with benefits, contact card, fallback message
  - FRs: FR-17, FR-18, FR-19
  - Done when: Services + benefits listed; deduplicated; fallback shows when empty; contact card included

---

## Phase 6: Styles (Mobile-First, Sass)

- [ ] **T32** Create `src/styles/_tokens.scss` — colors, spacing, breakpoints (mobile-first), typography, focus ring
  - FRs: NFR-1, NFR-2
  - Done when: Tokens defined; used by component styles

- [ ] **T33** Create `src/styles/_global.scss` — reset, base typography, box-sizing, reduced-motion media query
  - FRs: NFR-1, NFR-2
  - Done when: Global styles applied; `prefers-reduced-motion` disables transitions

- [ ] **T34** Create `src/styles/_utilities.scss` — container, sr-only, touch-target (44px), focus-visible, spacing utilities
  - FRs: NFR-1, NFR-2
  - Done when: Utilities work; touch targets ≥44px verified

- [ ] **T35** Create `src/styles/components/_question-screen.scss` — BEM styles for question card, options, buttons, validation
  - FRs: NFR-1, NFR-2
  - Done when: Mobile layout works; options stacked; buttons full-width on mobile

- [ ] **T36** Create `src/styles/components/_progress-bar.scss` — progress bar styles, responsive
  - FRs: NFR-1
  - Done when: Bar fills proportionally; visible on all viewports

- [ ] **T37** Create `src/styles/components/_result-screen.scss` — service list, benefit tags, fallback, contact card styles
  - FRs: NFR-1, NFR-2
  - Done when: Result readable on mobile; benefits as tags; contact card prominent

- [ ] **T38** Create `src/styles/components/_contact-card.scss` — contact card layout, link styles
  - FRs: NFR-1
  - Done when: Card displays cleanly; links touch-friendly

- [ ] **T39** Create `src/styles/main.scss` — imports tokens, global, utilities, components
  - FRs: NFR-1, NFR-2
  - Done when: Single import in `main.ts` applies all styles

---

## Phase 7: App Entry & Wiring

- [ ] **T40** Create `src/app/main.ts` — bootstrap: load data, init state+router, mount first screen, handle errors
  - FRs: FR-1, FR-7, FR-8, FR-13
  - Done when: App loads; shows Q1 on success; shows error screen on data failure

- [ ] **T41** Integration test: `flow.test.ts` — full flow: answer all → result → restart
  - FRs: FR-1–13, FR-17–19
  - Done when: `npm run test` passes; simulates user journey via router+state+components

- [ ] **T42** Integration test: `assessment-integration.test.ts` — loader + engine with real data file
  - FRs: FR-1, FR-14–16
  - Done when: `npm run test` passes; real `questionnaire.json` produces expected services

- [ ] **T43** Integration test: malformed data → friendly error screen
  - FRs: FR-7, FR-20
  - Done when: `npm run test` passes; corrupt JSON shows Spanish error, not crash

---

## Phase 8: End-to-End Tests (Playwright)

- [ ] **T44** E2E: `happy-path.spec.ts` — load → answer 5 questions → verify services/benefits/contact → restart
  - FRs: FR-1–21
  - Done when: `npm run test:e2e` passes; runs on Chromium, Firefox, WebKit

- [ ] **T45** E2E: `back-navigation.spec.ts` — Q3 → back to Q2 → change → forward → verify preserved → result
  - FRs: FR-8, FR-12
  - Done when: `npm run test:e2e` passes; answer persistence verified

- [ ] **T46** E2E: `fallback.spec.ts` — select options mapping to no services → verify fallback + contact card
  - FRs: FR-19, FR-18
  - Done when: `npm run test:e2e` passes; fallback message in Spanish

- [ ] **T47** E2E: accessibility audit — axe-core on all screens; touch targets ≥44px; focus visible; reduced-motion
  - FRs: NFR-2
  - Done when: `npm run test:e2e` passes; 0 a11y violations; mobile viewport tested

---

## Phase 9: Polish & Verification

- [ ] **T48** Run full quality gate: `npm run typecheck && npm run lint && npm run format:check && npm run test && npm run test:e2e && npm run build`
  - FRs: All; Constitution §12
  - Done when: All commands exit 0; `dist/` produced

- [ ] **T49** Cross-browser visual check: open `npm run preview` in Chrome, Firefox, Safari (mobile + desktop)
  - FRs: NFR-1, NFR-2
  - Done when: No layout breaks; touch targets work; text readable

- [ ] **T50** Final review: verify no hardcoded Spanish in TS (only in data/i18n), no `any`, no console.log, no secrets
  - FRs: Constitution §3, §8, §9, §13
  - Done when: Grep confirms; manual spot-check passes

---

**Total: ~50 tasks × ~25 min = ~20 hours**

All tasks trace to FRs. Dependencies flow: Setup → Data → Engine → State/Router → Components → Styles → Wiring → E2E → Polish.
