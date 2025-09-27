# #CTIP-UI Threat Intelligence - User Interface

This application is a user interface that communicates with [ThreatCompass](https://gitlab.blueliv.net/java/ThreadCompass)

## Development

Install dependencies with
```
npm install
```

Start the app with

```
npm start
```


# Stack
- [SvelteKit](https://kit.svelte.dev/) as web framework
- [Tailwind CSS](https://tailwindcss.com/) for the styles
- [Carbon Design System](https://carbon-components-svelte.onrender.com/) for the UI components
- [Felte](https://felte.dev/) + [Yup](https://github.com/jquense/yup) for form validation
- [Day.js](https://day.js.org/) for dates

Parts of the app is in this stack, that is being migrated to the new one
- [Angular 14](https://angular.io/) as web framework, migrating to SvelteKit, to be removed
- [Bootstrap CSS](https://getbootstrap.com/) for styling, to be removed
- [Moment.js](https://momentjs.com/) for dates, to be removed

# Recommended VSCode extensions
- Svelte
- Tailwind
- Headwind
- Paste JSON as Code
- Prettier

# Testing

This project uses two complementary testing layers:

- Unit and component tests with **Vitest** + **@testing-library/svelte** (fast feedback, component logic, utilities)
- End‑to‑end tests with **Cypress** + **Cucumber (Gherkin)** (user journeys, regression, smoke)

Below is a QA‑focused guide to the existing test suites, how to run them locally and in CI, and how to add new tests that match the current patterns.

## Unit and Component Tests (Vitest)

- Framework: Vitest with `jsdom` environment and SvelteKit plugin
- Location and naming: `src/**/__tests__/**` and `src/**/*.{test,spec}.ts`
- Coverage: `v8` provider; run with `npm run coverage`

Key config: see `vitest.config.ts` (aliases: `$lib`, `$stores`, `$src`, `$app`)

Common patterns used:
- Component behavior with Testing Library (e.g., `src/__tests__/components/ExampleWithTest.test.ts`, `src/__tests__/components/ThreatsCommons/ThreatsListCommon/ThreatsListCommon.test.ts`)
- Utility logic (e.g., `src/__tests__/functions/paginationUtils.test.ts`, `src/__tests__/functions/cve.test.ts`)
- Snapshot sanity checks (e.g., `src/__tests__/my-snapshot.test.ts`)

Run locally:

```bash
npm test            # run vitest in watch/interactive mode
npm run coverage    # run once with coverage report
```

Authoring guidelines:
- Place component tests near related code under `src/__tests__/...` and name files `*.test.ts`
- Prefer visible roles/text and `data-test` selectors for robustness
- Avoid internal implementation details; assert rendered output and user behavior

## End‑to‑End Tests (Cypress + Cucumber)

- Config: `cypress/cypress.config.ts` (TypeScript). Spec pattern is `cypress/e2e/**/*.feature`
- Test style: Gherkin feature files with TypeScript step definitions
- Page Object Model and helpers in `cypress/pom/**`
- Global support and custom commands in `cypress/support/**`
- Fixtures and mocked API responses in `cypress/fixtures/**`

Test suites and organization:
- Smoke tests: `cypress/e2e/smoke_tests/**` (e.g., `get_version`, `login`)
- Regression tests: `cypress/e2e/regression_tests/**`
  - Authentication: `authentication/**`
  - Change password: `change_password/**`
  - Modules: `modules/**` (Explorer, settings, credit cards, etc.)
  - Threats: `threat/**`
  - User profile, users management

Preprocessor and BDD:
- Uses `@badeball/cypress-cucumber-preprocessor` with esbuild bundling
- Step definitions are colocated with feature files (non‑global step defs enabled)

Custom commands and test data:
- See `cypress/support/commands.ts` for reusable flows such as `cy.login`, `cy.loginByRequest`, `cy.goToModule`, `cy.assertToast`, and navigation helpers
- API spoofing via `cy.interceptRequest`, driven by `cypress/pom/helpers/spoofing/*` and fixtures under `cypress/fixtures/**`
- By default, tests intercept backend calls with fixtures for reliability and speed

Real API vs. spoofed API:
- Default behavior: requests are intercepted with fixture data
- To hit a real backend, set env var `CYPRESS_realEnd2End=true` (disables spoofing in `interceptRequest`)

Base URL:
- Default in config: `https://tcmaster.blueliv.com`
- Override per run using `--config baseUrl=...` or the provided npm scripts in `cypress/package.json`

Run E2E locally (headless):

```bash
cd cypress
npm i
npx cypress run --config baseUrl=https://localhost:4200 --spec "cypress/e2e/smoke_tests/**/*"
```

Run E2E locally (open GUI):

```bash
cd cypress
npm i
npx cypress open --config baseUrl=https://localhost:4200
```

Convenience scripts (inside `cypress`):

```bash
# Smoke suite (local env)
npm run test:smoke:local

# Regression suite (local env)
npm run test:regression:local

# Against master instance
npm run test:smoke:master
npm run test:regression:master
```

CI smoke script (from repo root):

```bash
HOST=https://tcmaster.blueliv.com ./smoke_test.sh
```

Notes:
- The CI script installs dependencies under `cypress/` and runs smoke specs headlessly with video recording enabled (`video: true`)
- In constrained CI images, required tools (e.g., `curl`) are installed by `smoke_test.sh`

### Page Object Model (POM) Structure

- General pages: `cypress/pom/general/**` (e.g., `LoginPage.ts`, `Navigation.ts`, `UsersPage.ts`, `ThreatsPage.ts`)
- Modules: `cypress/pom/modules/**` (e.g., `pom-creditCard.ts`, `pom-explorer`, `pom-darkWeb.ts`, etc.)
- Helpers: `cypress/pom/helpers/**` (number/text generators, spoofing utilities, endpoint enums)

This separation keeps step definitions lean and emphasizes reusable page actions.

### Fixtures and Test Data

- Common data under `cypress/fixtures/common/**` (e.g., `fakedAuth.json`, `fakedPreferences.json`, `fakedVersion.json`)
- Module‑specific fixtures under `cypress/fixtures/modules/**`
- Threats data under `cypress/fixtures/threat/**`
- Organization and user data under `cypress/fixtures/organization/**` and `cypress/fixtures/users/**`

When adding a test that requires backend data, prefer adding or extending fixtures and configuring an intercept in `spoofedRequests` so the test remains deterministic.

### Adding New Tests

Unit/Component (Vitest):
1. Create `*.test.ts` under `src/__tests__/...` or alongside the feature area
2. Use Testing Library patterns: render the component, query by role/text, simulate events, assert UI changes
3. Use `npm test` locally; keep tests fast and isolated

E2E (Cypress + Cucumber):
1. Add a `.feature` file under the appropriate suite folder (e.g., `cypress/e2e/regression_tests/...`)
2. Create a matching `.ts` step definition file; import from POM and custom commands
3. If needed, add fixtures and an entry to `spoofedRequests` to ensure stable responses
4. Run locally with `npx cypress open` or the provided npm scripts

### Debugging Tips

- Use the Cypress GUI (`cypress open`) to step through tests and view network stubs
- Global cookie logging is enabled (`Cypress.Cookies.debug(true)`) in `cypress/support/e2e.ts`
- `testIsolation` is disabled to allow multi‑step flows; ensure each scenario sets its required preconditions
- Prefer `data-test` attributes in the UI for stable selectors (many commands already target them)

### Quality Gates

- Linting: `npm run lint` (TS/ESLint); security linting available via `npm run lint-security`
- Unit coverage: `npm run coverage`

If you are preparing a QA report or pipeline, combine unit coverage and E2E pass/fail with artifacted Cypress videos for a complete picture.
