# System Infrastructure Overview

## CI/CD
- Jenkins is primary orchestrator
- builds triggered per repo
- uses og-cli inside docker environments

## Build System
- og-cli standardizes all builds
- supports multiple project types

## Container System
- og-docker-images defines build/runtime environments

## Deployment
- helm-charts define k8s deployments
- currently manually maintained

## Artifact Management
- AWS CodeArtifact (packages)
- AWS ECR (docker images)
- AWS Secret Manager (Central secrert management for all repos)

## Observability
- Grafana used for metrics/logs/traces