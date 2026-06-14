# og-docker-images

## Role
- Serves as the centralized source repository for company CI/runtime Docker images used across Jenkins pipelines and internal automation.
- Standardizes build environments (Python, AWS/GCP toolchains, Flutter, Dart, and specialized builder images) so downstream repos execute with consistent toolchains.

## Responsibilities
- Define and maintain Dockerfiles for approved OG image variants under `images/*`.
- Build and push image artifacts to the shared ECR registry namespace (`og-*` image naming pattern).
- Detect changed image definitions and only rebuild/publish impacted images in non-cron runs.
- Perform scheduled full rebuild/push cycles (cron-triggered main-branch pipeline) to refresh base layers and security posture.
- Provide image-management helper scripts via `og execute` (`list`, `changelist`, `build`, `push`, `create`, `create_repo`, `run_image_sh`).
- Keep image naming conventions aligned with downstream resolver/mapping expectations in CI infrastructure.

## Not Responsible For
- Orchestrating downstream application CI/CD logic itself.
- Owning product or service business logic.
- Defining repository-specific Jenkins pipelines outside this image supply repository.
- Managing app/runtime configuration inside consumer repos.
- Replacing downstream repo-level security or functional testing.

## Consumers
- `og-jenkins` shared library and Jenkins pipelines that run in OG image containers.
- Repositories using OG-standard CI commands (`og ci`, `og cd`) that depend on specific image names/tooling.
- Internal engineering workflows requiring reproducible containerized toolchains for build/test/deploy tasks.

## Dependencies
- AWS ECR and AWS credentials for login/push operations.
- Jenkins + `og-jenkins` pipeline orchestration for automated rebuild/publish flows.
- Docker/Buildx tooling for image builds and multi-platform build targeting flags.
- `og-cli` execution flow (`og execute ...`) for script invocation in CI.
- External upstream base images and package mirrors (Debian/Python/Node/Flutter ecosystem artifacts).

## Architectural Constraints
- Image tags and names (`og-<image-name>`) are shared contracts with downstream CI consumers and must remain stable.
- Jenkins pipeline intentionally gates publish flow to `main`; non-main branches should not publish shared official images.
- ECR registry/account and regional assumptions are embedded in scripts and must stay aligned with infra ownership.
- Build scripts pass credentials/tokens as build args when present; changes must preserve secure handling and avoid leakage.
- Dockerfile changes can affect many repositories at once due to shared-image reuse.

## Runtime / Deployment
- Repo is modeled as `project_type: unique` and uses a custom Jenkins pipeline.
- Jenkins pipeline behavior:
  - runs in `og-docker-builder` container
  - executes only on `main` branch
  - determines image list by cron/full (`og execute list`) vs change-based (`og execute changelist`) mode
  - pushes selected images to ECR (`og execute push ...`)
- Scheduled trigger exists (weekly cron) for periodic full refresh rebuilds.

## Change Impact
- Any Dockerfile update can alter CI behavior/performance for many downstream repositories.
- High-impact areas:
  - base image upgrades (Python/Node/OS layers)
  - authentication/tooling setup (AWS CLI, Terraform, Docker CLI/Buildx, Flutter SDK)
  - image naming/tagging behavior in scripts
- Before merge, validate:
  - changed image builds locally and in CI
  - downstream pipelines still run with required tool versions
  - ECR push path remains valid
  - `changelist` logic correctly scopes which images publish

## Failure Modes
- ECR login/push failures block image publication and can stall downstream CI pipelines awaiting updated bases.
- Mis-tagging or naming drift breaks consumer pipeline image resolution.
- Build arg credential/token misconfiguration causes partial builds or unauthorized package fetch failures.
- Upstream base image/package repository changes can introduce reproducibility or compatibility regressions.
- Incomplete changed-image detection can skip needed publishes or push stale image sets.
- Scheduled full rebuild failures can leave security/patch cadence behind expected baseline.

## Shared Utility Opportunities
- Add explicit image metadata/manifest mapping to track consumers and required tool versions per image.
- Add pre-push validation checks (lint/vulnerability/contract checks) for image quality gates.
- Centralize ECR registry/account/region config into a single declarative config to reduce hardcoded drift.
- Add compatibility smoke tests against representative downstream repo pipelines before publishing.

## Modification Guidance
- Treat this repository as CI supply-chain infrastructure; assume broad downstream blast radius for changes.
- Prefer additive image evolution over breaking changes to existing image contracts.
- Keep image names, tags, and script interfaces stable unless a coordinated migration is planned.
- Validate both selective (`changelist`) and full (`list`) publish paths when updating scripts/pipeline.
- Coordinate major toolchain upgrades with known consumer repos before merge.
- Avoid embedding repo-specific assumptions into shared builder images unless intentionally scoped.

## Knowledge Gaps
- Complete inventory of all repositories consuming each image variant is not explicitly documented here.
- Formal deprecation policy for old image variants/tags is not visible in repository docs.
- Security scanning/signing/SBOM policy for published images is not explicitly defined in this repo.
- Ownership split between `og-docker-images`, `og-jenkins`, and downstream repo maintainers for image compatibility is only partially inferred.

## Notes
- This repo functions as a shared CI substrate; regressions often appear first as downstream pipeline failures rather than local repo failures.
- Cron-driven full rebuild behavior suggests an implicit patch-refresh strategy even when image definitions do not change.

## Related Pages
- `wiki/infra/ci-cd/og-jenkins.md`
- `wiki/infra/dev-tools/og-cli.md`
- `wiki/infra/ci-cd/og-docker-images.md`
- `wiki/repos/loan_calculation_service.md`
- `wiki/repos/delonayde_assets.md`
- `wiki/repos/loan-calculator-app.md`
