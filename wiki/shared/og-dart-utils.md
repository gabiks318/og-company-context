# og-dart-utils

## Role
- Provides a shared Dart/Flutter utility package (`og_dart_utils`) that centralizes reusable application primitives used across internal mobile/client projects.
- Reduces duplicate app bootstrap and cross-cutting concerns (Firebase lifecycle, analytics wrappers, request metadata, DI helpers) so product repositories can focus on domain behavior.

## Responsibilities
- Maintain reusable service abstractions for Firebase initialization, authentication lifecycle, and Remote Config access.
- Provide shared analytics interfaces and implementations (including PostHog and no-op options) for consistent client telemetry integration.
- Provide reusable network request metadata primitives (header/request identity/session helpers).
- Provide shared DI/module and lightweight routing/dev-support primitives used by app-layer code.
- Publish and version the package through the organization CI/CD workflow for downstream app consumption.
- Trigger known dependent jobs after main-branch updates to keep consumer pipelines aligned.

## Not Responsible For
- Owning product-specific business logic, screen flows, or feature decisions in consuming apps.
- Defining backend API contracts or server-side authorization behavior.
- Managing infrastructure provisioning or deployment environments.
- Replacing app-owned configuration decisions (for example, environment mapping, key ownership, or routing policy).
- Acting as a full application framework; it provides shared building blocks, not end-to-end app architecture.

## Consumers
- Internal Flutter/Dart applications importing `og_dart_utils`.
- Explicit downstream CI consumer referenced in this repo pipeline: `loanwise-app`.
- App bootstrap/auth/config layers that reuse Firebase and analytics helpers from this package.
- Additional downstream consumer inventory is likely broader but not fully enumerated in-repo.

## Dependencies
- Firebase platform services and SDKs (Core/Auth/Remote Config) for shared client-side integration paths.
- PostHog client integration for analytics implementation.
- Internal CI/CD toolchain: Jenkins shared library + `og` CLI flow.
- Organization GitHub/Jenkins credential and branch workflow conventions used for build/release automation.

## Architectural Constraints
- Keep shared logic generic and reusable; avoid app-specific policy/feature rules in this package.
- Preserve stable public interfaces because downstream apps may directly depend on exported service contracts.
- Keep SDK integrations behind shared abstractions so consumer apps can test and evolve independently.
- Do not couple shared services to a single app environment strategy; consuming apps own environment and key mapping decisions.
- Treat branch-driven CI behavior (`main`/`release`) as an operational contract for publish and downstream trigger flows.

## Runtime / Deployment
- Distributed as a Dart package (`og_dart_utils`) consumed by Flutter/Dart repositories.
- Jenkins pipeline behavior:
  - non-`release` branches: run `og ci`
  - `release` branch: run CD stage (release path)
  - `main` branch: merge to `release` and trigger dependent jobs (`loanwise-app/main`)
- Uses containerized pipeline execution via shared Jenkins handlers and organization credentials.

## Change Impact
- Changes can affect multiple client apps simultaneously when shared service contracts or behavior change.
- High-risk areas:
  - auth lifecycle semantics and token/persistence expectations
  - Remote Config initialization/fetch behavior
  - analytics event plumbing interfaces
  - request metadata/header generation behavior
- Before merge, validate:
  - package tests and any contract-level tests around shared service behavior
  - compatibility with known downstream app integration points
  - CI/CD publish and downstream-trigger behavior for `main`/`release` paths
  - migration notes for any behavioral or API-level breaking changes

## Failure Modes
- Shared contract drift can break app bootstrap/auth flows in multiple repositories at once.
- Firebase initialization/auth/Remote Config behavior regressions can cause startup/login/config failures across consumers.
- Analytics abstraction changes can silently drop or mis-shape telemetry events.
- Request metadata or header-generation regressions can break backend request tracing/correlation.
- CI/CD credential or branch-flow issues can block package delivery and delay dependent application pipelines.

## Shared Utility Opportunities
- Formalize and version consumer-facing integration contracts for auth/config/analytics services.
- Add cross-app integration fixtures or contract tests for common bootstrap/auth scenarios.
- Expand documentation for recommended migration patterns from app-local utilities to package abstractions.
- Add clearer release notes policy for behavior changes in shared exported modules.

## Modification Guidance
- Confirm first whether new logic belongs in shared package scope or should stay app-local.
- Prefer extending existing abstractions instead of adding parallel service paths.
- Maintain backward compatibility for exported APIs unless a coordinated consumer migration is planned.
- Validate at least one representative downstream app integration path for high-impact changes.
- Keep app-specific environment, key naming, and routing decisions out of shared code.
- Coordinate CI/release expectations when changing branch-triggered delivery behavior.

## Knowledge Gaps
- Full list of downstream repositories consuming `og_dart_utils` is not explicitly documented here.
- Exact artifact publication destination/registry details for this Dart package are not fully explicit in-repo.
- Current governance/versioning policy for breaking shared API changes is not documented.
- Production criticality ranking of individual modules (Firebase vs analytics vs networking vs DI/routing) is only partially inferable.

## Notes
- The Firebase module README explicitly emphasizes keeping app-level ownership for environment and key decisions, reinforcing a strict shared-vs-app boundary.
- Pipeline-triggered dependent jobs indicate operational coupling, so coordinated communication is important when changing shared behavior.

## Related Pages
- `wiki/projects/loanwise.md`
- `wiki/repos/loan-calculator-app.md`
- `wiki/infra/ci-cd/og-jenkins.md`
- `wiki/infra/dev-tools/og-cli.md`
- `wiki/shared/cloud_resources.md`
