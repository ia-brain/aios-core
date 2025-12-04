# STORY 3.12: Documentation Sprint 3

**ID:** 3.12 | **Epic:** [EPIC-S3](../../../epics/epic-s3-quality-templates.md)
**Sprint:** 3 | **Points:** 5 | **Priority:** 🟡 Medium | **Created:** 2025-01-19
**Updated:** 2025-12-03
**Status:** 📋 Draft

**Reference:** [EPIC-S3 Documentation Requirements](../../../epics/epic-s3-quality-templates.md#documentation)

**Predecessor:** All Sprint 3 stories (3.1-3.11)

---

## User Story

**Como** developer ou tech lead,
**Quero** documentação completa do Sprint 3,
**Para que** possa entender e usar Quality Gates, Template Engine e Dashboard.

---

## Acceptance Criteria

### Quality Gates Guide
- [ ] AC3.12.1: Documenta arquitetura de 3 layers
- [ ] AC3.12.2: Inclui diagrama visual das 3 layers
- [ ] AC3.12.3: Documenta configuração de cada layer
- [ ] AC3.12.4: Inclui troubleshooting section

### Template Engine Guide
- [ ] AC3.12.5: Documenta API do TemplateEngine
- [ ] AC3.12.6: Inclui exemplos de uso para cada template type
- [ ] AC3.12.7: Documenta como criar novos templates
- [ ] AC3.12.8: Inclui schema reference

### CodeRabbit Setup Guide
- [ ] AC3.12.9: Documenta instalação do CLI
- [ ] AC3.12.10: Documenta integração com GitHub App
- [ ] AC3.12.11: Inclui configuração de self-healing
- [ ] AC3.12.12: Documenta WSL setup (Windows)

### Dashboard User Guide
- [ ] AC3.12.13: Documenta acesso ao dashboard
- [ ] AC3.12.14: Explica cada métrica exibida
- [ ] AC3.12.15: Documenta interpretação de trends

### Main Docs Update
- [ ] AC3.12.16: README atualizado com links para novos guides
- [ ] AC3.12.17: Architecture docs atualizados

---

## Scope

### Deliverables

| Document | Location | Dependencies |
|----------|----------|--------------|
| Quality Gates Guide | `docs/guides/quality-gates-3-layers.md` | Stories 3.1-3.5 |
| Template Engine Guide | `docs/guides/template-engine-v2.md` | Story 3.6 |
| CodeRabbit Setup | `docs/guides/coderabbit/` (existing, update) | Stories 3.2-3.4 |
| Dashboard Guide | `docs/guides/quality-dashboard.md` | Story 3.11 |

---

## Tasks

### Quality Gates Guide (4h)
- [ ] 3.12.1: Write Quality Gates guide
  - [ ] 3.12.1.1: Architecture overview with diagram
  - [ ] 3.12.1.2: Layer 1 (Pre-Commit) configuration
  - [ ] 3.12.1.3: Layer 2 (PR Review) configuration
  - [ ] 3.12.1.4: Layer 3 (Human Review) workflow
  - [ ] 3.12.1.5: Troubleshooting section

### Template Engine Guide (3h)
- [ ] 3.12.2: Write Template Engine guide
  - [ ] 3.12.2.1: API reference
  - [ ] 3.12.2.2: Usage examples per template type
  - [ ] 3.12.2.3: Creating custom templates
  - [ ] 3.12.2.4: Schema validation reference

### CodeRabbit Setup (3h)
- [ ] 3.12.3: Update CodeRabbit setup guide
  - [ ] 3.12.3.1: CLI installation (pip, WSL)
  - [ ] 3.12.3.2: GitHub App configuration
  - [ ] 3.12.3.3: Self-healing configuration
  - [ ] 3.12.3.4: Troubleshooting common issues

### Dashboard Guide (2h)
- [ ] 3.12.4: Write Dashboard guide
  - [ ] 3.12.4.1: Accessing the dashboard
  - [ ] 3.12.4.2: Understanding metrics
  - [ ] 3.12.4.3: Reading trends

### Main Docs Update (2h)
- [ ] 3.12.5: Update main docs
  - [ ] 3.12.5.1: Update README with guide links
  - [ ] 3.12.5.2: Update docs/architecture with QG reference
  - [ ] 3.12.5.3: Cross-link between guides

**Total Estimated:** 14h (~2 days)

---

## Dev Notes

### Document Templates

#### Quality Gates Guide Outline
```markdown
# Quality Gates - 3 Layer Architecture

## Overview
Brief description of the 3-layer quality gate system.

## Architecture Diagram
[Mermaid diagram showing Layer 1 → Layer 2 → Layer 3]

## Layer 1: Pre-Commit (Local)
### Purpose
### Tools
### Configuration
### Bypassing (emergencies)

## Layer 2: PR Review (Automated)
### Purpose
### CodeRabbit Integration
### Quinn (QA Agent)
### Self-Healing

## Layer 3: Human Review
### Purpose
### Review Workflow
### Approval Process

## Metrics & Dashboard
Link to dashboard guide.

## Troubleshooting
Common issues and solutions.
```

#### Template Engine Guide Outline
```markdown
# Template Engine v2.0

## Overview
What is the Template Engine and why use it.

## Quick Start
`aios generate prd` example.

## API Reference
### TemplateEngine class
### Methods
- loadTemplate()
- elicitVariables()
- render()
- validate()

## Supported Templates
- PRD
- ADR
- PMDR
- DBDR
- Story
- Epic
- Task

## Creating Custom Templates
Step-by-step guide.

## Schema Reference
JSON Schema documentation.
```

### Testing

| Test ID | Name | Priority |
|---------|------|----------|
| DOC-01 | Quality Gates guide has all sections | P0 |
| DOC-02 | Template Engine guide has API ref | P0 |
| DOC-03 | CodeRabbit guide covers CLI + App | P0 |
| DOC-04 | Dashboard guide explains metrics | P0 |
| DOC-05 | All internal links work | P0 |
| DOC-06 | Diagrams render correctly | P1 |

---

## 🤖 CodeRabbit Integration

### Story Type Analysis

**Primary Type:** Documentation
**Secondary Type(s):** N/A
**Complexity:** Low (documentation only)

### Specialized Agent Assignment

**Primary Agents:**
- @dev: Documentation writing

**Supporting Agents:**
- @qa: Link validation, completeness check

### Quality Gate Tasks

- [ ] Pre-Commit (@dev): Markdown lint check
- [ ] Pre-PR (@github-devops): Link validation

### Self-Healing Configuration

**Expected Self-Healing:**
- Primary Agent: @dev (light mode)
- Max Iterations: 2
- Timeout: 10 minutes
- Severity Filter: CRITICAL only

**Predicted Behavior:**
- CRITICAL issues: Auto-fix (broken links)
- HIGH issues: Document only

### Focus Areas

**Primary Focus:**
- Documentation completeness
- Internal link validity
- Markdown formatting

**Secondary Focus:**
- Diagram rendering
- Cross-guide consistency

---

## Dependencies

**Depends on:**
- Story 3.1 (Pre-Commit Hooks) ✅
- Story 3.3-3.4 (PR Automation) ⏳
- Story 3.5 (Human Review) ⏳
- Story 3.6 (Template Engine Core) ⏳
- Story 3.7-3.10 (Templates) ⏳
- Story 3.11 (Dashboard) ⏳

**Blocks:**
- Sprint 3 sign-off

---

## Definition of Done

- [ ] All acceptance criteria met
- [ ] All 4 guides written
- [ ] DOC-01 to DOC-06 checks pass
- [ ] Internal links validated
- [ ] README updated
- [ ] QA Review passed
- [ ] PR created and approved

---

## Dev Agent Record

_To be populated during implementation_

---

## Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-01-19 | 1.0 | Story created (in bundled file) | River |
| 2025-12-03 | 2.0 | Separated into individual story file with outlines | Pax (@po) |

---

## QA Results

_To be populated after implementation_

---

**Created by:** River 🌊
**Validated by:** Pax 🎯 (PO)
