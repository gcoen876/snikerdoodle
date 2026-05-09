# BrowserStack Accessibility DevTools: Concrete Endpoints & URLs (Official Sources)

This document lists only real routes/URLs explicitly shown in official BrowserStack-owned materials (docs/readmes/source in `browserstack/AccessibilityDevTools` and `browserstack/mcp-server`).

## 1) Explicit API endpoints

| Endpoint | Method(s) shown | Auth in docs/examples | Parameters / identifiers shown | Security tester focus |
|---|---|---|---|---|
| `https://api.browserstack.com/sdk/v1/download_cli` (`/sdk/v1/download_cli`) | `GET` (CLI scripts), `HEAD` then `GET` (plugin redirect resolution) | None shown in request headers | `os`, `os_arch` query params | Official scripts/plugins currently show the same route with `http`; verify strict redirect/upgrade to HTTPS and ensure no credentials are ever sent; also validate binary integrity/signature and query-param tampering (`os`, `os_arch`) controls |
| `https://api-accessibility.browserstack.com/api/website-scanner/v1/scans` | `POST` | `Authorization: Basic base64(username:access_key)` | Body fields shown: `name`, `urlList[]`, `recurring`, optional `authConfigId`, optional `scanSettings.advancedRules`, optional `localTestingInfo.localIdentifier/localEnabled` | AuthZ by account/team scope, SSRF/open-redirect risk via `urlList`, duplicate-name handling, local-testing boundary controls, rate limits |
| `https://api-accessibility.browserstack.com/api/website-scanner/v1/scans/{scanId}/scan_runs/{scanRunId}/status` | `GET` | Basic auth | Path params: `scanId`, `scanRunId` | IDOR/BOLA on `scanId`/`scanRunId`, status leakage across tenants, brute-force resistance |
| `https://api-accessibility.browserstack.com/api/website-scanner/v1/scans/{scanId}/scan_runs/issues` | `GET` | Basic auth | Query shown: `scan_run_id` (init report task) or `task_id` (poll task result) | Task ID predictability/replay, report-link exposure, authorization checks across scan/task IDs, pagination/DoS behavior |
| `https://api-accessibility.browserstack.com/api/website-scanner/v1/auth_configs` | `POST` | Basic auth | Body fields shown: `name`, `type` (`form`/`basic`), `authData` (`url`, `username`, `password`, selectors for form auth) | Secret handling (credential storage/echoing), selector injection abuse, auth config ownership checks |
| `https://api-accessibility.browserstack.com/api/website-scanner/v1/auth_configs/{configId}` | `GET` | Basic auth | Path param: `configId` | IDOR/BOLA for auth configs, sensitive field masking in response, audit logging |

## 2) Explicit workflow / UI / developer URLs

| URL | Type | Auth context | Parameters / identifiers shown | Security tester focus |
|---|---|---|---|---|
| `https://scanner.browserstack.com/site-scanner/scan-details/{name}` | Scan details UI link surfaced by official MCP tool output text | BrowserStack session/login implied | Path token `{name}` = scan name used when starting the scan | Authorization checks on scan name, enumeration risk, cross-tenant visibility |
| `https://www.browserstack.com/accounts/profile/details` | Account/Profile UI (where username/access key are retrieved) | Logged-in BrowserStack account | None | Session security, exposure of access key, anti-CSRF for key actions |
| `https://www.browserstack.com/users/sign_in` | Login UI | Public login endpoint | None | Brute-force/rate limit/MFA controls, account lockout, session fixation |
| `https://www.browserstack.com/docs/accessibility-dev-tools/xcode-linter#CLI` | Official workflow documentation URL for CLI path | Public docs | Fragment `#CLI` | Content integrity and supply-chain trust in setup instructions |

## 3) Authentication model explicitly shown

- Accessibility scan/auth-config APIs use **HTTP Basic Auth** with BrowserStack **username/access key**.
- Credentials are documented as environment variables:
  - `BROWSERSTACK_USERNAME`
  - `BROWSERSTACK_ACCESS_KEY`
- For `scan-details/{name}`, cited sources do not document exact encoding/transformation rules for scan names; test URL-encoding behavior explicitly.

## 4) Official source locations used

- BrowserStack Accessibility DevTools (official):
  - `https://github.com/browserstack/AccessibilityDevTools`
  - `scripts/bash/cli.sh`, `scripts/zsh/cli.sh`, `scripts/fish/cli.sh`
  - `Plugins/BrowserStackAccessibilityLint/BrowserStackAccessibilityLint.swift`
  - `README.md`
- BrowserStack MCP Server (official):
  - `https://github.com/browserstack/mcp-server`
  - `src/tools/accessibility.ts`
  - `src/tools/*` (scanner/auth/report utility implementations used by the accessibility tool)
