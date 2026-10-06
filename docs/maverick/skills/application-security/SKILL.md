---
name: application-security
title: Application Security — Project Implementation
topic: application-security
last-verified: 2026-10-07
status: recommended
---

## Stack

No application security libraries are present (no helmet, cors, jsonwebtoken, sanitization libraries). Security responsibilities are partly delegated to the Form.io hosted service (form submission, data persistence) and the Squiz Matrix CMS (authentication, CSP headers). The SPA itself is responsible for input validation on form fields via custom Form.io validation scripts.

## Configuration

No SAST, DAST, or dependency vulnerability scanning is configured. `npm audit` is available via npm but not integrated into CI or pre-commit hooks. The ESLint ruleset (airbnb-base) catches some insecure code patterns (eval, with statements, for-in loops).

## Patterns

Input validation is implemented in `src/validations/` as Form.io IIFE scripts. Each script checks for: empty input, maximum length, and a blocklist of special characters (`|&;$">{}+/`). Email fields use regex format validation. Confirmation email uses cross-field equality check. These scripts are evaluated by the Form.io SDK in the browser context and also enforced server-side by Form.io.

The `formEnv` flag is injected into submissions via the `beforeSubmit` hook to tag submissions by environment.

Dependency pinning: `package-lock.json` is committed, providing reproducible installs. Dependency versions use caret ranges (not exact pins) in `package.json`.

## File Locations

- `src/validations/**/*.js` — per-field input validation scripts
- `src/config.js` — `beforeSubmit` hook for environment flag injection
- `package.json` — dependency declarations with caret ranges
- `package-lock.json` — lock file for reproducible installs
