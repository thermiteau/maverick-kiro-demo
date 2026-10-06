---
name: cicd
title: CI/CD — Project Implementation
topic: cicd
last-verified: 2026-10-07
status: recommended
---

## Stack

No CI/CD pipeline configuration exists in the repository. The project has a Squiz Matrix deployment integration described in `readme.md` (webhook-triggered), but no automated pipeline files are present. Given the project uses GitHub (repo URL in `package.json`), GitHub Actions is the recommended platform.

## Configuration

A GitHub Actions pipeline for this project would need three jobs: lint (ESLint + Prettier check), test (Karma single-run), and build (Parcel production build). The husky pre-commit hook (`npm run build && npm test`) demonstrates the expected local sequence. Secrets needed: Form.io API credentials for integration tests and any Squiz Matrix webhook token for deployment triggers.

## Patterns

Recommended pipeline stages based on `package.json` scripts:
- `npm run lint` — lint gate (ESLint + Prettier check)
- `npm test` — unit test gate (Karma single-run)
- `npm run build` — production build gate
- Deploy artifact to Squiz Matrix via webhook on merge to main

Branch protection should require all three gates to pass before merge. The Squiz Matrix webhook handles environment promotion (dev → uat → prod) externally.

## File Locations

- `package.json` — defines all runnable scripts for the pipeline
- `karma.conf.js` — test runner used in the test stage
- `.eslintrc.json` — ESLint config used in the lint stage
- `readme.md` — describes Squiz Matrix + Form.io deployment workflow
- `.github/workflows/` — recommended location for pipeline definition (does not exist yet)
