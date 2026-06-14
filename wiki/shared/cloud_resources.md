# cloud_resources

## Role
- Provides `og-tech-common`, a shared Python infrastructure utility package used across internal services and pipelines.
- Centralizes cloud-facing primitives (AWS CodeArtifact/S3/CloudWatch helpers), schema validation utilities, logging conventions, and OpenTelemetry provider setup so downstream repositories avoid duplicating platform logic.

## Responsibilities
- Maintain reusable AWS integration interfaces for package distribution and artifact/storage operations.
- Provide common logging and observability bootstrap utilities for service instrumentation.
- Provide reusable HTTP schema factory/validation utilities based on YAML + Pydantic.
- Maintain shared constants and base utility abstractions consumed by multiple internal repositories.
- Publish and version the package through the standard CI/CD flow for internal distribution.
- Trigger dependent downstream jobs when shared package changes land on main/release path.

## Not Responsible For
- Owning business logic for product repositories that import this package.
- Defining service-specific API endpoints, database models, or UI behavior.
- Acting as the deployment orchestrator itself (downstream repos own deploy runtime behavior).
- Managing organization-wide cloud account policy outside provided helper wrappers.
- Replacing service-level observability design decisions (it provides tooling primitives, not service SLO strategy).

## Consumers
- Internal Python repositories consuming `og-tech-common` from CodeArtifact.
- CI/CD flows that rely on shared package publishing and install behavior.
- Repositories using shared observability/logging/schema utilities (exact full consumer inventory not fully enumerated in-repo).
- Explicit downstream jobs referenced in this repo pipeline (`og-cli`, `delonayde-assets`).

## Dependencies
- AWS services and credentials for CodeArtifact, S3, and CloudWatch interactions.
- Internal package distribution infrastructure (CodeArtifact index/domain).
- Jenkins + `og` pipeline tooling for CI/CD and documentation deployment.
- OpenTelemetry ecosystem for telemetry provider/export setup.
- Pydantic/YAML stack for runtime schema validation utilities.

## Architectural Constraints
- Keep this repository as shared infrastructure utility logic; avoid embedding domain-specific application behavior.
- Preserve backward-compatible interfaces where possible because many downstream services may import these helpers directly.
- Treat AWS wrapper method signatures and behavior as shared contracts; changes can fan out broadly.
- Keep environment-variable contract stability for auth/config paths (CodeArtifact/AWS/OTEL settings).
- Avoid introducing service-specific assumptions into generic logging/observability/schema modules.

## Runtime / Deployment
- Packaged/distributed as `og-tech-common` Python package.
- Branch-based pipeline behavior:
  - non-`release`: `og ci`
  - `release`: `og cd` + docs deployment
  - `main`: merge-to-release and trigger dependent jobs
- Uses internal CodeArtifact index configuration for dependency resolution and package publishing paths.

## Change Impact
- Changes can affect many repositories simultaneously due to shared-package reuse.
- High-impact areas:
  - AWS auth/token/repository endpoint handling in helper interfaces
  - schema validation behavior and dependency rules
  - logging/observability defaults and initialization side effects
  - constants used by deployment/distribution flows
- Before merge, validate:
  - full test suite for aws/general/observability/schema modules
  - compatibility with downstream repos known to consume the package
  - CI package publish/install flows against CodeArtifact
  - no breaking behavior changes in widely used utility interfaces unless coordinated

## Failure Modes
- CodeArtifact auth/token or endpoint regressions break package publish/install across downstream repos.
- AWS credential/config mismatches can break S3/CloudWatch helper operations at runtime.
- Shared schema utility behavior changes can cause validation failures in multiple services.
- Observability provider setup regressions can disable tracing/metrics or introduce runtime initialization issues.
- Shared constant drift (domain/region/bucket assumptions) can cause deployment/distribution path failures.
- Pipeline release failures can block downstream repos waiting for shared utility updates.

## Shared Utility Opportunities
- Introduce explicit stability tiers/versioning policy for high-use utility modules.
- Add integration-contract tests against representative downstream usage patterns.
- Consolidate auth/config resolution patterns across AWS helper modules to reduce divergence.
- Add clearer consumer-facing migration notes when changing behavior in shared primitives.

## Modification Guidance
- Treat every change as potentially multi-repo-impacting; design for compatibility first.
- Prefer extending existing interfaces over replacing semantics in-place.
- Keep cloud helper abstractions generic and reusable; do not add repo-specific policy logic.
- When behavior changes are required, coordinate downstream rollout and update contract-level tests.
- Validate CI publish/install paths in addition to local unit tests.
- Avoid introducing secrets or credential material into repository files/logging.

## Knowledge Gaps
- Full downstream consumer graph for `og-tech-common` is not fully mapped in-repo.
- Current governance/versioning policy for breaking shared-interface changes is not explicitly documented.
- Complete operational ownership split between this package and infra repos (`og-cli`, `og-jenkins`, etc.) is only partially inferred.
- Production dependency criticality ranking by module (aws/schema/observability/logging) is not explicitly documented.

## Notes
- This repository functions as shared platform glue; regressions may surface first in dependent repositories rather than here.
- Pipeline explicitly triggers downstream jobs, indicating tight CI coupling and emphasizing coordinated change discipline.

## Related Pages
- `wiki/infra/dev-tools/og-cli.md`
- `wiki/infra/ci-cd/og-jenkins.md`
- `wiki/repos/delonayde_assets.md`
- `wiki/repos/loan_calculation_service.md`
- `wiki/projects/loanwise.md`
