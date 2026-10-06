---
name: alerting
title: Alerting — Project Implementation
topic: alerting
last-verified: 2026-10-07
status: recommended
---

## Stack

No alerting mechanism exists in the codebase. The application is a browser-embedded SPA; alerting responsibilities belong to the Form.io hosted service (API uptime, email delivery failures) and the Squiz Matrix CMS. Client-side error rates and availability are not currently monitored.

## Configuration

For a browser SPA of this type, alerting would be implemented via an error monitoring service integrated at the boundary where the SPA interacts with the external Form.io API. A Sentry integration would capture unhandled promise rejections (Form.io load failures, PDF generation failures) and alert the team via configured notification channels. Alert routing should target the environment-specific admin email addresses already present in `src/environment.js`.

## Patterns

No alert calls exist in the current codebase. The existing admin email addresses (`FoodPantryDefects@DSITIAQLD.onmicrosoft.com` for dev, `foodpantry@health.qld.gov.au` for uat, `foodsafety@health.qld.gov.au` for prod) are used for form submission notifications, not operational alerts.

Recommended alerting boundary: `FormioWrapper.initialise()` promise rejection (Form.io API down), email/PDF dispatch failures, and any form submission errors.

## File Locations

- `src/environment.js` — admin email addresses per environment; model for alert routing config
- `src/components/formio-wrapper.js` — alerting boundaries (API load, email, PDF dispatch)
- `src/index.js` — application entry point; recommended location for monitoring SDK initialisation
