# snikerdoodle

## BrowserStack Accessibility DevTools API surface (evidenced endpoints only)

This document extracts **likely actual** BrowserStack Accessibility-related API endpoints from public artifacts, then ranks them by likely abuse value (IDOR/BAC/config tampering/report manipulation).

### Evidence used
- BrowserStack official MCP server code:
  - `browserstack/mcp-server/src/tools/accessiblity-utils/scanner.ts`
  - `browserstack/mcp-server/src/tools/accessiblity-utils/report-fetcher.ts`
  - `browserstack/mcp-server/src/tools/accessiblity-utils/auth-config.ts`
- BrowserStack AccessibilityDevTools CLI scripts:
  - `browserstack/AccessibilityDevTools/scripts/*/cli.sh`
- Public OpenAPI spec for BrowserStack Accessibility:
  - `shirish87/browserstack-client/packages/openapi/specs/accessibility.yml`

Base host shown in spec: `https://api-accessibility.browserstack.com`

---

## Ranked endpoint targets (highest priority first)

| Priority | Endpoint (method + full path) | Parameterization | Why high-value target | Recommended attack focus |
|---|---|---|---|---|
| 1 | `GET https://api-accessibility.browserstack.com/api/website-scanner/v1/scans/{scan_id}/scan_runs/{scan_run_id}` | Path IDs | Direct scan-run summary retrieval; classic cross-tenant object access risk | IDOR on `scan_id`/`scan_run_id`, weak role checks, horizontal BAC |
| 2 | `GET https://api-accessibility.browserstack.com/api/website-scanner/v1/scans/{scan_id}/scan_runs/issues?scan_run_id=&task_id=&next_page=` | Path + query IDs | Full issue data export surface; supports task and pagination pivots | IDOR via `scan_id` + `scan_run_id`, bulk scraping via pagination, auth bypass |
| 3 | `GET https://api-accessibility.browserstack.com/api/automated-tests/v1/builds/{thBuildId}` | Path ID | Build-level accessibility summaries are high-signal internal QA data | IDOR, cross-project access, role downgrade checks |
| 4 | `GET https://api-accessibility.browserstack.com/api/automated-tests/v1/builds/{thBuildId}/test-cases/{test_case_id}` | Nested path IDs | Fine-grained test-case artifacts amplify report leakage/manipulation value | IDOR on nested IDs, broken object-level authorization |
| 5 | `GET https://api-accessibility.browserstack.com/api/automated-tests/v1/builds/issues?build_id=&task_id=&next_page=` | Query IDs | Bulk issue retrieval endpoint suitable for large-scale exfil | Missing authorization filters, pagination abuse, tenant boundary checks |
| 6 | `GET https://api-accessibility.browserstack.com/api/workflow-analyzer/v1/reports/{report_id}` | Path ID | Report dashboard summary object; often directly referenceable | IDOR, weak project binding, report enumeration |
| 7 | `GET https://api-accessibility.browserstack.com/api/workflow-analyzer/v1/reports/issues?report_id=&task_id=&next_page=` | Query IDs | Report issue export path for workflow analyzer | Report IDOR, bulk extraction, missing row-level authorization |
| 8 | `GET https://api-accessibility.browserstack.com/api/assisted-test/v1/reports/{report_id}` | Path ID | Assisted test report object with execution metadata | IDOR, stale-link access, weak ownership checks |
| 9 | `GET https://api-accessibility.browserstack.com/api/assisted-test/v1/reports/issues?report_id=&task_id=&next_page=` | Query IDs | Issue details export from assisted scans | Pagination scraping, query tampering, weak authz |
| 10 | `POST https://api-accessibility.browserstack.com/api/website-scanner/v1/scans` | JSON body (`name`, `urlList`, optional `authConfigId`, `scanSettings` from SDK code) | Scan triggering/config surface (closest evidenced surface to partial/targeted scans) | Unauthorized scan creation, config tampering, SSRF-style URL validation checks, abuse of `authConfigId` linkage |
| 11 | `GET https://api-accessibility.browserstack.com/api/website-scanner/v1/scans` | Pagination/query | Scan inventory enumeration across account scope | Enumeration + ID harvesting for follow-on IDOR |
| 12 | `GET https://api-accessibility.browserstack.com/api/website-scanner/v1/scans/{scan_id}/scan_runs` | Path + pagination | Historical run listing enables object discovery | Scan/run correlation attacks, mass ID collection |
| 13 | `GET https://api-accessibility.browserstack.com/api/website-scanner/v1/scans/{scan_id}/scan_runs/{scan_run_id}/status` | Path IDs | Operational state endpoint useful for race/timing abuse | Unauthorized polling, state confusion, run-ID probing |
| 14 | `GET https://api-accessibility.browserstack.com/api/website-scanner/v1/scans/{scan_id}/scan_runs/{scan_run_id}/scan_logs` | Path IDs | Crawl logs may leak URLs, redirects, auth flow failures | IDOR, sensitive log exposure, over-broad log access |
| 15 | `GET https://api-accessibility.browserstack.com/api/website-scanner/v1/auth_configs` | Collection | Auth config inventory (high sensitivity if shared across scans) | Enumeration, auth-config ID harvesting |
| 16 | `POST https://api-accessibility.browserstack.com/api/website-scanner/v1/auth_configs` | JSON body (`name`, `type`, `authData`) | Custom auth/config management entry point | Privilege checks for create, malicious config injection, credential handling validation |
| 17 | `GET https://api-accessibility.browserstack.com/api/website-scanner/v1/auth_configs/{configId}` | Path ID | Direct object retrieval for stored auth configs (shown in SDK code) | IDOR against config IDs, secret material exposure |
| 18 | `GET http://api.browserstack.com/sdk/v1/download_cli?os=&os_arch=` | Query params | CLI supply-chain path (binary delivery) from AccessibilityDevTools scripts | Unauthorized binary retrieval patterns, integrity/tamper validation, caching/CDN poisoning checks |

