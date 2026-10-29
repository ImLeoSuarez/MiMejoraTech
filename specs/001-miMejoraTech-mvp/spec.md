# Spec 001 — miMejoraTech MVP

## 1. Context and goal

MiMejoraTech is a mobile-first web application that guides a business owner through a short,
step-by-step questionnaire to discover digital improvement and optimization opportunities for
their business. The business owner is not expected to have technical or automation expertise.

The flow asks the user a small set of questions (currently no more than five), evaluates their
answers against a fixed service catalog, and presents a result screen with:

- a list of recommended **services**, and
- the **benefits** (outcomes) each recommendation delivers, and
- a **contact card** with the owner's personal data so the user can reach out.

The application is fully anonymous and static: it does **not** collect, persist, or transmit any
user data, has no backend, no authentication, and no external integrations (per the project
constitution §13 and §14). The contact card only _displays_ the owner's contact information; the
user contacts them through their own channels.

The scope of the flow is **broader digital improvement** across the full service catalog (not
limited to process automation).

### Data-driven questionnaire content

All questionnaire content is **externalized into a single, separately editable data file**. This
file contains:

- the questions, their options, their order, and their selection type;
- the user-facing labels and text (Spanish);
- the service catalog;
- the benefit catalog;
- the `option -> service` mapping;
- the `service -> benefit` mapping;
- the contact card fields and values.

The flow engine reads this file and renders whatever it contains. The "no more than five
questions" rule is a **current content guideline**, not an engine constraint: the engine must
render a variable number of questions, so the owner can add, remove, reorder, or edit questions
and mappings without changing application code.

The data file format is **JSON** (hand-editable), validated against a schema at runtime/build
time. This format choice is flagged as adjustable in the open questions.

### Out of scope (explicit)

- Backend services, authentication, analytics, or external integrations.
- Collecting, storing, or transmitting any user data.
- Auto-sending messages via email or WhatsApp from the application.
- A contact/lead form that captures the user's own information.

## 2. Users

**Primary user — Business owner.** A non-technical person who runs a business and wants to know
whether and how digital tools can improve it. They are not expected to understand programming or
automation concepts, so all wording must be plain, professional Spanish.

**Secondary user — Content owner (the developer/owner).** Maintains the data file to add or edit
questions, options, mappings, services, benefits, and contact information.

## 3. User stories

- **US-1:** As a business owner, I want to answer simple questions one at a time so that I can
  understand the process without technical knowledge.
- **US-2:** As a business owner, I want to see my progress so that I know how far along I am.
- **US-3:** As a business owner, I want to go back and change an answer so that I can correct a
  mistake without starting over.
- **US-4:** As a business owner, I want to skip questions that do not apply to me so that the
  result reflects my actual situation.
- **US-5:** As a business owner, I want a clear list of recommended services and the outcomes
  they deliver so that I understand the value of each recommendation.
- **US-6:** As a business owner, I want a way to contact the service provider so that I can ask
  follow-up questions.
- **US-7:** As a business owner, I want to restart the questionnaire so that I can try again with
  different answers.
- **US-8:** As a business owner, I want to be able to leave the page and eventually come back and
  continue the questionnaire where I left it.
  different answers.
- **US-9:** As the content owner, I want to add, remove, reorder, and edit questions, options,
  services, and mappings in a data file so that I can evolve the questionnaire without changing
  code.

## 4. Functional requirements

Acceptance criteria use EARS notation. The application `SHALL` behave as follows.

### Content and data

**FR-1 — Externalized questionnaire content.**
The system `SHALL` read questions, options, their order, their selection type, labels, the service
catalog, the benefit catalog, the `option -> service` mapping, the `service -> benefit` mapping,
and the contact card data exclusively from the data file.

- `WHEN` the questionnaire is loaded, `THE system SHALL` contain no question text, option text,
  service names, benefit names, or mapping rules hardcoded in flow logic or UI components.

**FR-2 — Variable question count.**
The system `SHALL` render the number of questions defined in the data file without assuming a
fixed count.

- `WHEN` the data file defines `N` questions, `THE system SHALL` present all `N` questions in a
  single run.

**FR-3 — Question order from data.**
The system `SHALL` present questions in the order declared in the data file.

