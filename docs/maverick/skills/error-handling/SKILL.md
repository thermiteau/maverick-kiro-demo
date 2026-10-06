---
name: error-handling
title: Error Handling — Project Implementation
topic: error-handling
last-verified: 2026-10-07
status: recommended
---

## Stack

No structured error handling library is present. There are no error classes, global error handlers, retry logic, or circuit breakers. The ESLint `no-console` rule prohibits console logging, so errors that are caught silently fail or are swallowed by the Form.io SDK.

## Configuration

The Form.io SDK handles submission errors internally and renders its own error messages within the wizard. `Formio.createForm()` returns a promise; rejection is not handled in the current implementation. The `beforeSubmit` hook mutates submission data and calls `next()` to continue the submission pipeline without error handling.

## Patterns

Current patterns:
- Validation errors: handled declaratively by Form.io validation scripts in `src/validations/`. Scripts set `valid = "error message"` to surface errors to the user in the wizard UI.
- UI state errors: implicit — if a wizard event fires when the component is not ready, the event is ignored.
- localStorage: no try/catch around localStorage access; Safari in private mode throws on `localStorage.setItem`, which would crash the initialisation.

Recommended pattern: wrap `Formio.createForm()` rejection with a user-facing error display. Wrap localStorage access in a try/catch to handle Safari private mode. Add rejection handling to the PDF download and email dispatch promises.

## File Locations

- `src/components/formio-wrapper.js` — unhandled promise locations: `Formio.createForm()`, `_sendEmail()`, `_downloadPDF()`
- `src/config.js` — `beforeSubmit` hook (no error handling)
- `src/validations/**/*.js` — field-level validation error messages
- `src/components/help-guide.js` — localStorage access without try/catch
