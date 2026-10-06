---
name: integration-testing
title: Integration Testing — Project Implementation
topic: integration-testing
last-verified: 2026-10-07
status: recommended
---

## Stack

No integration testing is currently implemented. The existing Karma suite in `test/` exercises components in isolation with stubbed Form.io API calls. Given the project's Karma and `@open-wc/testing` setup, integration tests would use the same toolchain. An end-to-end layer (e.g. Playwright) would be suitable for testing the full Form.io wizard flow across environments.

## Configuration

Integration tests, if added, would follow the same Karma configuration in `karma.conf.js`. They would require a real Form.io API connection (dev environment) or a local mock server. The `--grep` flag in `karma.conf.js` allows targeting a specific test directory such as `test/integration/**/*.test.js`.

## Patterns

Integration tests for this project should cover:
- The full wizard navigation flow across all 10 steps.
- Interaction between `FormioWrapper`, `ButtonGroup`, and `HelpGuide` through real DOM events.
- Email dispatch at step 10.
- Terms acceptance persistence in localStorage.

Tests that cross the Form.io API boundary should target the `development` environment endpoint and be excluded from standard CI runs with a separate npm script.

## File Locations

- `test/` — existing unit test directory; integration tests should be placed in `test/integration/`
- `karma.conf.js` — would need an additional files entry for the integration test glob
- `src/environment.js` — provides the dev environment API endpoint for integration test connections
