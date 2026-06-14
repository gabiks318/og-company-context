# loan-calculator-app

## Role
- Delivers the end-user Loanwise client application for creating, editing, and comparing loans with visual repayment insights.
- Serves as the presentation and interaction layer over loan-calculation backends, while supporting an offline calculation mode for no-network usage.

## Responsibilities
- Own the mobile/web user experience for loan input, summaries, payment distribution, and related workflows.
- Manage client-side state, navigation, localization, and accessibility behavior.
- Orchestrate loan calculation requests to backend API environments (staging/production) through repository abstractions.
- Provide an offline fallback calculation path for local/offline scenarios.
- Persist user loan drafts/history in local app storage.
- Emit product analytics and app lifecycle events.
- Integrate client configuration behavior (Remote Config-driven backend base URL in production mode).

## Not Responsible For
- Authoritative loan formula implementation and financial business-rule ownership (handled by backend/shared calculation services).
- API hosting, server-side validation policy, and backend deployment operations.
- Persistent server-side storage of user financial data.
- Infrastructure provisioning and IAM policy management.
- Cross-repo backend contract governance (this app consumes contracts; it does not define backend service internals).

## Consumers
- Loanwise end users on supported Flutter targets (Android, iOS, web).
- Product/operations stakeholders relying on app analytics and behavioral telemetry.
- QA and release pipelines validating the user loan-calculation journey.

## Dependencies
- `loan_calculation_service` API endpoints for network-based loan computation.
- `delonayde_assets` (`dla`) indirectly via backend outputs and contract expectations.
- Shared internal Flutter utility package `og_dart_utils`.
- Firebase platform services (app initialization and Remote Config usage).
- PostHog analytics pipeline.
- CI/CD and secret management through Jenkins + `og` tooling.

## Architectural Constraints
- Keep core financial authority in backend services; client-side calculations are fallback/UX continuity behavior, not source-of-truth business logic.
- Preserve repository abstraction boundaries (UI/BLoC should not directly embed HTTP concerns).
- Maintain environment-mode behavior:
  - `offline` routes to local calculator implementation.
  - non-offline routes to network repository.
- Preserve request/response compatibility assumptions with backend API contracts.
- Preserve accessibility, localization, and navigation stability across feature changes.

## Runtime / Deployment
- Classified as `flutter_app` in `og.yml`.
- Jenkins runs `og ci` on non-`release` branches and `og cd` on `release`.
- Pipeline relies on signing-related secrets for mobile artifact handling.
- App runtime selects behavior by compile-time `ENV` (`offline`, `development`, `staging`, `production`).
- Production backend URL can be overridden by Firebase Remote Config; staging uses fixed API constant.

## Change Impact
- UI/state changes can directly affect loan-entry correctness, user trust, and conversion.
- API contract or repository-mapping changes can break calculation flows or cause subtle result mismatches.
- Offline/online behavior changes can produce inconsistent user-visible outcomes between environments.
- Analytics event/schema changes can break product dashboards and funnel analysis.
- Before merge, validate:
  - Unit tests for BLoC/repository/model behavior.
  - Integration tests covering end-to-end loan creation/calculation/edit/delete flows.
  - Offline mode and network mode parity for core user journeys.
  - Localization/accessibility regressions on key loan screens.

## Failure Modes
- Backend endpoint misconfiguration or availability issues cause remote calculation failures.
- Drift between offline approximation logic and backend authoritative calculations can confuse users.
- Environment mis-selection (`ENV`) can route users to unintended backend/offline paths.
- Remote Config fetch/default behavior can lead to stale or incorrect backend target URL.
- Local persistence corruption/serialization failures can lose saved loan state.
- Analytics/third-party SDK initialization failures can degrade observability or startup behavior.

## Shared Utility Opportunities
- Shared domain-level validation and formatting helpers for loan input constraints across app surfaces.
- Unified API contract test fixtures shared with backend repositories to reduce drift.
- Common error taxonomy/mapping utility between repository errors and user-facing messages.
- Cross-app design-system extraction for reusable financial UI components if similar products emerge.

## Modification Guidance
- First decide whether the change belongs in client UX/state or backend business logic.
- Prefer extending existing module abstractions (`LoanRepository`, DI modules, BLoC flows) over adding parallel pathways.
- Treat backend request/response assumptions as contracts; coordinate with service owners before breaking changes.
- Keep offline fallback explicit and clearly scoped; do not silently alter its authority level.
- Update unit + integration tests when changing calculation flows, state transitions, or repository selection logic.
- Avoid introducing infra/deployment concerns into app feature code unless explicitly required.

## Knowledge Gaps
- Full external consumer map for analytics outputs and event consumers is not documented in-repo.
- Exact production ownership for Remote Config key lifecycle and rollback policy is not explicit.
- Canonical list of all downstream systems depending on app-generated behavioral events is not discoverable from repository evidence alone.
- Business policy for acceptable divergence between offline approximation and backend calculations is not explicitly documented.

## Notes
- The app intentionally supports offline calculations, but this path is an approximation and can diverge from backend authoritative results for advanced loan scenarios.
- Production API endpoint selection is partially runtime-configurable (Remote Config), which adds operational flexibility but also configuration risk.

## Related Pages
- `wiki/projects/loanwise.md`
- `wiki/repos/loan_calculation_service.md`
- `wiki/repos/delonayde_assets.md`
- `wiki/repos/loan-calculator-app.md`
