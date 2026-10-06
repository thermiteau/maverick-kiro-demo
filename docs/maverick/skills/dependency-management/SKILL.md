---
name: dependency-management
title: Dependency Management — Project Implementation
topic: dependency-management
last-verified: 2026-10-07
---

## Stack

npm as the package manager. `package-lock.json` committed for reproducible installs. Dependencies versioned with caret ranges (`^`) rather than exact pins. No Dependabot, Renovate, or automated vulnerability scanning configured. No license compliance tooling.

## Configuration

`npm install` is triggered as part of `npm run build` (included in the build script). The `postinstall.js` script runs after every `npm install`. The husky pre-commit hook runs `npm run build`, which re-runs `npm install`, ensuring the lock file is current before commit.

No `.npmrc` configuration file. `devDependencies` contains build and test tooling; `dependencies` contains only `lit-html` as the sole runtime dependency.

## Patterns

Single runtime dependency (`lit-html ^1.3.0`). All other packages are devDependencies. The Form.io SDK is loaded from an external CDN (not an npm dependency) and accessed as the global `window.Formio`.

`npm audit` is available but not integrated into any automated gate. Updates are applied manually. The lock file should be committed on every dependency change to prevent drift.

## File Locations

- `package.json` — dependency declarations, npm scripts
- `package-lock.json` — lock file (committed, npm v6 format)
- `postinstall.js` — post-install script (runs after npm install)
- `.gitignore` — excludes `node_modules/`
