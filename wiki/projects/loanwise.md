# Loanwise

## System Overview
- Loanwise is a multi-repository loan analysis system that delivers user-facing loan planning experiences and repayment insights while centralizing financial calculation rules in shared backend components.
- End-to-end capability: users model loans in the app, the system computes repayment schedules through API orchestration over shared loan logic, and results are returned for visualization and comparison.
- The architecture separates concerns across UI experience, API orchestration, and reusable financial computation to reduce duplication and preserve contract consistency.

## System Components
### Core Services
- `wiki/repos/loan_calculation_service.md` - HTTP calculation API (Lambda + API Gateway) that validates requests and orchestrates schedule generation.
- `wiki/repos/delonayde_assets.md` - shared `dla` financial calculation library implementing repayment models and core contracts.

### Frontend / Interfaces
- `wiki/repos/loan-calculator-app.md` - Flutter client application (Android/iOS/web) for loan input, simulation, and repayment visualization.

### Supporting Services
- No dedicated background worker repository is currently documented in the provided project set.

### Shared Libraries
- `delonayde_assets` (`dla`) acts as the shared business-logic contract layer used by backend API flows.

### Infrastructure
- Jenkins + `og` pipelines across all three repositories (`og ci` / `og cd` branch-driven execution model).
- AWS runtime stack for core API path (API Gateway + Lambda).
- Internal package infrastructure (CodeArtifact) used for artifact/dependency distribution.
- Mobile signing/secret handling in app CI pipeline.
- Observability/analytics surfaces: OpenTelemetry (service) and PostHog/Firebase integrations (app side).

## System Architecture Flow
- Runtime request flow:
  - User interacts with `loan-calculator-app`.
  - In online environments, app sends calculation request to `loan_calculation_service` endpoint.
  - API validates query contract and routes to the relevant loan model.
  - API invokes `dla` repayment logic to compute schedule.
  - API returns normalized payments to app for rendering and comparison UX.
- Offline runtime branch:
  - App can switch to local/offline calculation path for continuity when network/backend is unavailable.
  - This branch preserves UX continuity but is not the authoritative business source.
- CI/CD flow:
  - Each repository runs Jenkins pipeline with branch-based `og` commands.
  - `delonayde_assets` main-to-release propagation can trigger dependent downstream jobs (including loan API refresh path).
  - Service/app deployment lifecycles are repository-local but operationally coupled through shared contracts.

## Repository Interactions
- `loan-calculator-app` -> `loan_calculation_service`:
  - Contracted by request/response schema and repayment enum/value expectations.
  - Breaking API contract changes propagate immediately to user-facing flows.
- `loan_calculation_service` -> `delonayde_assets`:
  - Service delegates core math and schedule semantics to shared library.
  - Library changes can alter API outputs even when service code is unchanged.
- `loan-calculator-app` -> `delonayde_assets` (indirect):
  - App behavior and user trust depend on consistency of outputs ultimately produced by `dla` through service layer.
- Release coupling:
  - Shared library release cadence and compatibility discipline influence service correctness and, transitively, app behavior.

## Deployment & Runtime Model
- Deployment strategy is repository-centric but coordinated by common patterns:
  - Jenkins pipeline orchestration.
  - `og ci` on non-release branches; `og cd` on release branches.
- API runtime is AWS-managed serverless (Lambda + API Gateway).
- App runtime behavior changes by compile-time environment (`offline`, `development`, `staging`, `production`) plus production remote configuration for endpoint targeting.
- System environments exist across at least alpha/staging/prod-style paths, but full promotion topology is not fully documented in one place.

## Critical Dependencies
- AWS API Gateway/Lambda availability for online calculations.
- CodeArtifact and CI credentials for build/release continuity across service/library repos.
- Stable cross-repo calculation contract (loan types, repayment types, payload formats).
- Firebase Remote Config correctness for production endpoint resolution in app.
- Observability dependencies (OpenTelemetry metrics/traces and analytics pipelines) for operational insight.

## System-Level Constraints
- Financial authority boundary:
  - Authoritative repayment semantics belong to shared/backend path (`dla` via API), not UI.
- Contract stability requirement:
  - Query/response schemas and enum naming conventions must remain backward compatible unless migration is coordinated.
- Responsibility segregation:
  - UI handles presentation/workflow.
  - Service handles API validation/orchestration.
  - Shared library handles core calculation semantics.
- Cross-repo change discipline:
  - Behavior-changing updates in `dla` require downstream validation in service and app journeys.

## End-to-End Workflows
- User loan simulation workflow:
  - User enters loan data in app -> app chooses online/offline mode -> online calls service -> service validates and computes via `dla` -> app presents schedule insights.
- Shared logic evolution workflow:
  - Library behavior update -> library CI/CD release path -> service compatibility validation -> app scenario regression checks -> coordinated rollout if output semantics change.
- Delivery workflow:
  - Repo-local CI checks -> branch-based CD -> runtime environment availability for app/API -> post-deploy validation through integration/user-journey tests.

## Failure Modes (System Level)
- Contract mismatch cascade:
  - Change in `dla` semantics or service schema -> API output drift -> app rendering/business interpretation mismatch.
- Environment/configuration drift:
  - Incorrect app endpoint routing (ENV/Remote Config) -> calls wrong environment or stale backend.
- Shared infrastructure outage:
  - CodeArtifact/auth/Jenkins issues block release chain across dependent repositories.
- Partial rollout inconsistency:
  - Library/service version skew can produce unexpected schedule outputs across environments.
- Degraded observability:
  - Metric/event contract changes can hide incidents and delay cross-repo diagnosis.
- Offline/online divergence:
  - App fallback approximation differs from backend authoritative results, causing trust/support issues.

## Shared Knowledge / Abstractions
- Strong existing abstraction: `dla` as shared financial contract layer.
- Opportunity: shared API contract fixtures usable by both service and app integration tests to reduce drift.
- Opportunity: shared cross-repo error taxonomy/mapping for consistent user and operational semantics.
- Opportunity: centralized documentation/ownership for environment and endpoint governance to reduce routing ambiguity.

## Knowledge Gaps
- Full production consumer map for `loan_calculation_service` is not explicitly documented.
- Canonical environment promotion path and release governance across app/service/library are only partially inferred.
- Ownership model for Remote Config key lifecycle and emergency rollback is not explicit.
- Documented SLO/alerting linkage across app analytics and service telemetry is not available in the provided pages.
- Complete list of auxiliary systems (dashboards, batch jobs, additional services) interacting with Loanwise is incomplete.

## Evolution & Change Strategy
- Add new capabilities by preserving current layering:
  - extend `dla` for new reusable financial primitives,
  - expose/compose via service contract,
  - consume in app UX.
- Require contract-first change management for any schema or repayment semantic updates.
- Treat shared-library behavior changes as system changes, not isolated repo updates.
- Prefer coordinated multi-repo validation gates (library fixtures, service integration checks, app journey tests) before broad rollout.
- Expand system safely by introducing new repos as clearly scoped components (core service, supporting service, interface, or shared abstraction) with explicit boundaries and dependency contracts.

## Related Projects
- `wiki/repos/loan-calculator-app.md`
- `wiki/repos/loan_calculation_service.md`
- `wiki/repos/delonayde_assets.md`