---

## Mapping to requested feature buckets

- **Partial Page Scan configuration/triggering/management**
  - Evidenced trigger/management surfaces: `POST /api/website-scanner/v1/scans`, `GET /api/website-scanner/v1/scans`, `GET /scans/{scan_id}/scan_runs`, `GET /status`.
  - In retrieved public artifacts, no explicit separate `partial-page` REST path is exposed.

- **Accessibility report fetching/downloading/viewing**
  - `GET /api/workflow-analyzer/v1/reports*`
  - `GET /api/assisted-test/v1/reports*`
  - `GET /api/website-scanner/v1/scans/{scan_id}/scan_runs/issues`
  - `GET /api/automated-tests/v1/builds*` and related test-case issue paths

- **Linter rule configuration (CRUD)**
  - No public REST CRUD endpoints for linter rules were evidenced in retrieved sources.
  - Public artifacts indicate local config/CLI behavior, not a documented remote rule-CRUD API path.

- **Project/test suite/test run management, viewing, deletion**
  - Viewing/listing endpoints evidenced under `/api/automated-tests/v1/projects`, `/builds`, `/test-cases`, and website scanner scan/run paths.
  - No documented DELETE endpoints were evidenced in retrieved accessibility sources.

- **User/team/project membership management**
  - No public accessibility API endpoints for membership CRUD were evidenced in retrieved sources.

- **CI integration result upload/download/triggering**
  - Trigger-like: `POST /api/website-scanner/v1/scans`
  - Result retrieval: build/report/issues endpoints above
  - CLI delivery: `GET /sdk/v1/download_cli`

- **Custom component/config management**
  - `GET/POST /api/website-scanner/v1/auth_configs`
  - `GET /api/website-scanner/v1/auth_configs/{configId}`

---

## Practical attack playbook by endpoint class

1. **IDOR/BOLA first:** mutate all path/query IDs (`report_id`, `scan_id`, `scan_run_id`, `thBuildId`, `test_case_id`, `build_id`, `configId`).
2. **Role-boundary tests:** execute same calls as owner, low-priv collaborator, and cross-project user.
3. **Pagination abuse:** iterate `next_page` to validate tenant filtering and rate controls.
4. **Config tampering:** test whether `authConfigId` can reference unauthorized configs when creating scans.
5. **Report manipulation paths:** verify if any write/update companion endpoints exist in runtime traffic even when not publicly documented.
6. **Bulk export controls:** test for mass issue/report extraction without per-object authorization.