- `WHEN` the data file lists questions in a given order, `THE system SHALL` traverse them in that
  exact order.

**FR-4 — Selection type from data.**
The system `SHALL` determine each question's selection type from data (single-select required vs
multi-select optional).

- `WHEN` a question is declared `single`, `THE system SHALL` require exactly one selected option
  to continue.
- `WHEN` a question is declared `multi`, `THE system SHALL` allow zero or more selected options.

**FR-5 — Editable mapping.**
The `option -> service` and `service -> benefit` mappings `SHALL` be defined in the data file.

- `WHEN` the content owner edits a mapping in the data file, `THE system SHALL` reflect the change
  in the next result computation without a code change.

**FR-6 — Contact card from data.**
The system `SHALL` render the contact card from the data file.

- `WHEN` the result screen is shown, `THE system SHALL` display the contact card fields and values
  defined in the data file (name, email, WhatsApp).

**FR-7 — Malformed data handling.**
The system `SHALL` handle an invalid or missing data file gracefully.

- `WHEN` the data file is missing or fails schema validation, `THE system SHALL` display a
  user-facing Spanish error message instead of crashing, and the build/tests `SHALL` fail.

### Questionnaire flow

**FR-8 — One question per screen.**
The system `SHALL` present exactly one question per screen.

- `WHEN` the user is on a question screen, `THE system SHALL` display only that question and its
  options.

**FR-9 — Progress indicator.**
The system `SHALL` show a progress indicator.

- `WHILE` the user advances through the questionnaire, `THE system SHALL` display progress that
  reflects the current question position relative to the total number of questions.

**FR-10 — Validation for single-select questions.**
The system `SHALL` block navigation forward on unanswered required questions.

- `WHEN` the user attempts to continue without selecting an option on a single-select question,
  `THE system SHALL` show a Spanish validation message and prevent advancing.

**FR-11 — Optional multi-select questions.**
The system `SHALL` allow skipping multi-select questions.

- `WHEN` the user selects no options on a multi-select question, `THE system SHALL` allow
  continuing and treat the question as having no selections.

**FR-12 — Back navigation preserves answers.**
The system `SHALL` preserve previously entered answers when navigating backward.

- `WHEN` the user navigates back and forward again, `THE system SHALL` restore their previous
  selections.

**FR-13 — Restart.**
The system `SHALL` provide a way to restart the questionnaire.

- `WHEN` the user chooses to restart, `THE system SHALL` clear all answers and return to the first
  question.

### Assessment logic

**FR-14 — Deterministic answer-to-service mapping.**
The system `SHALL` map selected options to services deterministically.

- `WHEN` the user completes the questionnaire, `THE system SHALL` compute the recommended
  services as the union of all services mapped from every selected option, using the
  `option -> service` mapping in the data file.

**FR-15 — Benefit derivation.**
The system `SHALL` derive benefits from the recommended services.

- `WHEN` a service is recommended, `THE system SHALL` include all benefits mapped to that service
  via the `service -> benefit` mapping in the data file.

**FR-16 — Deduplication.**
The system `SHALL` deduplicate the result.

- `WHEN` multiple selected options map to the same service, `THE system SHALL` list that service
  once; benefits `SHALL` also appear once per service.

### Result screen

**FR-17 — Result presentation.**
The system `SHALL` present the result as a list of services with their outcomes.

- `WHEN` the result is computed, `THE system SHALL` show each recommended service together with
  its associated benefits.

**FR-18 — Contact card on result.**
The system `SHALL` show the contact card on the result screen.

- `WHEN` the result screen is displayed, `THE system SHALL` show the contact card with name,
  email, and WhatsApp.

**FR-19 — Fallback when no services match.**
The system `SHALL` handle the case where no services are recommended.

- `WHEN` the computed result contains no services, `THE system SHALL` show a friendly Spanish
  fallback message and still display the contact card.

### Language and privacy

**FR-20 — Spanish user-facing content.**
The system `SHALL` display all user-facing text in Spanish.

- `WHILE` the application is in use, `THE system SHALL` show questions, options, validation
  messages, results, and the fallback message in Spanish.

**FR-21 — No user data collection.**
The system `SHALL` not collect, persist, or transmit any user data.

