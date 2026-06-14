# og-jenkins

## Role
- Provides the shared Jenkins pipeline library used to standardize CI/CD behavior across company repositories.
- Centralizes reusable pipeline primitives (node/container execution, AWS/Git operations, notification flows, and helper utilities) so repo Jenkinsfiles compose common infrastructure logic instead of reimplementing it.

## Responsibilities
- Expose Jenkins Global Library entrypoints (`vars/*`) that repository pipelines call (`getNodeHandler`, `getAwsHandler`, `getGitHandler`, etc.).
- Manage execution environments for pipelines, including containerized execution on specific node labels and ECR image pulls.
- Provide AWS helper operations needed by pipelines (authentication, CodeArtifact logins, package metadata queries).
- Provide Git helper operations (clone/checkout, branch handling, release-merge helpers, PR creation helpers).
- Provide notification and status propagation helpers (Telegram notifications and optional orchestrator webhooks).
- Provide shared constants and mappings for branch semantics, node names, and supported runner images.
- Offer utility methods for safe temporary workspace management during pipeline runs.

## Not Responsible For
- Owning product-specific build/test/deploy business logic for individual repositories.
- Defining application runtime architecture or service-level contracts.
- Acting as the source-of-truth for each repository’s delivery policy decisions.
- Managing infrastructure resources directly outside what downstream pipelines invoke.
- Replacing project-level CI/CD validation or integration test design.

## Consumers
- Repository Jenkins pipelines that import `@Library('og-jenkins-library')` and call exported handlers.
- Internal repos using OG-standard CI/CD patterns (Python package repos, Lambda/API repos, Flutter repos, and shared libraries).
- Operational notification channels receiving pipeline status events via Telegram/orchestrator integrations.

## Dependencies
- Jenkins Shared Library runtime and pipeline execution context.
- AWS credentials and AWS CLI availability for CodeArtifact/ECR/other cloud operations.
- Docker runtime availability for containerized pipeline execution.
- GitHub access tokens and API integration for clone/push/PR workflows.
- Optional webhook/notification endpoints for status reporting.

## Architectural Constraints
- This repository is a platform layer: interfaces exposed through `vars/*` and handler classes should remain stable to avoid widespread pipeline breakage.
- Branch constants and semantics (`main`, `release`, default fallback) are shared contracts and should be changed cautiously.
- Repo-to-image and node/image mappings are central coupling points; unsupported repo mappings can fail pipelines early.
- Container execution assumes credentials and Docker socket access are available on chosen nodes.
- Workspace temp-directory safety checks must remain conservative to avoid destructive filesystem actions in CI agents.

## Runtime / Deployment
- Used at pipeline runtime as a Jenkins shared library loaded by downstream Jenkinsfiles.
- Execution model commonly follows:
  - Jenkins node selection (`og-server` / other node labels),
  - containerized execution via configured image enums and ECR pulls,
  - handler-driven CI/CD steps in downstream repositories.
- Includes status-notification behavior that can report success/failure to Telegram and optional company orchestrator endpoints.

## Change Impact
- Changes here can affect multiple repositories simultaneously because many pipelines depend on the same shared handlers.
- High-risk areas:
  - Node/container execution behavior
  - Git authentication and release-merge helpers
  - AWS auth/CodeArtifact helpers
  - Notification/webhook flows used for operational visibility
- Before merge, validate:
  - Compatibility against representative downstream Jenkinsfiles across repo types
  - Backward compatibility of exported `vars/*` calls and handler method signatures
  - Credential-dependent paths (AWS/GitHub/webhook) under failure and success conditions
  - Safe behavior for temp workspace creation/deletion and container execution

## Failure Modes
- Missing/invalid AWS or GitHub credentials break clone/build/deploy stages across dependent pipelines.
- Unknown repository-image mapping or node/image enum mismatch causes pipeline startup failures.
- Docker/ECR auth failures block containerized execution.
- Shared helper regressions (e.g., merge, PR, temp-dir cleanup) can cascade into many repo CI/CD failures.
- Notification/webhook failures can obscure pipeline health even when build logic completes.
- Incorrect branch handling can produce unintended release merges or skipped deployment behavior.

## Shared Utility Opportunities
- Stronger compatibility test harness simulating downstream Jenkinsfile call patterns across repo archetypes.
- Centralized schema/validation for repo-to-image mapping to reduce runtime “unknown repository” failures.
- Unified error taxonomy and telemetry hooks across handlers for faster incident triage.
- Additional idempotent safety wrappers around mutating Git/AWS operations.

## Modification Guidance
- Treat all exported handler methods as platform contracts; prefer additive changes over breaking rewrites.
- Keep downstream compatibility first: update shared helpers only with explicit cross-repo impact analysis.
- Isolate product-specific behavior in downstream pipelines, not in shared library core.
- Validate credentialed operations and failure-path messaging to preserve diagnosability.
- When changing mappings/constants, verify all known consuming repos still resolve node/image/release behavior correctly.
- Avoid introducing hidden side effects in utility methods that are widely reused.

## Knowledge Gaps
- Full inventory of all repositories currently importing this shared library is not fully documented here.
- Canonical release/deployment policy ownership split between this shared library and downstream Jenkinsfiles is partially inferred.
- End-to-end observability/reporting architecture beyond Telegram/webhook calls is not explicitly mapped in this repository.
- Formal versioning/deprecation policy for shared library API changes is not documented in-repo.

## Notes
- This repository acts as CI/CD infrastructure glue; small changes can have organization-wide impact.
- Static repo-to-image mappings currently embed assumptions about known repositories, creating a central coupling hotspot.

## Related Pages
- `wiki/infra/dev-tools/og-cli.md`
- `wiki/infra/ci-cd/og-docker-images.md`
- `wiki/repos/loan_calculation_service.md`
- `wiki/repos/delonayde_assets.md`
- `wiki/repos/loan-calculator-app.md`
