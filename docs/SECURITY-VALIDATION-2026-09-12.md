# Security Validation Record - 2026-09-12

## Purpose

This record documents the defensive validation of the public static landing page at `https://iaton.ia.br`. It is a point-in-time assessment, not a guarantee of invulnerability or a substitute for continuous security review.

## Assessed architecture

At the time of testing, the site was a public static landing page hosted on Vercel. The assessed page had no authentication, application database, server-side API, cookies, user-input form, upload capability, or JavaScript.

## Evidence and results

### Local defensive baseline

A non-invasive PowerShell audit validated HTTPS availability, TLS 1.3 negotiation, certificate identity and validity, rejection of TRACE, security response headers, and absence of public access to common sensitive paths such as `/.env` and `/.git/config`.

### External independent checks

- Security Headers: grade A+ after hardening.
- MDN HTTP Observatory: grade A+, score 130/100, 12 of 12 tests passed after hardening.
- Qualys SSL Labs: grade A on both tested edge endpoints.

### OWASP ZAP Baseline

The passive OWASP ZAP Baseline workflow is stored at `.github/workflows/zap-baseline.yml`.

Initial run `34717044074` found:

- High: 0
- Medium: 2
- Low: 1
- Informational: 7

The actionable findings concerned inline CSS in the CSP, permissive CORS, and a missing Cross-Origin-Embedder-Policy header.

Corrections were published in commit `df1592187081a774458e184ff68cfa04afc333ad`:

- CSS was moved to `styles.css`.
- `unsafe-inline` was removed from `style-src`.
- CORS was restricted to `https://iaton.ia.br`.
- `Cross-Origin-Embedder-Policy: require-corp` was added.

The follow-up passive scan found:

- High: 0
- Medium: 0
- Low: 0
- Informational: 7

The remaining informational observations relate to caching behavior and request-side `Sec-Fetch-*` headers. They were reviewed and do not require a site change for the current public static content.

## Implemented controls

- HTTPS with HSTS
- Restrictive Content Security Policy
- Clickjacking protection
- MIME sniffing protection
- Referrer Policy
- Permissions Policy
- COOP, CORP, and COEP isolation headers
- Same-origin CORS policy
- External stylesheet without inline-style allowance
- Reproducible passive ZAP workflow in GitHub Actions

## Evidence handling

Generated scanner reports are retained as GitHub Actions artifacts and in local ignored evidence directories. Raw generated reports are not committed to the source tree. This avoids unnecessary repository churn while preserving the reproducible workflow and execution history.

## Retest triggers

Repeat the baseline and external checks before or after adding JavaScript, forms, APIs, authentication, cookies, analytics, third-party assets, or material hosting and DNS changes.