- `WHEN` the user completes the questionnaire, `THE system SHALL` produce the result in memory
  only and `SHALL` not send or store any answer, identifier, or personal information.

## 5. Non-functional requirements

- **NFR-1 — Mobile-first responsive design.** The interface `SHALL` be designed for narrow screens
  first and remain usable on desktop widths.
- **NFR-2 — Accessibility.** The interface `SHALL` use semantic HTML, support keyboard navigation,
  provide visible focus states, maintain touch targets of at least 44 × 44 CSS pixels, and respect
  `prefers-reduced-motion`.
- **NFR-3 — Deterministic, DOM-independent logic.** Assessment and mapping rules `SHALL` be pure,
  deterministic, and independent of UI rendering, per constitution §5.
- **NFR-4 — Zero production dependencies.** The application `SHALL` use vanilla TypeScript, Sass
  (SCSS), and native browser APIs without frontend frameworks or runtime UI libraries.
- **NFR-5 — Static and offline-capable.** The application `SHALL` work as a static bundle without a
  backend or network calls, per constitution §14.
- **NFR-6 — Testability.** Mapping, validation, navigation, conditional behavior, and boundary
  cases `SHALL` be covered by automated tests (constitution §12).

## 6. Edge cases — out of scope

The following are explicitly out of scope for this feature:

- Persisting answers or results between sessions or across reloads.
- Authentication, accounts, analytics, or advertising.
- Any backend service, database, or external API integration.
- Auto-sending email or WhatsApp messages from the application.
- A lead-capture form that collects the user's name, email, or phone.
- Localization to languages other than Spanish.

## 7. Completion criteria

The feature is complete when:

- All functional requirements (FR-1 through FR-21) are implemented and their acceptance criteria
  are met.
- The questionnaire content is fully externalized in the data file; no user-facing content or
  mapping rules are hardcoded in code.
- The current seed content (questions, options, catalogs, mappings, contact card) is present and
  validated.
- Unit and integration tests cover mapping, deduplication, validation, navigation, answer
  preservation, malformed-data handling, and the no-services fallback.
- End-to-end tests cover the full happy path, the back-navigation path, and the fallback path.
- `npm run typecheck`, `npm run lint`, `npm run format:check`, `npm run test`, `npm run
test:e2e`, and `npm run build` all pass.

## 8. Open questions

