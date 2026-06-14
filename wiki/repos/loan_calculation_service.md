# loan_calculation_service

## Role
- Provides the Loanwise HTTP calculation API for loan payment schedules, exposed through API Gateway and implemented as an AWS Lambda service.
- Translates request parameters into validated computation calls and returns normalized payment schedules for supported loan models.

## Responsibilities
- Own the public calculation endpoints (`/loan-calculation`, `/version`) and their response contract.
- Validate and normalize incoming query parameters via HTTP schema rules.
- Orchestrate loan computation using the shared `dla` financial library.
- Handle request-level error mapping (`400` validation issues, `404` unsupported paths, `500` unhandled errors).
- Emit service observability signals (OpenTelemetry traces/metrics and structured logs).
- Package and deploy the Lambda/API artifact through the `og` CI/CD flow.

## Not Responsible For
- Implementing core loan math formulas (owned by the `dla` package).
- Frontend rendering or user-facing UI workflows.
- Long-term loan data persistence or historical ledger storage.
- Authentication and authorization policy definition.
- Cross-service orchestration/business workflow sequencing outside this API boundary.

## Consumers
- Direct API consumers of `/loan-calculation` and `/version` (specific calling systems are not fully mapped from repository evidence alone).
- Internal CI/CD and deployment infrastructure that builds and deploys this service.

## Dependencies
- `dla` shared package for loan calculation domain logic.
- `og-tech-common` for schema validation, logging, and observability helpers.
- AWS API Gateway + AWS Lambda runtime defined through SAM/Terraform assets.
- Internal CodeArtifact package index and `og` pipeline tooling used in CI/CD.

## Architectural Constraints
- Keep this repository focused on API-layer orchestration and contracts; do not move business loan math here.
- Preserve backward compatibility of query parameters and response shape unless coordinated with API consumers.
- Maintain consistency between schema validation rules and supported runtime loan modes/repayment types.
- Preserve CORS behavior and endpoint semantics unless explicitly changing API contract.
- Keep observability attributes stable enough for dashboards/alerts that depend on metric names and dimensions.

## Runtime / Deployment
- `og.yml` identifies this repo as `python_lambda_api` with an alpha API endpoint.
- Jenkins executes `og ci` on non-`release` branches and `og cd` on `release`.
- Service is deployed as Lambda + API Gateway (SAM template present; Terraform assets also present), with Python 3.12 runtime.

## Change Impact
- Contract changes to query params, enum values, validation, or response fields can break all API clients immediately.
- Loan behavior changes can alter repayment outputs consumed by downstream product flows, pricing UX, and integrations.
- Observability name/dimension changes can silently break operational dashboards and alerting.
- Before merge, validate:
  - Unit tests for handler behavior and error status mapping.
  - Integration tests against a deployed API endpoint.
  - Representative loan scenarios across constant/dynamic/indexed variants.
  - Compatibility with the current `dla` package behavior.

## Failure Modes
- Schema mismatch or missing required params returns `400` and blocks calculations.
- Unsupported endpoint paths return `404`.
- Runtime exceptions in orchestration or dependency calls return `500`.
- Contract drift between `http_schema.yml` and handler logic can cause valid-looking client requests to fail.
- Deployment/auth failures in CI (CodeArtifact/AWS credentials) can block release delivery.
- Invalid or mismatched list inputs (`interest_by_month`, `index_list`) propagate as calculation-time failures.

## Shared Utility Opportunities
- Extract reusable API error-envelope/validation-response patterns for other Lambda APIs.
- Shared query-list normalization utility for comma-separated numeric arrays.
- Shared API contract testing harness for endpoint compatibility checks across environments.

## Modification Guidance
- First determine whether the requested change belongs in this API layer or in the shared `dla` math package.
- Prefer extending existing validation and orchestration paths over adding parallel endpoint logic.
- Treat any parameter/response adjustment as a consumer-facing contract change; coordinate and version if needed.
- Keep compatibility with existing repayment/loan type naming conventions unless a migration plan exists.
- Update unit and integration coverage whenever endpoint behavior, validation, or orchestration changes.
- Avoid embedding persistence, infra policy, or UI concerns in this service.

## Knowledge Gaps
- Full inventory of production consumers calling this API is not discoverable from this repository alone.
- Ownership boundary between SAM and Terraform deployment paths is present but the canonical source of truth is not explicitly documented.
- Environment-specific rollout and promotion policy beyond Jenkins branch logic is not fully specified in-repo.
- Current alerting/dashboard dependencies for metric names are not documented in this codebase.

## Notes
- The API requires some parameters that may be ignored for specific loan modes (for example, `annual_interest_rate` appears schema-required even when dynamic lists drive effective rates), which is a contract quirk consumers may already rely on.
- Repayment enum key uses `EQUAL_PRINCIPLE` naming for compatibility; changing it would be breaking for callers.

## Related Pages
- `wiki/projects/loanwise.md`
- `wiki/repos/delonayde_assets.md`
- `wiki/repos/loan_calculation_service.md`
