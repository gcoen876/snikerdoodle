# snikerdoodle
Test site for snikerdoodle.dpdns.org

## BrowserStack Accessibility DevTools: Pentest-First Targets (Black-Box)

Prioritized list of high-risk API/workflow areas to test first when you **do not have source access**.

1. **Report retrieval/export endpoints + dashboard object IDs**  
   - Why high priority: report IDs/build IDs are often enumerable; high IDOR/BAC risk.  
   - Test: swap `report ID/build ID/project ID/test suite ID` across accounts/teams; attempt direct resource access via manipulated URLs; test download/share/export links for unauthorized access.

2. **Test configuration APIs (partial page scan locators, scope, strict/non-strict, rule severity)**  
   - Why high priority: config directly changes scan coverage and CI outcome; strong workflow-abuse impact.  
   - Test: unauthorized updates to another project’s config; mass-assignment of unexpected parameters; inject malformed selector/rule payloads; attempt policy bypass by forcing non-strict mode.

3. **Partial page scan recorder flow (save/edit/delete scoped component entries)**  
   - Why high priority: attacker can hide real issues by narrowing scope to safe DOM fragments.  
   - Test: IDOR on component-scan entries; tamper `url + locator` pair; race save/delete requests; inject deeply nested or malformed selectors (parser resource-exhaustion/Denial of Service behavior).

4. **Automation scan commands / SDK ingestion endpoints (`performScan`, `startA11yScanning`, `stopA11yScanning`)**  
   - Why high priority: accepts user-controlled scope/selectors from CI pipelines and test code.  
   - Test: payload fuzzing in selector arrays, nested objects, oversized JSON; replay old signed requests/tokens; cross-project token misuse.

5. **Build/run association endpoints (linking test run to accessibility report)**  
   - Why high priority: broken object binding can leak reports between orgs or poison result attribution.  
   - Test: mutate run IDs during report finalization; submit results to foreign build IDs; parallel-run race to overwrite latest report.

6. **Organization/project membership + role/permission APIs**  
   - Why high priority: common BAC escalation path (viewer → editor/admin).  
   - Test: invite acceptance tampering, role-change endpoint abuse, stale session privilege retention, cross-org project listing by ID guessing.

7. **CLI/NPM/IDE plugin authentication and token handling workflows**  
   - Why high priority: long-lived tokens are frequently overprivileged and portable across contexts.  
   - Test: use token issued for one workspace on another; test revoked/rotated token acceptance windows; inspect whether report/config endpoints honor least privilege.

8. **CI/CD integration endpoints/webhooks/status callbacks**  
   - Why high priority: can be abused to force green pipelines, suppress failures, or inject false compliance signals.  
   - Test: replay/build-status callback spoofing, unsigned webhook acceptance, race between failing and passing status updates, branch-context confusion.

9. **File/config upload/import paths (YAML/JSON project config)**  
   - Why high priority: parser attack surface + deserialization/injection opportunities.  
   - Test: YAML alias bombs, JSON keys such as `__proto__`/`constructor`, command/template injection probes in rule IDs, path traversal in referenced config includes.

10. **Search/filter/query endpoints on Accessibility dashboard**  
    - Why high priority: broad query surfaces often expose SQL/NoSQL/ORM injection and data overexposure.  
    - Test: filter/sort parameter fuzzing, wildcard/object-operator injection, pagination abuse for cross-tenant data bleed.

### Attack patterns to run across all targets
- **IDOR matrix:** iterate predictable IDs; switch org/project/run/report IDs in every request.  
- **BAC checks:** compare viewer/editor/admin/API-token behaviors per endpoint.  
- **Workflow abuse:** toggle strict/non-strict, scoped scan settings, and report-binding order during concurrent runs.  
- **Injection/fuzzing:** selectors, rule IDs, config blobs, query params, export formats, webhook payloads.  
- **Race/replay:** duplicate finalize/update calls, delayed retries, replay signed requests, out-of-order state transitions.
