# AGENTS.md — MiMejoraTech

## Project

MiMejoraTech is a mobile-first web application that guides users through a step-by-step questionnaire to identify and assess business improvement and optimization opportunities. It evaluates the type, intent, and magnitude of a potential improvement and presents a structured assessment with actionable recommendations.

The application uses **vanilla TypeScript, Sass (SCSS), and native browser APIs**, without frontend frameworks or runtime UI libraries. Vite handles development and production builds. The architecture separates the questionnaire flow, assessment engine, UI components, and presentation layer to keep business rules testable and independent of the DOM.

Core technologies:
- TypeScript 5.x or a compatible newer version.
- JavaScript ES2022 target, using ES modules.
- Sass (SCSS) for styling.
- Vite for development and production builds.
- ESLint and Prettier for code quality and formatting.
- Vitest for unit and integration tests.
- Playwright for end-to-end browser tests.
- npm for package management.

Development and testing dependencies are allowed. Production runtime dependencies must not be added without explicit approval.

## Commands

- Install dependencies: `npm install`
- Run development server: `npm run dev`
- Build for production: `npm run build`
- Preview production build: `npm run preview`
- Type-check: `npm run typecheck`
- Run tests: `npm run test`
- Run tests in watch mode: `npm run test:watch`
- Run end-to-end tests: `npm run test:e2e`
- Run lint: `npm run lint`
- Format code: `npm run format`
- Check formatting without modifying files: `npm run format:check`

The production build must fail if type-checking, linting, or unit tests fail. End-to-end tests must also pass in CI before a release.

These commands assume that all tools and their corresponding configuration files have been set up.

## Style and Conventions

### Language and Compiler

- Use TypeScript with strict type checking enabled.
- Target ES2022 unless browser compatibility requirements dictate otherwise.
- Use ES modules (`import` / `export`).
- Enable `strict`, `noImplicitOverride`, `noUncheckedIndexedAccess`, `useUnknownInCatchVariables`, and `forceConsistentCasingInFileNames`.
- Preserve the Vite-generated TypeScript configuration where needed, including DOM types and bundler module resolution.
- Avoid `any`. Prefer `unknown` when a value's type is not yet known.
- Never suppress compiler errors with `@ts-ignore` without explicit justification and approval.
- Prefer `const`; use `let` only when reassignment is necessary.
- Prefer descriptive, explicit functions and small, focused modules.
- Keep business logic independent of DOM manipulation and UI rendering.

### Naming Conventions

- Variables and functions: `camelCase`.
- Types, interfaces, and classes: `PascalCase`.
- Configuration constants: `UPPER_SNAKE_CASE`.
- TypeScript and Sass filenames: `kebab-case`.
- CSS classes: BEM naming, such as `.question-card__option--selected`.
- Use descriptive names that communicate intent. Avoid unexplained abbreviations.
- Prefer string literal unions over enums when they provide a simpler model.

### Code and Communication Language

- Write source code, identifiers, type names, comments, technical documentation, test names, logs, and commit messages in English.
- Write all user-facing UI text, validation messages, errors, assessment results, and recommendations in Spanish.
- Keep technical logs separate from user-facing error messages. Never expose stack traces or sensitive implementation details to users.
- Use clear, professional, and consistent terminology throughout the application.

### Formatting

- Use two spaces for indentation.
- Use single quotes in TypeScript.
- Use semicolons.
- Use a preferred maximum line width of 100 characters, enforced where practical by Prettier.
- Use Prettier as the source of truth for formatting.
- Use Sass modules with `@use` and `@forward`; do not introduce deprecated Sass `@import`.
- Keep styles organized around design tokens, global styles, utilities, and feature-specific components.
- Avoid unnecessary nesting in Sass.

### Architecture

Organize code by responsibility and feature. A typical structure is:

```text
src/
├── app/
├── components/
├── features/
│   ├── questionnaire/
│   └── assessment/
├── styles/
└── utils/
tests/
├── unit/
├── integration/
└── e2e/
```

- Keep questionnaire definitions, navigation state, validation, and rendering separate.
- Keep assessment rules and calculations in a dedicated, framework-independent module.
- Represent questions, answers, assessment criteria, and results with explicit TypeScript types.
- Prefer structured question definitions that support conditional branching and validation.
- Keep reusable UI components small and focused.
- Avoid unnecessary abstractions and premature generalization.
- Do not introduce a backend, authentication, or persistent storage unless required by the approved specification.

### UI and Mobile-First Design

- Follow a mobile-first responsive design approach.
- Design for narrow screens first, then progressively enhance layouts for larger screens.
- Aim for touch targets of at least 44 × 44 CSS pixels.
- Keep each questionnaire screen focused on one primary question or decision.
- Provide clear progress indicators, selection states, and back/continue navigation.
- Preserve previous answers when navigating backward.
- Use accessible semantic HTML, keyboard navigation, visible focus states, and appropriate ARIA attributes.
- Respect `prefers-reduced-motion`.
- Use native browser APIs, CSS transitions, and SVG where appropriate instead of adding UI libraries.
- Ensure that the interface remains usable across supported mobile and desktop browsers.

## Rules

- Read `docs/constitution.md` and the active specification in `specs/` before modifying code.
- Follow the project constitution and active specification when they conflict with general implementation preferences.
- Do not modify project architecture, core assessment rules, or public data contracts without justification.
- Do not add frontend frameworks, UI component libraries, CSS frameworks, or production runtime dependencies without explicit approval.
- Do not add backend services, authentication, analytics, advertising, or external integrations unless explicitly required.
- Do not introduce persistent storage without an approved requirement, data lifecycle, and privacy considerations.
- Do not hardcode business assessment rules inside UI components.
- Do not store sensitive business information in browser storage without an explicitly approved design.
- Do not introduce new assessment categories, scoring formulas, or business assumptions without approval.
- Do not remove existing functionality or change established behavior without checking the active specification.
- Avoid unrelated refactoring and unnecessary changes outside the task's scope.
- Never commit secrets, credentials, personal data, or environment-specific configuration.
- Keep dependencies minimal and prefer native browser capabilities.
- Do not claim that a feature or test works unless it has been verified.

## After Every Task

- Read the relevant specification and existing implementation before making changes.
- Run `npm run typecheck`.
- Run `npm run lint`.
- Run `npm run format:check`.
- Run `npm run test`.
- Run `npm run test:e2e` when the change affects user flows, browser interactions, responsive behavior, or other end-to-end functionality.
- Run `npm run build` and confirm that the production build succeeds.
- Add or update tests for new functionality, bug fixes, validation rules, conditional questions, and assessment calculations.
- Verify boundary conditions, missing answers, invalid inputs, navigation behavior, and state preservation when relevant.
- Check mobile and desktop layouts for UI changes.
- Verify keyboard navigation, visible focus states, and reduced-motion behavior when relevant.
- Review the final diff to ensure there are no unrelated changes, debug statements, accidental generated files, or secrets.
- If a required command fails, investigate and fix the issue when it falls within scope. Otherwise, report the failure and its cause explicitly.
- Summarize the changes, verification commands executed, results, and any remaining limitations.
- Never report a check as passing if it was not executed successfully.