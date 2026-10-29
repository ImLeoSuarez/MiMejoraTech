# Plan — MiMejoraTech MVP (Spec 001)

## 1. Module Structure

```
src/
├── app/
│   ├── main.ts                    # Application entry point (FR-1, FR-7, FR-8, FR-13)
│   ├── router.ts                  # Navigation state & history (FR-8, FR-12, FR-13)
│   └── state.ts                   # Global questionnaire state (FR-12, FR-13)
├── components/
│   ├── question-screen.ts         # Renders single question + options (FR-8, FR-10, FR-11)
│   ├── progress-bar.ts            # Progress indicator (FR-9)
│   ├── result-screen.ts           # Result list + contact card (FR-17, FR-18, FR-19)
│   ├── contact-card.ts            # Contact card component (FR-6, FR-18)
│   └── error-message.ts           # User-friendly Spanish errors (FR-7, FR-20)
├── features/
│   ├── questionnaire/
│   │   ├── loader.ts              # Loads & validates data file (FR-1, FR-2, FR-3, FR-7)
│   │   ├── schema.ts              # JSON schema for data validation (FR-7)
│   │   ├── types.ts               # Question, Option, SelectionType types (FR-4)
│   │   └── validation.ts          # Answer validation logic (FR-10, FR-11)
│   └── assessment/
│       ├── engine.ts              # Core mapping & deduplication (FR-14, FR-15, FR-16)
│       ├── types.ts               # Service, Benefit, Mapping types
│       └── deduplication.ts       # Set-based deduplication helpers (FR-16)
├── styles/
│   ├── _tokens.scss               # Design tokens (colors, spacing, breakpoints)
│   ├── _global.scss               # Global resets & base styles
│   ├── _utilities.scss            # Mobile-first utilities (NFR-1, NFR-2)
│   └── components/
│       ├── _question-screen.scss
│       ├── _progress-bar.scss
│       ├── _result-screen.scss
│       └── _contact-card.scss
└── utils/
    ├── dom.ts                     # Safe DOM helpers (NFR-4)
    └── i18n.ts                    # Spanish string constants (FR-20)

data/
└── questionnaire.json             # Externalized content (FR-1, FR-5, FR-6, FR-9)

tests/
├── unit/
│   ├── assessment/
│   │   ├── engine.test.ts         # Mapping, deduplication (FR-14, FR-15, FR-16)
│   │   └── deduplication.test.ts
│   ├── questionnaire/
│   │   ├── loader.test.ts         # Data loading & validation (FR-1, FR-7)
│   │   ├── validation.test.ts     # Answer validation (FR-10, FR-11)
│   │   └── schema.test.ts
│   └── router.test.ts             # Navigation state (FR-8, FR-12, FR-13)
├── integration/
│   ├── flow.test.ts               # Full questionnaire flow (FR-8 through FR-13)
│   └── assessment-integration.test.ts
└── e2e/
    ├── happy-path.spec.ts         # Full user journey (FR-1 through FR-21)
    ├── back-navigation.spec.ts    # Back/forward preservation (FR-12)
    └── fallback.spec.ts           # No-services fallback (FR-19)

vite.config.ts
tsconfig.json
package.json
```

**FR coverage summary:**

- FR-1,2,3,4,5,6,7 → `features/questionnaire/`
- FR-8,9,10,11,12,13 → `app/`, `components/`, `features/questionnaire/validation.ts`
- FR-14,15,16 → `features/assessment/`
- FR-17,18,19 → `components/result-screen.ts`, `components/contact-card.ts`
- FR-20 → `utils/i18n.ts`, all components
- FR-21 → Architecture (no persistence layer)
- NFR-1,2 → `styles/`, components
- NFR-3 → `features/assessment/` (DOM-independent)
- NFR-4,5 → Stack choices, no runtime deps
- NFR-6 → `tests/unit/`, `tests/integration/`

---

## 2. JSON Data Model

### Schema (TypeScript interface for validation)

