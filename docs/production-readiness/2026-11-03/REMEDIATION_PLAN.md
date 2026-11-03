# rocketboardfeedback remediation plan

This is future work, not implemented or verified. See [the pinned review](REVIEW.md) for findings and evidence. All tests, drills and commands below are planned; no target code was executed.

## Phases and dependencies

| Phase | Entry | Exit |
|---|---|---|
| Baseline | Owner authorizes isolated investigation | T1 records architecture, intent and safe reproduction boundaries. |
| Defects and security | T1 plus access-design approval | T2 and T3 demonstrate isolation and truthful persistence; T5 completes security review. |
| Automation and experience | T1 and relevant defect work | T4 and T6 produce meaningful gates and observed UX/accessibility outcomes. |
| Operations and acceptance | Owner supplies staging, access and recovery targets | T7 observes release and restore; independent reviewer completes T8. |

| Task | Linked finding / requirement | Dependencies | Deliverable |
|---|---|---|---|
| RBF-T1 | RBF-003, baseline | owner authorization | boundary and product inventory |
| RBF-T2 | RBF-004 | T1, owner-approved model | isolated access and privacy evidence |
| RBF-T3 | RBF-005 | T1, T2 access design | durable, truthful save behavior |
| RBF-T4 | RBF-001/002 | T1; T2/T3 for regression coverage | deterministic CI and behavioral tests |
| RBF-T5 | preventive security | T1, T2 | independent security assessment |
| RBF-T6 | preventive UX/accessibility | T1, T3 | observed flow and accessibility assessment |
| RBF-T7 | preventive operations/recovery | T1, T2; owner staging access | release and observed restore evidence |
| RBF-T8 | independent acceptance | T2–T7 | independent negative-test and handoff record |

### RBF-T1 — Establish actual boundary and intent
**Paths:** `src/App.tsx`, `src/pages/FeedbackFlow.tsx`, `src/pages/Index.tsx`, `src/hooks/useFeedbackSession.ts`, `README.md`; proposed: owner-approved architecture notes. **Prerequisites:** owner authorizes static inspection and isolated baseline; no production data.
1. Trace route, chapter components, client configuration and actual consumers of `Index.tsx`; record what remains uncertain.
2. Confirm product scope, persistence requirements and whether the starter page has a legitimate use before changing or removing it.
3. Update README with the actual questionnaire and owner-confirmed setup/deployment; distinguish vision from implemented functionality.

**Deliverable:** source-backed inventory and corrected product description. **Verification:** another maintainer traces the root route and verifies any disposition of `Index.tsx` against consumers. **DoD:**
- [ ] Boundary and unknowns recorded.
- [ ] Owner decides starter-page disposition.
- [ ] README matches confirmed behavior and setup.

### RBF-T2 — Design and prove response isolation
**Paths:** `supabase/migrations/20260311125937_605e594f-10de-4f2e-9eb3-e63f070d72db.sql`, `src/integrations/supabase/client.ts`, `src/integrations/supabase/types.ts`, `src/lib/supabase.ts`, `src/pages/FeedbackFlow.tsx`; proposed: new migration and privacy notice. **Prerequisites:** T1; owner chooses roles, authentication/session ownership, intended submissions and who may read aggregates.
1. In authorized staging, inspect actual client/API configuration, deployed policies, table grants and applicable roles without recording credentials or respondent data.
2. Design and implement least-privilege access for all three tables, including ownership and intended aggregate access; review migration and rollback with the owner. Do not rely on an unguessable session ID alone as authorization.
3. Agree data minimization, retention/deletion, consent and truthful confidentiality copy; isolate synthetic test identities and data.

