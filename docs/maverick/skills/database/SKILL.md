---
name: database
title: Database — Project Implementation
topic: database
last-verified: 2026-10-07
---

## Stack

No database is managed by this project. Data persistence is entirely delegated to the Form.io hosted service, which stores form submissions. The project uses two browser-side persistence mechanisms: `localStorage` for UI state (terms acceptance, help guide seen state) and `localStorage` for partially-completed form state.

## Configuration

localStorage usage is configured via the `terms` and `storage` blocks in `src/config.js`. The storage type is `localStorage` (configurable to allow testing with mocked storage). The `storage.name` key is `lbcompleted` and `terms.termsStorageName` is `lbtermsAndConditions`.

HelpGuide persists its seen-state under the key `help-guide` directly via `localStorage.setItem`.

## Patterns

- Terms acceptance: stored as the string value of `terms.dataName` in `localStorage`. Checked on app init; if present, the terms dialog is skipped.
- Form completion state: stored under `storage.name` (`lbcompleted`). Cleared on cancel if `form.clearStorageOnCancel` is true.
- Help guide state: stored under `help-guide`. Presence of the key (any truthy value) means the user has seen the onboarding view.

All state is per-browser, per-domain. No server-side session management.

## File Locations

- `src/config.js` — `terms` and `storage` configuration blocks
- `src/components/help-guide.js` — direct `localStorage` reads and writes for help guide state
- `src/components/formio-wrapper.js` — storage clear-on-cancel logic