```typescript
interface QuestionnaireData {
  meta: {
    version: string;
    title: string;
  };
  questions: Question[];
  services: Service[];
  benefits: Benefit[];
  mappings: {
    optionToService: Record<string, string[]>; // optionId -> serviceIds
    serviceToBenefit: Record<string, string[]>; // serviceId -> benefitIds
  };
  contactCard: ContactCard;
}

interface Question {
  id: string;
  text: string;
  type: 'single' | 'multi';
  required: boolean;
  options: Option[];
}

interface Option {
  id: string;
  text: string;
}

interface Service {
  id: string;
  name: string;
}

interface Benefit {
  id: string;
  name: string;
}

interface ContactCard {
  name: string;
  email: string;
  whatsapp: string;
}
```

### Example (matches Appendix seed content)

```json
{
  "meta": {
    "version": "1.0.0",
    "title": "MiMejoraTech — Diagnóstico Digital"
  },
  "questions": [
    {
      "id": "p1",
      "text": "¿Cuál es tu principal objetivo en este momento?",
      "type": "single",
      "required": true,
      "options": [
        { "id": "p1-o1", "text": "Conseguir más clientes y vender más" },
        { "id": "p1-o2", "text": "Vender mis productos por internet" },
        { "id": "p1-o3", "text": "Tener una presencia profesional en internet" },
        { "id": "p1-o4", "text": "Organizar y entender mejor mi información" },
        { "id": "p1-o5", "text": "Reducir el trabajo manual y los errores" },
        { "id": "p1-o6", "text": "Modernizar mi empresa y crecer con tecnología" }
      ]
    },
    {
      "id": "p2",
      "text": "¿Qué tareas o procesos te quitan más tiempo hoy?",
      "type": "multi",
      "required": false,
      "options": [
        { "id": "p2-o1", "text": "Tareas manuales y repetitivas" },
        { "id": "p2-o2", "text": "Buscar información dispersa" },
        { "id": "p2-o3", "text": "Herramientas que no se conectan" },
        { "id": "p2-o4", "text": "Responder preguntas repetitivas de clientes" },
        { "id": "p2-o5", "text": "Generar reportes a mano" }
      ]
    }
  ],
  "services": [
    { "id": "landing-pages", "name": "Desarrollo de landing pages de conversión" },
    { "id": "sitios-corporativos", "name": "Creación de sitios web corporativos" },
    { "id": "ecommerce", "name": "Desarrollo de ecommerce y tiendas online" }
  ],
  "benefits": [
    {
      "id": "incremento-ventas",
      "name": "Incremento de ventas y generación de clientes potenciales"
    },
    { "id": "automatizacion-tareas", "name": "Automatización de tareas repetitivas" },
    { "id": "reduccion-errores", "name": "Reducción de errores operativos" }
  ],
  "mappings": {
    "optionToService": {
      "p1-o1": ["landing-pages", "seo-tecnico", "ecommerce"],
      "p1-o2": ["ecommerce", "landing-pages", "seo-tecnico"],
      "p2-o1": ["automatizacion", "software-a-medida"]
    },
    "serviceToBenefit": {
      "landing-pages": ["incremento-ventas", "mayor-visibilidad", "mejor-experiencia-clientes"],
      "automatizacion": [
        "automatizacion-tareas",
        "reduccion-errores",
        "ahorro-tiempo-costos",
        "mayor-productividad"
      ]
    }
  },
  "contactCard": {
    "name": "MiMejoraTech",
    "email": "hola@mimejoratech.com",
    "whatsapp": "+34 600 000 000"
  }
}
```

---

## 3. Assessment Algorithm (Pseudocode)

> Note: The spec refers to "mapping & deduplication" not "streak calculation". This algorithm computes recommended services & benefits from selected options.