- [NEED CLARIFICATION] Actual contact card values (owner's name, email, WhatsApp). Current spec
  uses placeholder values defined in the data file.
- [NEED CLARIFICATION] Confirm the data file format. Default chosen: JSON (hand-editable),
  validated by a schema. A TypeScript data module is the alternative if type-safety of the content
  is preferred.
- [NEED CLARIFICATION] Confirm the seed questions, option labels, and `option -> service`
  mappings (documented in the appendix) before implementation.
- [NEED CLARIFICATION] Confirm multi-select questions are optional (skippable) rather than
  required.
- [NEED CLARIFICATION] Confirm the malformed-data policy: fail loudly in build/tests, but show a
  friendly Spanish fallback at runtime.
- [NEED CLARIFICATION] Whether the result screen should offer any "share" or "save" action.
  Default: out of scope, since there is no backend.

## Appendix — Initial seed content (data file)

The following is the initial content of the data file. All text is Spanish and is the user-facing
value, not the technical identifier.

### Service catalog (13)

1. Desarrollo de landing pages de conversión
2. Creación de sitios web corporativos
3. Desarrollo de ecommerce y tiendas online
4. Construcción de dashboards y reportes en tiempo real
5. Automatización de procesos empresariales
6. Desarrollo de paneles administrativos internos
7. Integración entre sistemas y APIs
8. Optimización de bases de datos y manejo de información
9. Desarrollo de software y plataformas a medida
10. Implementación de herramientas con IA y chatbots
11. Optimización de rendimiento y velocidad web
12. SEO técnico y analítica de datos
13. Mantenimiento y soporte técnico continuo

### Benefit catalog (12)

1. Incremento de ventas y generación de clientes potenciales
2. Automatización de tareas repetitivas
3. Reducción de errores operativos
4. Centralización y mejor control de la información
5. Ahorro de tiempo y costos operativos
6. Mejor toma de decisiones basada en datos
7. Escalabilidad tecnológica para el crecimiento del negocio
8. Mejor experiencia para clientes y empleados
9. Mayor visibilidad y posicionamiento digital
10. Integración eficiente entre herramientas y plataformas
11. Mayor productividad y eficiencia interna
12. Modernización y transformación digital de la empresa

### Questions (initial seed — 5 questions)

**P1 — single (required):** "¿Cuál es tu principal objetivo en este momento?"

- Conseguir más clientes y vender más → Landing pages, SEO técnico, Ecommerce
- Vender mis productos por internet → Ecommerce, Landing pages, SEO técnico
- Tener una presencia profesional en internet → Sitios corporativos, SEO técnico
- Organizar y entender mejor mi información → Dashboards, Optimización BD, Paneles admin
- Reducir el trabajo manual y los errores → Automatización, Integraciones, Software a medida
- Modernizar mi empresa y crecer con tecnología → Software a medida, IA/chatbots, Integraciones

**P2 — multi (optional):** "¿Qué tareas o procesos te quitan más tiempo hoy?"

- Tareas manuales y repetitivas → Automatización, Software a medida
- Buscar información dispersa → Dashboards, Optimización BD, Paneles admin
- Herramientas que no se conectan → Integraciones, Paneles admin
- Responder preguntas repetitivas de clientes → IA/chatbots
- Generar reportes a mano → Dashboards, Paneles admin

**P3 — single (required):** "¿Cómo está tu presencia en internet hoy?"

- No tengo sitio web → Sitios corporativos, Landing pages
- Tengo sitio web pero no genera clientes → Landing pages, SEO técnico, Optimización rendimiento
- Tengo sitio web lento o desactualizado → Optimización rendimiento, Sitios corporativos,
  Mantenimiento
- Vendo o quiero vender por internet → Ecommerce
- No es una prioridad ahora → (sin servicios)

**P4 — multi (optional):** "¿Cómo manejas hoy la información y las herramientas de tu negocio?"

- Hojas de cálculo y papel → Automatización, Software a medida, Optimización BD
- Varias herramientas que no se comunican → Integraciones, Paneles admin
- Software propio o antiguo → Software a medida, Mantenimiento
- Apenas empiezo a digitalizarme → Software a medida, Sitios corporativos

**P5 — single (required):** "¿Qué necesitas una vez implementada la solución?"

- Solo el proyecto puntual → (sin servicios adicionales)
- Mantenimiento y soporte continuo → Mantenimiento y soporte
- Que escale con mi crecimiento → Software a medida, Mantenimiento, IA/chatbots

### Service → benefit mapping (initial seed)

| Servicio                                | Beneficios asociados                                                                               |
| --------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Landing pages de conversión             | Incremento de ventas/leads · Mayor visibilidad · Mejor experiencia clientes                        |
| Sitios web corporativos                 | Mayor visibilidad · Mejor experiencia clientes · Modernización                                     |
| Ecommerce y tiendas online              | Incremento de ventas/leads · Mejor experiencia clientes · Escalabilidad · Mayor visibilidad        |
| Dashboards y reportes en tiempo real    | Mejor toma de decisiones · Centralización · Mayor productividad                                    |
| Automatización de procesos              | Automatización tareas repetitivas · Reducción errores · Ahorro tiempo/costos · Mayor productividad |
| Paneles administrativos internos        | Centralización · Mayor productividad · Mejor toma de decisiones                                    |
| Integración entre sistemas y APIs       | Integración eficiente · Automatización tareas · Reducción errores · Centralización                 |
| Optimización de bases de datos          | Centralización · Mejor toma de decisiones · Ahorro tiempo/costos                                   |
| Software y plataformas a medida         | Automatización tareas · Escalabilidad · Modernización · Mayor productividad                        |
| Herramientas con IA y chatbots          | Mejor experiencia clientes · Automatización tareas · Ahorro tiempo/costos · Mayor productividad    |
| Optimización de rendimiento y velocidad | Mejor experiencia clientes · Mayor visibilidad                                                     |
| SEO técnico y analítica de datos        | Mayor visibilidad · Incremento de ventas/leads · Mejor toma de decisiones                          |
| Mantenimiento y soporte técnico         | Reducción errores · Ahorro tiempo/costos · Escalabilidad                                           |
