---
name: api-design
title: API Design — Project Implementation
topic: api-design
last-verified: 2026-10-07
---

## Stack

This project is a consumer of the Form.io REST API, not an API producer. There are no internal API routes, controllers, or OpenAPI definitions. The Form.io API endpoints are environment-specific URLs managed in `src/environment.js`.

## Configuration

API base URLs are injected at runtime via the `Environment` class. Three endpoints are consumed:
- Form wizard JSON schema (`form.location`) — fetched by `Formio.createForm()`
- PDF generation endpoint (`form.baseLocation + form.pdfEndpoint + '/' + form.endpoint`)
- Admin email endpoint (same PDF base, triggered by form submission at step 10)

No authentication headers are set by this client — Form.io token-based auth is handled by the Form.io SDK internally.

## Patterns

API interactions are encapsulated entirely within `FormioWrapper`. The component exposes no HTTP methods directly; all API calls go through the Form.io SDK (`Formio.createForm()` and wizard events). Error handling for API failures is not implemented (promise rejections are unhandled).

The `beforeSubmit` hook in `src/config.js` mutates the submission data object to inject `formEnv` before the SDK posts it to the Form.io API.

## File Locations

- `src/environment.js` — environment-specific API base URLs
- `src/config.js` — Form.io SDK configuration including `beforeSubmit` hook
- `src/components/formio-wrapper.js` — all API interaction logic via Form.io SDK
