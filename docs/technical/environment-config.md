---
title: Environment and Configuration
scope: component
relates-to: [architecture-overview.md, formio-wrapper.md]
last-verified: 2026-10-07
---

## Overview

Label Buster uses a two-layer configuration model: a static `configuration` object in `src/config.js` provides defaults, and an `Environment` instance in `src/environment.js` provides environment-specific overrides. Both are merged in `src/index.js` before being passed to `FormioWrapper`.

## Merge Pattern

```
// src/index.js (simplified)
const config = {};
Object.keys(configuration).forEach(key => {
  config[key] = { ...configuration[key], ...environment[key] };
});
```

Each key in `configuration` is a config domain. `environment[key]` is the override object for that domain. Override properties win over defaults for matching keys.

## Environment Detection

`Environment` class reads `window.location.href` at construction time (or accepts an injected URL for testing):

| URL pattern | `selectedEnvironment` | `flag` |
|---|---|---|
| contains `localhost` or `dev` | `development` | `dev` |
| contains `uat` | `uat` | `uat` |
| all other | `production` | `prod` |

`window.formEnv` is set to `flag` and injected into Form.io submission data via `beforeSubmit` hook.

## Environment-Specific Values

| Key | Development | UAT | Production |
|---|---|---|---|
| `form.location` | `…/dev-tzkqydhwrjrviss/labelbuster` | `…/uat-tzkqydhwrjrviss/labelbuster` | `…/tzkqydhwrjrviss/labelbuster` |
| `form.baseLocation` | `…/dev-tzkqydhwrjrviss/` | `…/uat-tzkqydhwrjrviss/` | `…/tzkqydhwrjrviss/` |
| `form.adminEmail` | `FoodPantryDefects@DSITIAQLD.onmicrosoft.com` | `foodpantry@health.qld.gov.au` | `foodsafety@health.qld.gov.au` |

## Static Configuration Domains

### `form`

| Key | Type | Value / Description |
|---|---|---|
| `selector` | `string` | `#formio` — Form.io mount point |
| `title` | `string` | `Label Buster` |
| `endpoint` | `string` | `submission` |
| `pdfEndpoint` | `string` | `labelBusterPdf` |
| `downloadPDF` | `boolean` | `false` |
| `pdfDownloadName` | `function` | `data => \`Label Buster summary - Label Buster Foods ${data.foodName}\`` |
| `clearStorageOnCancel` | `boolean` | `true` |
| `showCancel/Previous/Next/Submit` | `boolean` | all `false` — suppresses default Form.io navigation |
| `adminField` | `string` | `adminEmail` |
| `emailField` | `string` | `toConfirmEmail` |
| `emailConfirmField` | `string` | `email` |

### `scroll`

| Key | Value | Description |
|---|---|---|
| `target` | `0` | Scroll-to Y position (top of page) |
| `type` | `'auto'` | Scroll behaviour |
| `focusTarget` | `'#focusTarget'` | Element to focus after scroll |

### `terms`

| Key | Value | Description |
|---|---|---|
| `title` | `'terms of use'` | Dialog title |
| `termsStorageType` | `localStorage` | Storage for terms acceptance |
| `termsStorageName` | `'lbtermsAndConditions'` | localStorage key |
| `skipIfTermsAlreadyAccepted` | `true` | Skip terms dialog if previously accepted |
| `dataName` | `'lbtermsAndConditions'` | Form.io data field key |

### `buttons`

| Key | Value | Description |
|---|---|---|
| `overwriteFirstButton` | `true` | Replaces generated first button label |
| `overwriteValue` | `'Start'` | Replacement label for first button |
| `showButtonsOnLast` | `true` | Shows buttons on final step |
| `confirmOnCancel` | `true` | Triggers confirmation dialog on cancel |
| `css.base` | `'qg-btn'` | Base button CSS class |
| `css.previous` | `'btn-default'` | Previous button class |
| `css.next` | `'btn-primary'` | Next button class |
| `css.cancel` | `'btn-link'` | Cancel button class |

### `confirmation`

Cancel confirmation dialog text and button config. See `src/config.js` for exact values.

### `navigation`

| Key | Value | Description |
|---|---|---|
| `baseClass` | `'qg-btn btn-link'` | Step nav button class |
| `skipFirstNavStep` | `true` | First wizard step is terms; not shown in step nav |

### `storage`

| Key | Value | Description |
|---|---|---|
| `type` | `localStorage` | Persistence mechanism |
| `name` | `'lbcompleted'` | localStorage key for completion state |

### `helpGuide`

| Key | Value | Description |
|---|---|---|
| `initialState` | `'onboarding'` | Default guide state on first visit |
| `overwriteMobileStateWith` | `false` | No mobile override by default |
| `mobileSize` | `991` | Mobile breakpoint in px |

## Design Decisions

- **URL-based environment detection**: avoids a build-time environment injection step; works in the Squiz Matrix CMS deployment model where the same bundle is promoted through environments by URL.
- **Shallow merge per domain**: each config domain is independently overridable; environment only specifies keys that differ, leaving defaults intact.
