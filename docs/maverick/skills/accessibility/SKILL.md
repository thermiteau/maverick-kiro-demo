---
name: accessibility
title: Accessibility — Project Implementation
topic: accessibility
last-verified: 2026-10-07
---

## Stack

No dedicated accessibility testing library (no axe-core, pa11y, or jest-axe). ARIA attributes are used manually in lit-html component templates. The project targets Queensland Government web standards, which require WCAG 2.1 AA compliance. No eslint-plugin-jsx-a11y configured (the project uses vanilla JS, not JSX).

## Configuration

No automated a11y testing is configured. Accessibility relies on manual audit and the QLD Government SWE (Single Web Experience) CSS framework, which provides accessible base styles for QG components. The Squiz Matrix CMS page shell provides the skip-to-content link and landmark regions.

## Patterns

ARIA usage in component templates:
- `help-guide-lb-main.js` accordion: each accordion article uses a checkbox input with `aria-controls`, `aria-expanded`, and `role="checkbox"`. The `<section>` has `aria-label`.
- `ButtonGroup` cancel confirmation dialog: no ARIA roles on the dialog overlay (missing `role="dialog"`, `aria-modal`, `aria-labelledby`).
- `HelpGuide` sets `returnTab` to the button that opened the guide, intended for focus return after animation, but `returnTab.focus()` is not called.

Keyboard navigation: accordion buttons are keyboard-accessible as they are real `<button>` elements. The `ButtonGroup` renders standard `<button>` elements. The cancel confirmation dialog can be closed via background click but not via the Escape key.

## File Locations

- `src/components/partials/help-guide-lb-main.js` — accordion ARIA attributes
- `src/components/button-group.js` — confirmation dialog (missing dialog ARIA roles)
- `src/components/help-guide.js` — `returnTab` focus management (incomplete implementation)
- `src/index.html` — page structure; Squiz Matrix provides landmark regions via the outer shell