**Deliverable:** approved policy/grant migration and privacy contract. **Verification:** test two separate sessions and relevant roles: intended submission succeeds; cross-session reads and writes fail; unauthorized access to each table fails; approved aggregate access works. Inspect effective staging grants and policies after applying the migration, and record observed results. **DoD:**
- [ ] Effective grants and policies documented without secrets.
- [ ] Positive and negative isolation tests pass for each table and role.
- [ ] Consent, retention and access model approved; production application separately authorized.

### RBF-T3 — Make saves truthful and recoverable
**Paths:** `src/hooks/useFeedbackSession.ts`, `src/pages/FeedbackFlow.tsx`, `supabase/migrations/20260311125937_605e594f-10de-4f2e-9eb3-e63f070d72db.sql`. **Prerequisites:** T1; T2 access design; owner decides whether local-only continuation is allowed.
1. Inspect actual client/API return conventions, schema constraints and staging state before choosing a persistence or idempotency mechanism.
2. Handle returned errors and thrown exceptions distinctly for creation, answers, summary and completion; surface pending, failed and saved status, and label any local fallback clearly.
3. Design safe retry/recovery, including partial completion and duplicate prevention, without claiming a report was durably submitted prematurely.

**Deliverable:** explicit save-state contract, implementation and failure-path tests. **Verification:** inject returned errors and thrown failures at each operation, check UI and synthetic database state, retry after recovery and confirm no duplicate answers or summary. **DoD:**
- [ ] Local fallback is never represented as a confirmed save.
- [ ] Each failure and partial completion produces a truthful status.
- [ ] Retry and successful completion are independently verified against database state.

### RBF-T4 — Build meaningful automation and supply-chain baseline
**Paths:** `.github/workflows/ci.yml`, `package.json`, `src/test/example.test.ts`, `vitest.config.ts`, `bun.lock`, `bun.lockb`, `package-lock.json`; proposed: behavior tests and optional new scripts. **Prerequisites:** T1, and T2/T3 interfaces for their regression tests; owner chooses authoritative package manager and gate policy.
1. Determine lockfile authority and integrity, review moving Bun/action references, and select reproducible versions without assuming existing lockfiles agree.
2. Add route, mode, save-failure and isolation behavior tests; retain independent assertions against expected user-visible results, not copied implementation logic.
3. Run existing lint, test and build scripts in CI; if format or type checks are required, define and verify scripts first. Configure required checks only with owner approval.

**Deliverable:** reproducible CI and behavior-focused tests. **Verification:** in an isolated branch, deliberately break an assertion, route and lint rule separately and observe failing checks; restore and observe success. **DoD:**
- [ ] Lockfile and tool-version policy documented.
- [ ] Meaningful behavior and failure-path tests execute in CI.
- [ ] Deliberate failures are detected; no check is weakened to pass.

### RBF-T5 — Review security beyond policy predicates
**Paths:** `src/pages/FeedbackFlow.tsx`, `src/components/feedback/chapters/`, `src/components/feedback/InsightReport.tsx`, `src/integrations/supabase/client.ts`, `.env`, `package.json`; proposed: private security assessment. **Prerequisites:** T1 and T2; owner authorizes isolated inspection and provides a reviewer other than the implementer.
1. Trace authentication, input validation, storage and output rendering across real chapters and reports; test injection and authorization boundaries using synthetic data.
2. Inspect secret provisioning and shipped artifacts without copying values; distinguish public client identifiers from privileged credentials, and assess dependency integrity and advisories.
3. Document risks, evidence and fixes or owner-approved deferrals; privately hold genuinely restricted additional evidence and exclude credentials, personal data and operational exploit payloads from public records.

**Deliverable:** independent, redacted security findings and disposition. **Verification:** reviewer demonstrates relevant negative cases in staging and confirms sensitive values and personal data are absent from the record. **DoD:**
- [ ] Input/output, auth and authorization reviewed.
- [ ] Secrets and dependency risks assessed without disclosure.
- [ ] Findings resolved or explicitly accepted by owner before release.

