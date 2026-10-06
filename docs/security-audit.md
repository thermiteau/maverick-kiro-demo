# Security Audit

**Generated:** 2026-10-07T08:50:00+11:00
**Stack detected:** Node.js browser SPA — lit-html, Parcel bundler, Form.io SDK (CDN), Karma/Sinon tests

## Summary

A hardcoded token value (`14ba081-11d8-4517-9171-f85f9057f199`) used as an email-dispatch gate is committed in the tracked Form.io schema file `json/stage-json/foodPantry.json`. This is the highest-severity finding. Additionally, the Form.io SDK is loaded from a CDN without a pinned version or Subresource Integrity (SRI) hash, exposing the application to supply-chain compromise.

| Category | Status |
| --- | --- |
| Secret exposure | FAIL |
| Dependency hygiene | WARN |
| Authentication / authorisation | N/A |
| Input validation / output encoding | WARN |
| Transport, headers, CORS | N/A |
| Data at rest | PASS |
| Logging, monitoring, rate limiting | N/A |
| Container / infrastructure | N/A |

---

## Details

### Secret exposure — FAIL

**Evidence:**

`json/stage-json/foodPantry.json:5448` contains:
```
"custom": "execute = ((data.secureToken === '14ba081-11d8-4517-9171-f85f9057f199') && (data.sendEmail === 'admin'))"
```

The same token value appears at lines 5448, 5486, and 5538. The file is tracked in git (`git ls-files` confirms `json/stage-json/foodPantry.json`). This token gates admin email dispatch and user confirmation email dispatch on Form.io. Any person with read access to this repository can read the token and craft a request that bypasses the intended email-gate condition.

**Recommendation:** Rotate the token immediately (change it in the Form.io admin console). Do not store bearer tokens or conditional gate values in tracked source files. Use Form.io environment variables or a server-side action condition that does not embed the secret in the schema JSON. Remove the token from git history with `git filter-repo` or BFG Repo-Cleaner after rotating it.

---

### Dependency hygiene — WARN

**Evidence 1 — Unpinned CDN dependency:**

`src/index.html:2` loads the Form.io SDK from a CDN without a pinned version or SRI hash:

```
<script src="https://unpkg.com/formiojs@latest/dist/formio.full.min.js"></script>
```

`@latest` means any future breaking or malicious release on unpkg will be loaded automatically. No `integrity` attribute is present, so there is no browser-enforced cryptographic verification of the script content.

**Recommendation:** Pin to a specific semver (e.g. `formiojs@4.x.x`) and add a Subresource Integrity `integrity` attribute with the correct hash. Evaluate whether the Form.io SDK can be bundled as an npm dependency instead (it is not currently in `package.json`).

**Evidence 2 — No automated vulnerability scanning:**

No `npm audit` step in CI (no CI configured). No Dependabot or Renovate configuration present. Dependencies use caret ranges in `package.json`, so patch and minor updates are applied on fresh installs. `package-lock.json` is committed but the `#yarn.lock / #package-lock.json` comments in `.gitignore` suggest lock file tracking was previously debated.

**Recommendation:** Add `npm audit --audit-level=high` to any CI pipeline. Configure Dependabot (`.github/dependabot.yml`) for automated security updates.

---

### Input validation / output encoding — WARN

**Evidence 1 — `unsafeHTML` used in ButtonGroup:**

`src/components/button-group.js:107` renders the close button icon via `unsafeHTML()`:

```
${unsafeHTML(configuration.confirmation.closeXButton)}
```

The value `configuration.confirmation.closeXButton` is `'<i class="fa fa-times"></i>'` — a static string from `src/config.js`. The value is not user-controlled, so this is a low-severity XSS vector in the current implementation. However, if this config value ever came from an external or user-controlled source, it would be a direct XSS injection point.

**Recommendation:** Replace `unsafeHTML` with a non-HTML icon alternative (e.g., a CSS-only close button or an SVG). If `unsafeHTML` is retained, add a code comment documenting that the value must remain a static literal.

**Evidence 2 — `document.body.innerHTML` overwrite in print.js:**

`src/scripts/print.js:9`:
```
document.body.innerHTML = contentSource.innerHTML;
```

This replaces the entire document body with the content of `#help-guide`. The source is always the app's own rendered DOM, not user input, so XSS risk is low in practice. However, any content rendered via `unsafeHTML` (see above) or injected from Form.io responses would be included.

**Recommendation:** Use `window.print()` with a print-specific CSS media query instead of replacing `document.body.innerHTML`. This avoids the full-page clobber and is more maintainable.

**Evidence 3 — `innerHTML` assignment in formio-wrapper.js and index.js:**

`src/components/formio-wrapper.js:769`: `this.emailElement.innerHTML = ''` — clears content, no risk.
`src/index.js:62`: `sectionNav.innerHTML = '<ul><li>...</li></ul>'` — static literal, no risk.

---

### Authentication / authorisation — N/A

This is a browser-embedded SPA with no authentication layer. Form.io handles authentication for the API backend. The Squiz Matrix CMS manages page-level access. No auth code is present in this repository to audit.

---

### Transport, headers, CORS — N/A

There is no server component in this repository. CSP, HSTS, CORS, and security headers are the responsibility of the Squiz Matrix CMS hosting the page and the Form.io hosted service. This audit cannot verify those configurations — they require a live environment review.

---

### Data at rest — PASS

The application uses `localStorage` for UI state only (terms acceptance flag, help-guide seen flag, form completion flag). No PII or sensitive form data is stored in `localStorage`. Form submission data is persisted server-side by Form.io.

---

### Logging, monitoring, rate limiting — N/A

No server component exists in this repository. The `no-console` ESLint rule prevents logging instrumentation in the bundle. Logging, monitoring, and rate limiting are handled by the Form.io hosted service.

---

### Container / infrastructure — N/A

No Dockerfile, Docker Compose, or IaC is present in this repository. Infrastructure is managed externally (Squiz Matrix, Form.io).

---

## Recommendations (prioritised)

1. **High** — Rotate the `secureToken` value (`14ba081-11d8-4517-9171-f85f9057f199`) in the Form.io admin console immediately, then rewrite git history to remove it from `json/stage-json/foodPantry.json`. `json/stage-json/foodPantry.json:5448,5486,5538`. Committed tokens remain exploitable after the file is updated unless history is rewritten.

2. **High** — Pin the Form.io CDN dependency to a specific version and add an SRI hash. `src/index.html:2`. An unpinned `@latest` CDN script can be silently updated to a malicious version.

3. **Medium** — Replace `unsafeHTML` in `ButtonGroup` with a static non-HTML close icon. `src/components/button-group.js:107`. Eliminates a latent XSS vector.

4. **Medium** — Add `npm audit --audit-level=high` to CI and configure Dependabot. No dependency vulnerability scanning is currently automated.

5. **Low** — Replace the `document.body.innerHTML` print pattern with a CSS print media query. `src/scripts/print.js:9`. Reduces DOM mutation scope and eliminates the risk of inadvertently rendering unsanitised content.
