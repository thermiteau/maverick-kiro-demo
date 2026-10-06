---
name: linting
title: Linting — Project Implementation
topic: linting
last-verified: 2026-10-07
---

## Stack

ESLint 7 with eslint-config-airbnb-base and eslint-config-prettier. Prettier 2.2.1 for formatting. eslint-plugin-html for HTML file linting. lint-staged for staged-file checks. husky 5 for pre-commit hooks. No stylelint configured (SCSS files are not linted).

## Configuration

ESLint is configured in `.eslintrc.json`. It extends `airbnb-base` and `eslint-config-prettier` (to disable formatting rules that conflict with Prettier). Custom rules enforce: `no-console`, `no-alert`, `valid-jsdoc`, `eol-last`, `max-len` of 80 (with URL/template exemptions), `arrow-parens as-needed`, and `no-underscore-dangle` is off. `ForInStatement` and `WithStatement` are restricted via `no-restricted-syntax`.

Prettier is configured inline in `package.json`: single quotes, trailing commas, 80-character print width, arrow parens `avoid`.

lint-staged runs `eslint --fix` on staged `.js` files and `prettier --write` on staged `.html` files. Husky pre-commit hook also runs `npm run build` and `npm test`.

## Patterns

Two categories of linting scripts: `lint:*` (check-only) and `format:*` (auto-fix). `npm run lint` runs both eslint and prettier checks. `npm run format` auto-fixes both. Partials in `src/components/partials/` are formatted via a custom `formatFile.js` script called as `npm run format:partials`.

`/* eslint-disable */` directives appear in validation scripts (IIFEs in `src/validations/`) where Form.io implicit variables would otherwise cause lint errors. `// eslint-disable-next-line` is used sparingly for unavoidable `max-len` violations in URLs.

## File Locations

- `.eslintrc.json` — ESLint configuration
- `package.json` — Prettier config, husky hooks, lint-staged config, npm lint/format scripts
- `src/validations/**/*.js` — contain targeted eslint-disable comments
- `formatFile.js` — custom script for formatting partials
- `postinstall.js` — runs after `npm install`