### RBF-T6 — Test respondent experience and accessibility
**Paths:** `src/pages/FeedbackFlow.tsx`, `src/components/feedback/chapters/`, `src/components/feedback/ProgressBar.tsx`, `src/components/feedback/InsightReport.tsx`; proposed: accessibility test records. **Prerequisites:** T1 and T3; representative staging data and accessible test environment.
1. Exercise fast and deep modes, back/continue, completion and save-failure/retry with synthetic responses.
2. Inspect keyboard navigation, visible focus, labels, screen-reader announcements and responsive layouts across chapters and error states.
3. Correct or document observed barriers and verify that consent and save status are understandable without relying on color alone.

**Deliverable:** reproducible accessibility and usability results with issue dispositions. **Verification:** independent tester repeats affected flows, including keyboard and screen reader, and confirms errors are perceivable and actionable. **DoD:**
- [ ] Both paths and recovery states exercised.
- [ ] Accessibility barriers addressed or owner-approved for deferral.
- [ ] Consent and submission status are verified with users or representative reviewers.

### RBF-T7 — Prove release and recovery
**Paths:** `README.md`, `supabase/migrations/`, `.github/workflows/ci.yml`; proposed: owner-approved release and recovery runbook. **Prerequisites:** T1/T2, designated operations owner, isolated staging, approved backup access and recovery objectives.
1. Inventory deployment target, credentials ownership, migration ordering, monitoring and backup scope; establish measurable recovery objectives with the owner.
2. Document staged release, application rollback and database forward/restore decision points; protect backup access and avoid production data in drills.
3. Create a staging backup and restore it into an isolated target, observing duration, schema, policies and synthetic response integrity.

**Deliverable:** release/runbook and dated restore-drill record. **Verification:** an operator other than the author follows the runbook and compares restored data, access controls and measured recovery times to agreed objectives. **DoD:**
- [ ] Owners, monitoring and release/rollback decisions documented.
- [ ] Backup and isolated restore actually observed.
- [ ] Restored data, permissions and recovery objectives checked.

### RBF-T8 — Independent acceptance
**Paths:** T2–T7 artifacts and observed repository paths above; proposed: acceptance record. **Prerequisites:** all preceding tasks completed or documented as owner-approved deferrals; independent reviewer has staging access.
1. Compare the deployed candidate revision and effective staging configuration to the reviewed artifacts and owner-approved scope.
2. Deliberately break representative route/save behavior and an isolation rule in a disposable environment; confirm corresponding independent tests fail, then restore and rerun.
3. Review UX, security, CI, release and actual restore evidence; record unresolved risks and request an explicit owner release decision.

**Deliverable:** signed, revision-bound acceptance record, not a test-count claim. **Verification:** independent reviewer observes failures and recovery, validates source/config identity and checks the restore record. **DoD:**
- [ ] Deliberate regressions are detected and restored.
- [ ] Candidate revision and staging configuration match evidence.
- [ ] Owner accepts remaining risk or blocks release.

## Production acceptance, approvals and rollback

- [ ] Effective role/grant isolation, consent and retention are approved and observed.
- [ ] Failure/retry tests prove truthful persistence and avoid duplicates.
- [ ] CI, accessibility, independent security and actual restore evidence are reviewed.
- [ ] Deployment owner, monitoring, rollback and release decision are recorded.

Owner approval is required for access model, migrations, retention, release infrastructure, lockfile policy, required checks and any deletion or route change. Never weaken checks to obtain green. Roll back application changes only through an approved release procedure; database rollback versus forward fix or restore requires an assessed, tested plan to avoid data loss. The owner's existing authorization for defensive documentation and draft PRs is recorded in `fleet-hardening-review/PUBLICATION_AUTHORIZATION.md`; no new per-finding consent is required. Before publication, exclude real credentials, personal data and operational exploit payloads, and privately hold genuinely restricted additional evidence. The documentation authorization does not authorize application changes, deployment, live tests or merges.
