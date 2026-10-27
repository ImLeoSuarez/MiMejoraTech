# Project Constitution — MiMejoraTech

1. **Stack:** Use vanilla TypeScript, Sass (SCSS), native browser APIs, and Vite. No frontend frameworks.
2. **Dependencies:** Keep production dependencies at zero unless explicitly approved.
3. **Type Safety:** Enable strict TypeScript checks. Avoid `any` and unjustified type assertions.
4. **Architecture:** Separate questionnaire flow, assessment logic, UI components, and presentation.
5. **Business Logic:** Keep assessment rules deterministic, documented, and independent of the DOM.
6. **Mobile First:** Design for mobile screens first; verify responsive behavior on desktop.
7. **Accessibility:** Use semantic HTML, keyboard navigation, visible focus states, and reduced-motion support.
8. **Consistency:** Enforce naming conventions, ESLint, Prettier, and English-language source code.
9. **User Language:** All user-facing content, errors, and recommendations must be in Spanish.
10. **Testing as a gate:** Cover business rules, validation, conditional questions, navigation, and boundary cases with automated tests.
11. **End-to-End Quality:** Verify critical user journeys with Playwright before release.
12. **Build Integrity:** Type-checking, linting, unit tests, and production builds must pass before merging.
13. **Privacy:** Do not collect, persist, or transmit user data.
14. **Scope Control:** Do not add backend services, analytics, authentication, or external integrations.
15. **Traceability:** Read `docs/constitution.md` and the active specification `specs/` before coding; document changes and report verification results.