---
name: create-project-documentation
description: Creates or updates project-level documentation pages by synthesizing cross-repository system behavior, interactions, runtime flow, dependencies, constraints, and failure modes. Use when documenting a multi-repository system or when the user asks for project/system documentation rather than repository documentation.
disable-model-invocation: true
---

# Skill: Create Project Documentation

Your task is to create or update a PROJECT documentation page in the company knowledge base.

A project represents a *system composed of multiple repositories and infra components*.

Unlike repository documentation, which focuses on local ownership and implementation boundaries, project documentation focuses on:
- system-level behavior
- interactions between repositories
- end-to-end workflows
- cross-repo dependencies
- runtime flow and integration points

The goal is to help an engineer or AI agent understand how the system works as a whole and how all repositories interact.

---

## Core Principle

A project document is NOT a summary of repositories.

It is a **system design description derived from multiple repositories**.

You must assume:
- You have access to all repository docs for this project
- Each repository already has its own canonical documentation
- You must NOT duplicate repository-level details
- You MUST synthesize cross-repository behavior

---

## Information Sources

When building this document, use:

- Repository documentation pages
- Infra documentation pages (CI/CD, og-cli, docker, k8s, etc.)
- Shared library docs
- Observed CI/CD flows (Jenkins, og pipelines)
- og.yml definitions across repos

---

## What to Extract

From multiple repos, identify:

- How data flows between services
- How repositories are composed into a system
- Which repos are runtime-critical vs auxiliary
- Deployment and CI/CD orchestration flow
- Shared infrastructure usage
- Cross-repo contracts and dependencies
- System-level failure modes
- Integration boundaries and coupling points

---

## What NOT to Include

- Internal class/function details
- File structures of repositories
- Package-level dependencies (unless system-critical)
- Implementation details inside a single repo
- Repetition of repo documentation content

---

## Project Documentation Template

### Project Name

Name of the system (e.g., Loanwise, Reddit Agent, etc.)

---

### System Overview

Describe the project as a whole system:

- What problem does this system solve?
- What is its end-to-end purpose?
- What capabilities does it provide?

Focus on system behavior, not code.

---

### System Components

List all repositories and infra components involved:

Group them logically:

#### Core Services
- primary backend services
- computation engines
- APIs

#### Supporting Services
- background workers
- data collectors
- schedulers

#### Frontend / Interfaces
- apps
- dashboards
- UIs

#### Shared Libraries
- reusable packages

#### Infrastructure
- CI/CD pipelines
- Docker images
- Kubernetes / Helm
- secrets management

---

### System Architecture Flow

Describe how the system works end-to-end.

Include:

- request flows
- data flows
- CI/CD flows if relevant
- async workflows
- inter-service communication patterns

Focus on *runtime behavior*.

---

### Repository Interactions

Explain how repositories interact:

- Which services depend on outputs of others
- Which repos trigger downstream processes
- Where contracts exist between repos
- Where shared libraries enforce consistency

---

### Deployment & Runtime Model

Describe:

- how the system is deployed (Jenkins, og-cli, etc.)
- how services are built and released
- how infrastructure is provisioned
- how environments (dev/staging/prod) behave

---

### Critical Dependencies

Highlight system-wide dependencies:

- AWS services
- external APIs
- shared infrastructure systems
- internal platforms (CodeArtifact, ECR, etc.)

Focus on what can break the whole system.

---

### System-Level Constraints

Describe global rules that apply across repos:

- architectural boundaries between services
- shared contracts that must remain stable
- versioning rules
- coupling restrictions

---

### End-to-End Workflows

Describe key workflows such as:

- user flows (if applicable)
- data processing pipelines
- CI/CD lifecycle
- background job execution

Each workflow should describe multiple repositories interacting.

---

### Failure Modes (System Level)

Describe how the entire system can fail:

- broken contracts between services
- CI/CD pipeline failures
- shared dependency failures (CodeArtifact, secrets, infra)
- cascading failures between services

Focus on cross-repo failure propagation.

---

### Shared Knowledge / Abstractions

Identify:

- duplicated logic across repos
- missing shared utilities
- potential consolidation opportunities
- cross-project patterns

---

### Knowledge Gaps

Explicitly state unknowns:

- missing repo relationships
- unclear runtime dependencies
- incomplete deployment topology
- partially inferred system flows

Never guess — mark uncertainty explicitly.

---

### Evolution & Change Strategy

Describe how the system is expected to evolve:

- how new repos are added
- how shared libraries are introduced
- how infra changes propagate
- how breaking changes are managed

---

### Related Projects

Link to other system-level projects.

---

## Writing Rules

- Think at system level, not repository level
- Prefer diagrams-in-text (flows, bullet chains)
- Avoid duplication of repo docs
- Be explicit about uncertainty
- Focus on interactions and dependencies
- Optimize for AI agent reasoning and cross-repo changes

Always ask:

"How does this system behave as a whole when all repositories are deployed together?"
