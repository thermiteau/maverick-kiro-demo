---
name: infrastructure-as-code
title: Infrastructure as Code — Project Implementation
topic: infrastructure-as-code
last-verified: 2026-10-07
---

## Stack

No IaC tooling is present (no Terraform, CloudFormation, Pulumi, Docker Compose, Kubernetes manifests). The deployment infrastructure is external: Squiz Matrix CMS hosts the page, Form.io hosts the form backend. Deployment is triggered via a Squiz Matrix webhook, as described in `readme.md`. No runbook exists in the repository.

## Configuration

Infrastructure is managed outside this codebase:
- Form.io: form schema, submission handling, and PDF generation are managed via the Form.io admin console using environment-specific projects (identified by the slug in `form.location` URLs in `src/environment.js`).
- Squiz Matrix: page hosting, CMS authentication, and webhook deployment configuration are managed in the Squiz admin interface.
- There is no container or server managed by this repository.

## Patterns

The only deployment artefact produced by this repo is a compiled static bundle (`dist/` from `npm run build`). The CI/CD integration (not yet configured) would need to push this bundle to Squiz Matrix via the webhook mechanism. Environment promotion is implicit in the URL routing (dev/uat/prod Squiz pages load the same bundle but point to different Form.io projects via the `Environment` class URL detection).

## File Locations

- `src/environment.js` — environment-specific Form.io project URLs (the only IaC-adjacent configuration in this repo)
- `readme.md` — describes the Squiz + Form.io deployment workflow and webhook integration
- `package.json` — `build` and `build:squiz` scripts produce the deployable artefact
