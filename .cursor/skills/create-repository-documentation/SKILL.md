---
name: create-repository-documentation
description: Creates or updates repository documentation pages in the company knowledge base, focusing on high-value architectural context and boundaries rather than implementation details. Use when documenting a repository, populating repo wiki pages, or when the user asks to capture repo role, responsibilities, dependencies, constraints, impacts, and failure modes.
disable-model-invocation: true
---

# Skill: Create Repository Documentation

Your task is to create or update the repository documentation page in the company knowledge base.

The goal of this document is NOT to describe the codebase in detail. The code already exists and can be inspected directly.

The goal is to capture the information that an engineer or AI agent cannot easily infer from reading the repository alone.

---

## Core Principles

### Focus on reasoning, not implementation

Document:

* Why the repository exists
* What responsibilities it owns
* What responsibilities it explicitly does NOT own
* Which systems depend on it
* Which systems it depends on
* Architectural constraints
* Change impact and downstream effects
* Important behavioral contracts
* Common failure modes
* Modification guidance (how to safely change this repo)

Do NOT document:

* File structure
* Class hierarchy
* Function lists
* Dependency versions (unless they are architectural constraints)
* Installation instructions
* Information that can be easily discovered from code

---

## Repository Documentation Template

### Role

Describe the business and architectural purpose of the repository.

Questions to answer:

* Why does this repository exist?
* What problem does it solve?
* What capability does it provide to the company?

Keep this section concise.

---

### Responsibilities

List the responsibilities that belong to this repository.

Examples:

* Provides loan calculation logic
* Manages Reddit comment generation
* Stores shared Flutter utilities

Use bullet points.

---

### Not Responsible For

List responsibilities intentionally handled elsewhere.

Examples:

* API layer
* Database persistence
* Authentication
* Infrastructure provisioning

This section is important because it helps prevent architectural boundary violations.

---

### Consumers

List systems, repositories, services, dashboards, applications, or pipelines that depend on this repository.

Examples:

* loan-calculation-service
* loan-calculator-app
* reddit-comment-generator

Focus on meaningful consumers.

---

### Dependencies

List important internal dependencies.

Include:

* Shared repositories
* Internal services
* Infrastructure systems
* External APIs

Do NOT include ordinary package dependencies unless they are architecturally significant.

---

### Architectural Constraints

Document rules that must be respected.

Examples:

* Pure business logic only
* No AWS interactions
* No UI code
* No persistence layer
* Shared logic should remain reusable

These constraints are critical for AI agents.

---

### Change Impact

Describe what is likely affected when this repository changes.

Examples:

* API consumers
* Frontend applications
* Shared utility packages
* Data contracts

Be explicit and actionable:

* What should be validated before merging?
* What systems could break?
* What requires coordinated rollout?

If changes affect business-critical behavior, explicitly mention validation requirements.

---

### Failure Modes

Document important failure scenarios.

Examples:

* Invalid configuration
* Contract mismatch
* Missing secrets
* Data format incompatibilities
* Business rule violations

Focus on operational and architectural failures rather than implementation details.

---

### Shared Utility Opportunities

Document functionality that may belong in a shared repository.

Examples:

* Common retry logic
* Shared API clients
* Validation frameworks
* UI components

This section helps identify opportunities for extraction into shared packages.

---

### Modification Guidance (NEW — IMPORTANT)

This section guides engineers and AI agents when making changes.

Before modifying this repository:

1. Identify whether the change belongs in this repository or a shared dependency.
2. Prefer extending existing abstractions over introducing parallel logic.
3. Ensure backward compatibility unless a coordinated migration is planned.
4. Update contract-level tests when business behavior changes.
5. Consider downstream consumers before merging.
6. Avoid introducing infra, API, or persistence logic unless explicitly part of this repo’s responsibility.

This section is critical for preventing architectural drift.

---

### Knowledge Gaps (NEW)

Explicitly document unknowns or incomplete mappings.

Examples:

* List of all downstream consumers may be incomplete
* External integrations may not be fully mapped
* Some contract dependencies may be implicit in downstream services

This helps AI agents reason under uncertainty.

---

### Notes

Document only surprising or non-obvious information.

Good examples:

* Historical constraints
* Backward compatibility requirements
* Intentional deviations from expected behavior
* Known quirks relied upon by consumers

Bad examples:

* Obvious implementation details
* File locations
* Class names

---

### Related Pages

Link to:

* Project page
* Shared repositories
* Infrastructure systems
* Other closely related repositories

---

## Information Gathering

When creating the document:

1. Read the repository structure.
2. Inspect README files.
3. Inspect CI/CD configuration.
4. Inspect og.yml.
5. Inspect major entrypoints.
6. Identify upstream and downstream dependencies.
7. Infer architectural boundaries.
8. Capture only high-value information.

---

## Writing Style

* Concise and factual.
* Use bullet points where possible.
* Avoid implementation details.
* Avoid duplicating information already documented in infrastructure pages.
* Optimize for future engineering work and AI agent reasoning.

Always ask:

"Will this help an engineer or AI agent make a better change to this repository?"

If the answer is no, omit it.

## Clarification Behavior (IMPORTANT)

If critical architectural information is missing or cannot be reliably inferred from the repository, the skill should explicitly ask the user for clarification instead of guessing.

This includes cases such as:

- Unknown or unclear downstream consumers
- Uncertain repository ownership boundaries
- Ambiguous responsibility separation between services
- Missing or incomplete CI/CD or deployment understanding
- Unclear shared utility extraction decisions
- Any situation where incorrect assumptions could lead to misleading documentation

### How to behave

When information is missing:

1. Explicitly state what is unknown.
2. Explain why it matters for correct documentation.
3. Ask targeted questions to fill the gap.
4. Do NOT proceed with assumptions in these areas.

### Example behavior

Instead of guessing:

- "This repository is used by X service"

Ask:

- "Are there any additional consumers of this repository besides X?"
- "Is this logic also used by any offline jobs or batch processes?"

### Guiding principle

It is better to produce incomplete but accurate documentation than complete but incorrect assumptions.