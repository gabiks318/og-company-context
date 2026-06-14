# og-cli

## Role
- Provides the company-standard command-line control plane for project lifecycle operations across multiple repository types (init, build, test, deploy, CI/CD, docs, secrets, and local execution utilities).
- Encodes common engineering workflows so repositories can run consistent automation through `og` commands instead of bespoke per-repo scripts.

## Responsibilities
- Detect and load project context from `og.yml`, then route commands to the correct project-type executor.
- Scaffold new repositories from company templates (Python package, Python Lambda API, Flutter, FastAPI, Streamlit, and custom/unique flows).
- Standardize CI/CD entrypoints (`og ci`, `og cd`) used by Jenkins pipelines.
- Orchestrate build/test/deploy behavior per project type, including environment-aware execution.
- Manage scoped secrets/environment-variable mapping between `og.yml` and AWS Secrets Manager.
- Support infra-related bootstrap flows (repository initialization, GitHub/Jenkins bootstrap hooks, docs deployment, and release helpers).

## Not Responsible For
- Owning business-domain logic of product repositories.
- Serving runtime customer traffic directly (it is an engineering tool, not an application service).
- Defining product-specific API contracts or UI behavior.
- Replacing project-specific implementation code where custom behavior is intentionally required.
- Acting as the source-of-truth for Jenkins library internals or cloud-account policy definitions.

## Consumers
- Internal repositories that execute lifecycle commands via `og` (Python packages, Lambda APIs, Flutter apps, FastAPI apps, Streamlit apps, and unique project types).
- Jenkins pipelines invoking `og ci`, `og cd`, and `og docs deploy`.
- Engineers and AI agents creating new repos and operating delivery workflows.
- Documentation and infrastructure workflows dependent on OG-standard project metadata (`og.yml`).

## Dependencies
- `og-tech-common` for shared infrastructure/service utilities.
- AWS services and credentials path (CodeArtifact, Secrets Manager, S3, and Terraform-driven cloud deployment workflows).
- Git/GitHub and Jenkins integration services used during bootstrap and delivery orchestration.
- Cookiecutter template system for project initialization.
- Standard project tools executed through subprocess orchestration (`uv`, `sam`, `terraform`, `flutter`, `docker`, etc.).

## Architectural Constraints
- `og.yml` is the canonical per-repo contract; command behavior depends on valid `project_type`, scopes, and mappings.
- Executor boundaries must remain project-type specific; avoid leaking one project type's workflow assumptions into another.
- Command exit behavior must remain CI-safe (explicit success/failure signaling for pipeline reliability).
- Scoped secret/env resolution (`common` + environment scopes) must stay deterministic and backward compatible.
- Reserved telemetry/env namespaces are protected and must not be overridden by repo-level mappings.

## Runtime / Deployment
- Packaged as Python CLI (`og` entrypoint), versioned with SCM-derived tags.
- Repo-level pipeline follows shared branch model:
  - non-`release`: `og ci`
  - `release`: `og cd` + docs deployment
  - `main`: merged to `release` in Jenkins flow
- Publishes/distributes through internal package infrastructure (CodeArtifact-backed index).
- Works as a local developer tool and as a non-interactive CI orchestration tool.

## Change Impact
- Changes to CLI command semantics can affect many downstream repositories simultaneously.
- High-impact areas:
  - `OgConfig` parsing/normalization behavior
  - executor selection and command dispatch
  - secrets/environment resolution
  - CI/CD/versioning/tagging flows
- Before merge, validate:
  - Cross-project-type CLI tests
  - backward compatibility for existing `og.yml` files
  - non-interactive CI behavior and exit codes
  - secret-mapping checks and deployment command safety paths

## Failure Modes
- Invalid/missing `og.yml` prevents command execution for target repos.
- Breaking config normalization can block multiple repos from running CI/CD.
- Secrets mapping drift between `og.yml` and AWS payloads causes deployment/runtime failures.
- Toolchain availability issues (`uv`, `sam`, `terraform`, `flutter`, `docker`) break executor workflows.
- Version/tagging or release-flow regressions can corrupt downstream release automation.
- Partial bootstrap failures (GitHub/Jenkins setup) leave newly initialized repos in inconsistent infra state.

## Shared Utility Opportunities
- Stronger shared contract tests for `og.yml` schemas across all project templates.
- Unified reusable release/rollback policy helpers across executors.
- Centralized cross-executor observability for command duration/failure taxonomy.
- Shared compatibility test matrix for template -> executor -> CI path evolution.

## Modification Guidance
- Treat this repository as a platform dependency; assume broad blast radius for behavior changes.
- Prefer extending existing executor abstractions rather than introducing parallel command paths.
- Preserve existing command interfaces and defaults unless a migration plan is explicit.
- Validate both local developer UX and non-interactive CI execution for any command change.
- When changing secret/env logic, test scope resolution and failure messaging across environments.
- Avoid embedding product-specific assumptions in core CLI abstractions.

## Knowledge Gaps
- Full inventory of all internal repositories and pipelines depending on `og-cli` is not fully enumerated here.
- Canonical ownership boundaries between `og-cli`, Jenkins shared libraries, and infra repos are only partially visible from this repository.
- Organization-wide rollout strategy for breaking CLI changes is not explicitly documented in one place.
- Complete list of unique-project executors and their production usage is not fully mapped from current docs.

## Notes
- `og-cli` acts as infrastructure glue across repo types; regressions can manifest as widespread CI/CD failures rather than isolated feature bugs.
- Branch-driven automation conventions (CI vs CD vs release merge behavior) are encoded in the tooling ecosystem and relied on by downstream repos.

## Related Pages
- `wiki/infra/ci-cd/og-jenkins.md`
- `wiki/infra/ci-cd/og-docker-images.md`
- `wiki/projects/loanwise.md`
- `wiki/repos/loan_calculation_service.md`
- `wiki/repos/delonayde_assets.md`
- `wiki/repos/loan-calculator-app.md`