```
FUNCTION computeAssessment(selectedOptions: Set<OptionId>, data: QuestionnaireData) -> AssessmentResult:

  // FR-14: Union of services from all selected options
  recommendedServiceIds ← EMPTY SET
  FOR each optionId IN selectedOptions:
    IF optionId EXISTS IN data.mappings.optionToService:
      FOR each serviceId IN data.mappings.optionToService[optionId]:
        recommendedServiceIds.ADD(serviceId)

  // FR-16: Deduplication — SET ensures uniqueness
  // FR-15: Derive benefits from recommended services
  recommendedBenefits ← EMPTY MAP<ServiceId, Set<BenefitId>>
  FOR each serviceId IN recommendedServiceIds:
    benefitIds ← data.mappings.serviceToBenefit[serviceId] OR EMPTY ARRAY
    recommendedBenefits[serviceId] ← NEW SET(benefitIds)

  // FR-19: Handle empty result
  IF recommendedServiceIds IS EMPTY:
    RETURN {
      services: [],
      benefits: [],
      fallbackMessage: "No hemos encontrado servicios que coincidan con tus respuestas. ¡Contáctanos para una valoración personalizada!",
      contactCard: data.contactCard
    }

  // Build output with full objects (for display)
  services ← data.services.FILTER(s → recommendedServiceIds.HAS(s.id))
  benefitsMap ← {}
  FOR each service IN services:
    benefitIds ← recommendedBenefits[service.id]
    benefitsMap[service.id] ← data.benefits.FILTER(b → benefitIds.HAS(b.id))

  RETURN {
    services: services,
    benefits: benefitsMap,
    fallbackMessage: NULL,
    contactCard: data.contactCard
  }

TYPE AssessmentResult = {
  services: Service[],
  benefits: Map<ServiceId, Benefit[]>,
  fallbackMessage: string | NULL,
  contactCard: ContactCard
}
```

**Determinism guarantees (NFR-3):**

- Pure function: no side effects, no DOM access, no randomness
- Same inputs → same outputs
- Set-based operations guarantee stable ordering (sort by original data order for display)

---

## 4. CLI Contract

> The application is a static web app (Vite). No custom CLI is required beyond npm scripts. This section documents the standard npm commands as the effective CLI.

| Command                | Description                                          | Exit Code                                | Output                           |
| ---------------------- | ---------------------------------------------------- | ---------------------------------------- | -------------------------------- |
| `npm install`          | Install dependencies                                 | 0 on success, 1 on failure               | Stdout: progress; Stderr: errors |
| `npm run dev`          | Start Vite dev server                                | 0 (runs until killed), 1 on config error | Server URL, HMR logs             |
| `npm run build`        | Production build (typecheck + lint + test + compile) | 0 on success, 1 on any failure           | Dist/ folder, build summary      |
| `npm run preview`      | Preview production build                             | 0 (serves until killed)                  | Server URL                       |
| `npm run typecheck`    | Run `tsc --noEmit`                                   | 0 on success, 1 on type errors           | Type errors to stderr            |
| `npm run lint`         | Run ESLint                                           | 0 on success, 1 on lint errors           | Lint violations to stdout        |
| `npm run format`       | Run Prettier --write                                 | 0 on success, 1 on error                 | Formatted files list             |
| `npm run format:check` | Run Prettier --check                                 | 0 if formatted, 1 if changes needed      | Files needing formatting         |
| `npm run test`         | Run Vitest (unit + integration)                      | 0 if all pass, 1 if any fail             | Test results, coverage           |
| `npm run test:watch`   | Vitest watch mode                                    | 0 (runs until killed)                    | Live test output                 |
| `npm run test:e2e`     | Run Playwright E2E tests                             | 0 if all pass, 1 if any fail             | Test report, traces              |

**Build pipeline enforcement (NFR-12, Constitution §12):**

```
npm run build
  ├─> npm run typecheck   (must pass)
  ├─> npm run lint        (must pass)
  ├─> npm run format:check (must pass)
  ├─> npm run test        (must pass)
  └─> vite build          (produces dist/)
```

---

## 5. Justified Technical Decisions

