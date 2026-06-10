# Knowledge Base Schema (Operational Rules)

This file defines the behavioral rules for all agents (Cursor cloud agents, automation scripts, MCP clients) interacting with the knowledge base.

It is a strict operational contract.

---

# 1. Core Principles

## 1.1 Git is the source of truth
- All knowledge must exist in this repository.
- No system knowledge may exist only in:
  - Cursor chat
  - agent memory
  - external tools

---

## 1.2 Wiki is architectural memory (not documentation)
- wiki reflects system structure and behavior
- it is NOT a user manual
- it is NOT API documentation

---

## 1.3 MCP is the only access layer
- all reads/writes of wiki content MUST go through MCP tools
- direct reasoning about file system without MCP is discouraged for agents

---

## 2. Knowledge Structure Rules

## 2.1 Layer separation

The system has 3 layers:

- index.md → navigation map (what exists)
- wiki/ → system knowledge (how it works)
- log.md → historical changes (what changed)

No mixing of responsibilities is allowed.

---

## 2.2 Single responsibility per page
Each wiki file must represent exactly one:

- repository
- shared system
- infra component
- project

---

## 2.3 No duplication of meaning
- each concept must have one primary definition location
- other references must link to it, not redefine it

---

## 2.4 index.md constraints
- must NOT contain deep explanations
- must only contain:
  - system names
  - grouping
  - high-level structure
- index.md is a routing layer only

---

# 3. Reading Rules (Agent Behavior)

## 3.1 Mandatory context order

When performing any task involving system understanding:

1. index.md (system orientation)
2. relevant wiki page
3. related repos pages
4. infra/shared pages if needed

Skipping this order is forbidden for multi-repo reasoning.

---

## 3.2 No guessing rule
- if system behavior is unclear → inspect wiki or code via MCP
- agents must NOT infer architecture from partial knowledge

---

## 3.3 Cross-repo awareness requirement
Any task affecting multiple repositories requires:

- reading all relevant repo pages
- identifying shared dependencies
- verifying infra constraints

---

# 4. Writing Rules (Wiki Updates)

## 4.1 When to create a page
Create a new wiki page ONLY if:

- the system has independent lifecycle
- multiple repos depend on it
- it represents an infra or shared abstraction
- it is part of a production pipeline

---

## 4.2 When to update a page
Update existing page ONLY when:

- responsibilities change
- dependencies change
- system behavior changes
- debugging knowledge is discovered

---

## 4.3 When NOT to update wiki
Do NOT update wiki for:

- local refactors
- temporary experiments
- internal code restructuring with no external effect

---

## 4.4 Append-only log rule
- log.md is append-only
- no modification of past entries
- all meaningful system changes must be logged

---

# 5. System Boundaries

## 5.1 Shared utilities boundary (CRITICAL)
Repositories under shared/ must:

- contain ONLY reusable logic
- be used by at least 2 systems to justify existence
- never contain business-specific logic

Violations must be corrected via refactoring.

---

## 5.2 Infra boundary rule
Infra systems:

- define execution and deployment behavior
- must NOT contain business logic
- changes affect all systems by default

---

## 5.3 Repo independence rule
- each repository must remain independently buildable
- cross-repo dependencies must be explicit in wiki

---

# 6. Dependency Rules

## 6.1 Explicit dependency requirement
Every repo page must list:

- upstream dependencies
- downstream consumers

Hidden dependencies are forbidden.

---

## 6.2 Change impact rule
If a change affects:

- shared/
- infra/
- multiple repos

→ it MUST be reflected in wiki and log.md

---

# 7. Failure Handling Rules

## 7.1 Uncertainty rule
If agent is uncertain:

- it must inspect code via MCP OR
- create a wiki update proposal

Guessing is forbidden.

---

## 7.2 Debugging rule
When debugging system behavior:

- consult repo page first
- then shared/infra pages
- then logs

---

# 8. Mental Model Enforcement

Agents must treat the system as:

- index.md → navigation layer
- wiki → architectural memory
- log.md → historical truth
- code → execution truth
- MCP → controlled access layer

---

# 9. Consistency Rules

## 9.1 No orphan pages
All wiki pages must be referenced from:

- index.md OR
- another wiki page

---

## 9.2 Backlink awareness
Agents must consider:

- who depends on this system
- what breaks if this system changes

---

## 9.3 System-wide consistency
If multiple pages describe the same concept:

- resolve into a single canonical page
- remove duplication

---

# End of Schema