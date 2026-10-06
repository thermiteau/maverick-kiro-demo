---
name: unit-testing
title: Unit Testing — Project Implementation
topic: unit-testing
last-verified: 2026-10-07
---

## Stack

Karma 6 as the test runner. `@open-wc/testing` (built on Chai) for assertions and DOM fixtures via `fixture` and `html` tagged templates. Sinon 9 for stubs and spies. Tests run as ES modules in a real browser context via `@open-wc/testing-karma`. No code coverage tooling configured.

## Configuration

`karma.conf.js` merges the `@open-wc/testing-karma` default config with project overrides. Test files are discovered via the glob pattern `test/**/*.test.js` (overridable with `--grep`). ESM module resolution is enabled with `nodeResolve: true`. `npm test` runs a single-run Karma session; `npm run test:watch` runs with auto-watch enabled.

## Patterns

Tests import component classes directly from `src/components/` and exercise them via constructor instantiation. DOM fixtures are created with `@open-wc/testing`'s `fixture()`. The global `window.Formio` is stubbed using Sinon before each test to prevent real API calls. Assertions use Chai's `expect()` style. Tests are grouped with `describe()` and set up with `beforeEach()`. Test files live in `test/` at the repo root, named `<component>.test.js`.

Tests cover:
- `FormioWrapper` — event handling, config surface, navigation, email dispatch
- `ButtonGroup` — button rendering, click handling, cancel confirmation dialog
- `HelpGuide` — state transitions, template rendering

## File Locations

- `karma.conf.js` — test runner configuration
- `test/formio-wrapper.test.js` — FormioWrapper unit tests
- `test/formio-wrapper-events.test.js` — FormioWrapper event tests
- `test/button-group.test.js` — ButtonGroup unit tests
- `test/help-guide.test.js` — HelpGuide unit tests
- `test/config.js` — shared test configuration (mock config object)
