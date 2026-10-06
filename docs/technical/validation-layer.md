---
title: Validation Layer
scope: component
relates-to: [architecture-overview.md, formio-wrapper.md]
last-verified: 2026-10-07
---

## Overview

Field-level validation is implemented as Form.io custom validation scripts. Each script is a standalone IIFE evaluated by the Form.io SDK in the browser. Scripts set the `valid` variable to either `true` (pass) or a string error message (fail). Scripts are not imported at build time — they are embedded in the Form.io form schema on the remote API.

The files in `src/validations/` are the source of truth for those scripts; they are manually synced to the Form.io form configuration.

## Directory Structure

```
src/validations/
  business-details/     address-line-1, address-line-2, business-name, postcode, state, suburb
  date-marks/           best-date, lot-identification, no-date-lot-identification
  food-name-and-description/  food-description, food-name
  generic/              hiddenObjects, others
  ingredients/          ingredients
  statements/           alcohol, edible-oil, salt
  storage-and-use/      cook-at, cook-for, consume-within, cooking-preparation,
                        directions-other-details, keep-refrigerated-at-maximum,
                        keep-refrigerated-at-minimum, keep-refrigerated-at-or-below,
                        microwave-on, microwoave-for, stand-for-minutes,
                        storage-other-details, thawed-use-within
  your-label/           confirmEmail, email
```

## Validation Pattern

All validation files follow the same IIFE pattern:

```
(function () {
  // guard: empty/required check
  if (input === "") {
    valid = "<message>";
    return;
  }
  // constraint checks (length, format, regex)
  if (input.length > N) {
    valid = "<message>";
    return;
  }
  valid = true;
})();
```

Form.io provides these implicit variables in scope:
- `input` — current field value
- `valid` — write `true` to pass, write an error string to fail

## Validation Rules by Domain

### Business Details
- `business-name`, `address-line-1`, `suburb`: required; max length check; no special characters `|&;$">{}+/`.
- `postcode`: required; must be 4-digit numeric string.
- `state`: required selection.
- `address-line-2`: optional; max length and special-char check if provided.

### Ingredients
- Required (non-empty).
- Max 250 characters.
- No special characters: `|&;$">{}+/`.

### Food Name & Description
- Non-empty.
- Max length constraints.
- No disallowed special characters.

### Date Marks
- `best-date`: date format validation.
- `lot-identification`: alphanumeric pattern.
- `no-date-lot-identification`: conditional — validates only when "no date" path is selected.

### Storage & Use
- Numeric fields (temperatures, durations): range checks (e.g., refrigeration temperature must be within safe bounds).
- Text fields: max length + special-character exclusions.

### Email / Your Label
- `email`: valid email format regex.
- `confirmEmail`: must match `email` field value (cross-field check using Form.io `data` object).

### Statements
- `alcohol`, `edible-oil`, `salt`: pattern-specific checks against regulatory text requirements.

### Generic
- `hiddenObjects`: validates hidden components to prevent data leakage.
- `others`: fallback validation for catch-all fields.

## Design Decisions

- **IIFE wrapping**: Form.io evaluates validation scripts in a shared scope; the IIFE prevents variable leakage between fields.
- **Early return pattern**: exits on first failure with a user-readable message; avoids accumulating errors on a single field.
- **Source files separate from Form.io schema**: allows version control and local testing of validation logic independent of the Form.io hosted form.
