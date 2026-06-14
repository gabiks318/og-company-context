# helm-charts

## Role
- Centralizes Kubernetes deployment configuration and operational manifests for infrastructure services managed via Helm and `kubectl`.
- Acts as the infrastructure configuration source for selected platform components (notably Jenkins, n8n, and monitoring stack) including ingress, persistence, network controls, and backup/restore operations.

## Responsibilities
- Maintain environment values and operational manifests used to deploy/upgrade infrastructure services.
- Provide service-specific deployment guidance and runbooks (Jenkins, n8n, monitoring).
- Define backup and restore workflows for stateful infrastructure components (Jenkins and n8n) with S3-based artifact handling.
- Maintain security-relevant operational controls in manifests (namespace scoping, network policy artifacts, TLS/issuer references, secret conventions).
- Host/track Helm chart source integration for n8n chart operations and release/lint workflows in the included chart subtree.

## Not Responsible For
- Owning application business logic for product repositories.
- Replacing CI/CD pipeline orchestration tools (`og-cli`, `og-jenkins`) for build/test/release flows.
- Managing cloud infrastructure provisioning state directly (this repo defines deployment manifests/values, not full IaC ownership boundaries).
- Storing production credentials/secrets in version control.
- Serving as the single source of truth for every cluster-wide policy outside the manifests explicitly maintained here.

## Consumers
- Platform/infrastructure engineers operating Kubernetes-based internal services.
- Operational workflows for Jenkins and n8n continuity (backup, restore, migration procedures).
- Teams relying on monitoring stack availability and Grafana/Prometheus operational visibility.
- Downstream internal services that depend on stable Jenkins/n8n/monitoring runtime availability.

## Dependencies
- Kubernetes cluster runtime plus `kubectl` and Helm tooling.
- Ingress + TLS ecosystem (nginx ingress and cert-manager issuer references in values/manifests).
- AWS S3 backup bucket (`og-tech-backups`) and IAM credentials for backup/restore jobs.
- External chart ecosystem and registries (notably n8n chart release/lint pipeline dependencies in the chart subtree).
- Stateful storage classes and persistent volume provisioning in target clusters.

## Architectural Constraints
- This repository is operational infrastructure config; changes can directly impact platform availability and data durability.
- Stateful service backup/restore conventions (S3 key paths and namespace-specific jobs) must remain consistent to preserve recovery guarantees.
- Secret handling is out-of-band by design; manifests assume secrets are created/applied separately and should not embed raw credentials.
- Ingress hostnames/TLS issuer assumptions are contract points with DNS, cert-manager, and external traffic controls.
- n8n chart customizations and local values must stay compatible with upstream chart structure/version semantics.

## Runtime / Deployment
- Deployment model is operator-driven using Helm upgrades/install commands and targeted `kubectl apply` for supplemental manifests.
- Service folders represent operational domains:
  - `jenkins/` (controller values, namespace/PVC/network and backup/restore support)
  - `n8n/` (chart upgrade values/secrets references, network policy, backup/restore support)
  - `monitoring/` (kube-prometheus-stack values and exposure settings)
- n8n chart subtree includes its own GitHub Actions workflows for lint/testing and chart release automation, indicating partially independent upstream chart lifecycle management.

## Change Impact
- Values/manifests changes can alter service availability, ingress reachability, data persistence, and backup recoverability.
- High-impact areas:
  - storage/persistence configuration
  - ingress/TLS/network policy settings
  - backup/restore job behavior and S3 path conventions
  - chart version and chart-value compatibility (especially n8n)
- Before merge/apply, validate:
  - `helm lint` / dry-run compatibility for changed charts/values
  - backup job execution and restore procedure correctness in target namespace
  - ingress + certificate issuance behavior
  - no secret material is committed in plain text

## Failure Modes
- Misconfigured values can cause failed Helm upgrades, pod crash loops, or service outage.
- Backup credential/permission issues can silently break periodic backups, increasing disaster-recovery risk.
- Restore procedure errors can cause prolonged downtime or partial data recovery.
- Ingress/TLS misconfiguration can cut off external access to Jenkins/n8n/Grafana.
- Chart/version drift between local values and upstream chart expectations can break deployments.
- Network policy mistakes can isolate services from required dependencies.

## Shared Utility Opportunities
- Standardized reusable backup/restore job templates and verification checks across stateful services.
- Shared validation scripts for Helm values policy checks (ingress/TLS/storage/secret references).
- Unified documentation format for service runbooks (deploy, verify, backup, restore, rollback).
- Central mapping of cluster/environment overlays to reduce manual per-service drift.

## Modification Guidance
- Treat changes as production-impacting infra operations; prefer small, verifiable increments.
- Validate chart/value compatibility before apply, especially when chart versions or major value sections change.
- Keep secret material external; commit only references and operational instructions.
- Test backup and restore paths whenever touching stateful storage or backup manifests.
- Coordinate DNS/ingress/cert changes with platform owners before rollout.
- Avoid mixing unrelated service changes in a single update to simplify rollback and incident isolation.

## Knowledge Gaps
- Canonical CI/CD ownership for this repository is not explicit (no top-level `og.yml`/Jenkinsfile observed).
- Full environment topology (dev/staging/prod cluster separation and promotion strategy) is not documented in one place.
- Exact ownership split between local infra manifests and upstream n8n chart release lifecycle is partially inferred.
- Complete list of services expected to be managed from this repo (beyond Jenkins/n8n/monitoring) is unclear.
- Security/compliance policy for handling example secrets in values files is not fully documented in-repo.

## Notes
- This repository combines service operational manifests with a vendored/upstream-style chart subtree, creating mixed governance modes that need careful change discipline.
- Backup policy explicitly scopes S3 permissions by service prefixes (`jenkins/*`, `n8n/*`), making path consistency operationally critical.

## Related Pages
- `wiki/infra/system-overview.md`
- `wiki/infra/ci-cd/og-jenkins.md`
- `wiki/infra/dev-tools/og-cli.md`
- `wiki/infra/ci-cd/og-docker-images.md`
