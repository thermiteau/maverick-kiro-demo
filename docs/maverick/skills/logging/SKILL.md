---
name: logging
title: Logging — Project Implementation
topic: logging
last-verified: 2026-10-07
status: recommended
---

## Stack

No structured logging library is present. `console.error` is prohibited by the ESLint `no-console` rule, meaning there is no logging instrumentation in the current codebase. The application runs entirely in the browser as an embedded SPA; server-side logging is handled by the Form.io hosted service and is outside the project's scope.

## Configuration

For client-side error capture, integrating a browser error monitoring service (e.g. Sentry) would be appropriate. Sentry can be initialised once in `src/index.js` and configured via an environment-specific DSN injected by the `Environment` class pattern already in place. Log level should be `error-only` in production, with additional breadcrumbs captured on Form.io page change events.

## Patterns

Current codebase has no logging calls. Unhandled errors surface silently in the browser. Errors in `FormioWrapper.initialise()` (e.g. Form.io API failures) are not caught or reported. Errors in validation scripts are swallowed by the Form.io SDK.

Recommended pattern: wrap the `Formio.createForm` promise rejection, the email dispatch, and the PDF download with error capture calls that route to a monitoring service.

## File Locations

- `src/index.js` — application bootstrap; recommended location for monitoring initialisation
- `src/environment.js` — pattern for environment-specific config; recommended for monitoring DSN configuration
- `src/components/formio-wrapper.js` — primary location for error capture integration points
