---
name: observability
title: Observability — Project Implementation
topic: observability
last-verified: 2026-10-07
status: recommended
---

## Stack

No observability tooling is present. The only instrumentation is a Google Analytics integration via `window.dataLayer` that captures form interaction events. There are no metrics, distributed traces, or health check endpoints.

## Configuration

Google Analytics is configured by pushing to `window.dataLayer` (a standard GTM/GA4 data layer). Events are fired from `FormioWrapper` on `change` events. The integration requires a GTM container or GA4 tag to be loaded by the host CMS page (Squiz Matrix), not within the SPA bundle itself. No configuration is needed inside this project for the dataLayer push to function.

## Patterns

The existing observability pattern is one-dimensional: form field change events are recorded in the GA dataLayer with form title, input ID, type, value, key, label, version, and category. This covers user interaction tracking but not error rates, API call latency, or availability.

The `formiowrapperPageChange` event fires on every wizard step render, which could serve as a hook for page-view tracking if a GTM trigger is configured to consume it.

## File Locations

- `src/index.js` — `formioWrapperTracking` event listener that pushes to `window.dataLayer`
- `src/components/formio-wrapper.js` — `_fireTrackingEvent()` method that dispatches `formioWrapperTracking` with form change payload