| Decision                                          | Alternatives Considered                              | Rationale                                                                                                                                                             |
| ------------------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Vanilla TypeScript + Vite**                     | React, Vue, Svelte, Astro                            | Constitution §4, §11, NFR-4: Zero runtime deps, minimal bundle, full control. Vite provides fast DX without framework overhead.                                       |
| **JSON data file + runtime schema validation**    | TypeScript module (`const data = {...}`), YAML, TOML | JSON is hand-editable by non-devs (FR-9), schema validates at build+runtime (FR-7). TS module would require build step for content changes; YAML/TOML add parser dep. |
| **Single data file (questionnaire.json)**         | Split files (questions.json, services.json, ...)     | Simpler for content owner (FR-9), single source of truth, atomic validation.                                                                                          |
| **Set-based deduplication in assessment**         | Array.filter + indexOf, Map with counters            | `Set` is idiomatic, O(1) lookup, guarantees uniqueness (FR-16), clear intent.                                                                                         |
| **DOM-independent assessment engine**             | Embedded in UI component, React hooks                | Constitution §5, NFR-3: Pure functions testable without DOM, portable to SSR/Worker later.                                                                            |
| **SessionStorage for answer persistence (US-8)**  | localStorage, IndexedDB, nothing                     | US-8 requires "continue where left off" across reloads. sessionStorage survives reload, clears on tab close (privacy-friendly, FR-21). No backend needed.             |
| **Spanish-only user content**                     | i18n framework (i18next, etc.)                       | FR-20, Constitution §9: Single language, no i18n complexity. Spanish strings in data file + constants.                                                                |
| **BEM CSS with Sass modules**                     | CSS-in-JS, Tailwind, plain CSS                       | Constitution §4: No runtime CSS-in-JS. Sass modules provide scoping, tokens, no extra runtime. BEM = predictable class names.                                         |
| **Vitest + Playwright**                           | Jest + Cypress, uvu + Playwright                     | Vitest = Vite-native, fast, TypeScript-first. Playwright = best cross-browser E2E, constitution §11.                                                                  |
| **No answer persistence across sessions (FR-21)** | localStorage with expiry                             | Constitution §13: "Do not collect, persist, or transmit user data." Session-only is acceptable for US-8 (reload recovery) but not cross-session.                      |

---

## 6. Testing Strategy

### Unit Tests (`npm run test` — Vitest)

| Target                                 | Coverage                                                       | FRs                        |
| -------------------------------------- | -------------------------------------------------------------- | -------------------------- |
| `features/assessment/engine.ts`        | Mapping union, benefit derivation, deduplication, empty result | FR-14, FR-15, FR-16, FR-19 |
| `features/assessment/deduplication.ts` | Set operations, stable ordering                                | FR-16                      |
| `features/questionnaire/loader.ts`     | Valid load, missing file, schema violation, malformed JSON     | FR-1, FR-7                 |
| `features/questionnaire/validation.ts` | Single-select required, multi-select optional, empty selection | FR-10, FR-11               |
| `features/questionnaire/schema.ts`     | Valid/invalid data against JSON schema                         | FR-7                       |
| `app/router.ts`                        | Next/prev/restart, answer preservation, index bounds           | FR-8, FR-12, FR-13         |
| `app/state.ts`                         | Answer storage, retrieval, clear, sessionStorage sync          | FR-12, US-8                |

### Integration Tests (`npm run test` — Vitest)

| Target                                             | Scenario                                          | FRs               |
| -------------------------------------------------- | ------------------------------------------------- | ----------------- |
| `tests/integration/flow.test.ts`                   | Full questionnaire: answer all → result → restart | FR-1–13, FR-17–19 |
|                                                    | Back navigation restores answers                  | FR-12             |
|                                                    | Skip multi-select → continue allowed              | FR-11             |
|                                                    | Block on empty single-select                      | FR-10             |
|                                                    | Progress updates correctly                        | FR-9              |
| `tests/integration/assessment-integration.test.ts` | Loader + engine with real data file               | FR-1, FR-14–16    |
|                                                    | Malformed data → friendly error screen            | FR-7, FR-20       |

### End-to-End Tests (`npm run test:e2e` — Playwright)

