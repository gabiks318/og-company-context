# delonayde_assets

## Role
- Provides reusable loan payment calculation logic as a Python package (`dla`) for Loanwise-related systems.
- Centralizes repayment schedule behavior (annuity, equal-principal, bullet) so downstream services do not duplicate core financial math.

## Responsibilities
- Define loan domain primitives (`LoanBase`, `LoanPayment`, repayment enums).
- Implement loan calculators for different repayment types.
- Implement concrete loan models for:
  - Constant interest
  - Dynamic per-month interest
  - Indexed constant interest
  - Indexed dynamic interest
- Enforce core input contracts (for example, month bounds and schedule length validation).
- Preserve deterministic repayment outputs validated against fixture-based test data.
- Ship versioned package artifacts through the repo CI/CD process.

## Not Responsible For
- Serving HTTP/API endpoints.
- Database persistence or data modeling.
- Authentication/authorization.
- Infrastructure provisioning/runtime orchestration.
- UI presentation logic.
- End-user workflow orchestration (this repo is a computation library, not an application service).

## Dependencies
- `og-cli` pipeline tasks (`og ci`, `og cd`, `og docs deploy`) for CI/CD and docs deployment.
- Internal package feed via AWS CodeArtifact (configured index in `pyproject.toml` and authenticated in Jenkins).
- Internal shared package `og-tech-common`.
- Data tooling dependency `pandas` (used in test/fixture flows).

## Consumers
- `loan-calculation-service` (explicitly triggered as a dependent job from this repo's Jenkins pipeline after `main` updates).
- Other potential internal Loanwise services or jobs importing `dla` (not fully mapped from repository evidence alone).

## Architectural Constraints
- Keep this repository as pure financial calculation/library logic.
- Preserve importable package API compatibility unless a coordinated consumer migration is planned.
- Maintain repayment behavior parity with fixture-backed tests; calculation changes are contract changes.
- Input list lengths (interest/index schedules) must align with loan period, and month access must stay 1..loan_period bounded.
- Avoid embedding service-level concerns (API, persistence, infra) into this package.

## Runtime / Deployment
- Non-`release` branches run CI via `og ci`.
- `release` branch runs CD via `og cd`.
- Documentation is deployed in CI using `og docs deploy`.
- `main` merges are propagated to `release`, then dependent downstream job(s) are triggered.

## Change Impact
- Any change to calculators or loan implementations can alter repayment schedules consumed by downstream services.
- High-risk areas:
  - Interest conversion rules (annual/monthly handling).
  - Index adjustment behavior and principal re-basing.
  - Rounding/tolerance behavior in `LoanPayment` equality.
- Before merge, validate:
  - Full unit test suite for all repayment and implementation variants.
  - Mortgage-sized fixture scenarios (long-tenor schedules) where present.
  - Downstream compatibility for `loan-calculation-service` if behavior changes are intentional.
- If public behavior changes, coordinate rollout and contract expectations with consumers.

## Failure Modes
- `ValueError` on invalid month requests (outside `1..loan_period`).
- `ValueError` when dynamic/indexed schedules do not match loan period length.
- Silent downstream business regressions if formulas change but fixture expectations are not updated intentionally.
- Release/publish interruptions if CodeArtifact or CI credentials are unavailable.
- Contract drift if downstream services rely on legacy repayment semantics not captured in tests.

## Shared Utility Opportunities
- Common validation helpers for schedule-length and month-range checks (currently repeated across implementations).
- Shared numeric/rounding policy abstraction for financial precision behavior.
- Reusable schedule-generation utilities for consistent caching/population patterns.

## Modification Guidance
- Confirm the requested change belongs in shared loan math logic, not in a service-specific layer.
- Prefer extending existing calculator/implementation abstractions over adding parallel repayment logic.
- Treat fixture updates as contract updates; document intent when changing expected payment outputs.
- Preserve backward compatibility by default; use coordinated migration if breaking behavior is required.
- Run and update contract-level tests whenever repayment behavior or validation rules change.
- Do not introduce API, persistence, or infra concerns unless repo scope is explicitly expanded.

## Knowledge Gaps
- Complete downstream consumer inventory is not fully discoverable from this repository alone.
- Production usage scope of each loan implementation (constant/dynamic/indexed variants) is not fully mapped.
- Release artifact destinations and deployment topology are inferred from `og` pipeline commands, not explicitly documented here.

## Notes
- The constant-interest implementation contains explicit test-data-aligned schedule handling for specific parameter sets; this creates an important behavior lock that should be changed carefully.
- `RepaymentType` uses the enum key `EQUAL_PRINCIPLE` (spelling preserved for compatibility); changing this would likely break consumers.

## Related Pages
- `wiki/projects/loanwise.md`
- `wiki/repos/delonayde_assets.md`
