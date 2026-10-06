---
name: environment-management
title: Environment Management — Project Implementation
topic: environment-management
last-verified: 2026-10-07
---

## Stack

No `.env` files or dotenv library. Environment detection uses URL-based runtime logic in `src/environment.js`. Development environment requires only Node.js and npm. No Docker, devcontainer, or containerised development setup. No `.env.example` or CONTRIBUTING.md.

## Configuration

Environment is determined at runtime by the `Environment` class, which inspects `window.location.href` for `localhost`, `dev`, or `uat` substrings. This means no build-time environment injection is needed; the same bundle works across all environments.

Local development runs via Parcel dev server (`npm start`), which serves on localhost and triggers the `development` environment flag. No proxy configuration is needed for Form.io API calls, as requests go directly to the hosted API from the browser.

## Patterns

Three environments: development, uat, production. Each has distinct Form.io API URLs and admin email addresses. The `Environment` class overrides only the keys that differ per environment; the `configuration` object in `src/config.js` provides all other defaults. The merged config is passed to `FormioWrapper` at startup.

`window.formEnv` is set to the detected environment flag (`dev`, `uat`, `prod`) and injected into form submissions.

There is no `.env.example` or setup documentation beyond the `readme.md` Prerequisites and Installation sections.

## File Locations

- `src/environment.js` — environment detection and per-environment value overrides
- `src/config.js` — static configuration defaults
- `src/index.js` — config merge and `window.formEnv` assignment
- `readme.md` — prerequisites (Node.js) and installation steps (`npm install`, `npm start`)
- `package.json` — `start` script for local dev server