| Spec                      | User Journey                                                                     | FRs          |
| ------------------------- | -------------------------------------------------------------------------------- | ------------ |
| `happy-path.spec.ts`      | Load → answer 5 questions → see results with services/benefits/contact → restart | FR-1–21      |
| `back-navigation.spec.ts` | Answer Q3 → back to Q2 → change answer → forward to Q3 (preserved) → result      | FR-8, FR-12  |
| `fallback.spec.ts`        | Select options mapping to no services → see fallback message + contact card      | FR-19, FR-18 |

### Additional Quality Gates

| Gate               | Tool                               | Constitution |
| ------------------ | ---------------------------------- | ------------ |
| Type safety        | `npm run typecheck` (tsc --noEmit) | §3, §12      |
| Linting            | `npm run lint` (ESLint)            | §8, §12      |
| Formatting         | `npm run format:check` (Prettier)  | §8, §12      |
| Unit + integration | `npm run test` (Vitest)            | §10, §12     |
| E2E                | `npm run test:e2e` (Playwright)    | §11, §12     |
| Production build   | `npm run build` (Vite)             | §12          |

### Test Data Strategy

- **Unit tests**: Inline minimal fixtures (3 questions, 3 services, 3 benefits)
- **Integration tests**: Load real `data/questionnaire.json` (validates schema + content)
- **E2E tests**: Use real data file; test Chromium + Firefox + WebKit (mobile + desktop viewports)
- **Accessibility**: Playwright + axe-core in E2E (NFR-2)

### Boundary Cases to Verify

| Case                                                                | Test Level                      |
| ------------------------------------------------------------------- | ------------------------------- |
| Empty data file / missing file                                      | Unit (loader)                   |
| Schema validation failure (extra field, missing required)           | Unit (loader/schema)            |
| All single-select unanswered → cannot proceed                       | Integration                     |
| All multi-select skipped → valid result                             | Integration                     |
| Duplicate service via different options → appears once              | Unit (engine)                   |
| Duplicate benefit via different services → appears once per service | Unit (engine)                   |
| No services matched → fallback + contact card                       | Unit (engine), Integration, E2E |
| Back navigation after skip → multi-select still empty               | Integration                     |
| Restart clears sessionStorage                                       | Unit (state), E2E               |
| Mobile viewport (375px) — touch targets ≥44px                       | E2E (visual + axe)              |
| `prefers-reduced-motion` — no transitions                           | E2E                             |

---

## 7. Open Questions for Clarification

Before implementation, the following need confirmation (from Spec §8):

1. **Contact card values** — Confirm final name, email, WhatsApp for production data file.
2. **Data file format** — JSON confirmed, but validate: is a separate `.json` + schema preferred over a `.ts` module with Zod/Arktype validation?
3. **Seed content** — Confirm the 5 questions, option texts, and mappings in Appendix before coding.
4. **Multi-select = optional** — Spec assumes skippable (FR-11). Confirm no required multi-select.
5. **Malformed data policy** — Build/tests fail loudly; runtime shows friendly Spanish error (current plan). Confirm.
6. **Share/save on result** — Default: out of scope (no backend). Confirm no "copy link" or "download PDF" needed.

---

## 8. Implementation Order (Suggested)

1. **Project setup** — `package.json`, `tsconfig.json`, `vite.config.ts`, ESLint/Prettier/Vitest/Playwright configs
2. **Data model & schema** — `features/questionnaire/types.ts`, `schema.ts`, `loader.ts` + tests
3. **Assessment engine** — `features/assessment/` (pure TS) + unit tests
4. **State & router** — `app/state.ts`, `app/router.ts` + unit tests
5. **UI components** — `components/` (question, progress, result, contact, error) + integration tests
6. **Styles** — Sass tokens, global, components (mobile-first)
7. **App entry & wiring** — `app/main.ts` loads data, initializes router, mounts first screen
8. **E2E tests** — Happy path, back nav, fallback
9. **Polish** — Accessibility audit, reduced-motion, cross-browser, build verification

---

**All FRs addressed:** FR-1 through FR-21 mapped to modules, algorithms, tests, and CLI gates above.
