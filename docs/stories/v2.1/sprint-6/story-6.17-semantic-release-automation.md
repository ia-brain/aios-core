# Story 6.17: Semantic Release Automation

## Story Info

| Field | Value |
|-------|-------|
| **Story ID** | 6.17 |
| **Epic** | Infrastructure & DevOps |
| **Sprint** | 6 |
| **Priority** | High |
| **Estimated Points** | 5 |
| **Status** | Testing |
| **Created** | 2025-12-18 |
| **Author** | @devops (Gage) |

## Problem Statement

O projeto possui documentação (`docs/versioning-and-releases.md`) que descreve releases automáticos via semantic-release, mas a funcionalidade **NÃO está implementada**:

### Gap Identificado

| Aspecto | Documentado | Implementado |
|---------|-------------|--------------|
| Dependências semantic-release | ✅ | ✅ |
| Scripts npm (release, release:test) | ✅ | ✅ |
| Configuração `.releaserc` | ✅ | ❌ **FALTANDO** |
| Workflow GitHub Actions | ✅ | ❌ **FALTANDO** |
| Trigger on merge to main | ✅ | ❌ **FALTANDO** |

### Impacto Atual

- PRs merged para `main` **NÃO** atualizam a versão automaticamente
- Usuários que instalam via `npx aios-core` recebem versão desatualizada
- Releases manuais são esquecidos, causando gap entre código e distribuição
- Exemplo: Story 6.16 foi merged mas versão NPM ainda é 2.2.2 (anterior)

## User Story

**Como** mantenedor do AIOS,
**Quero** que merges para main disparem releases automáticos,
**Para que** usuários sempre recebam a versão mais recente via NPM.

## 🤖 CodeRabbit Integration

### Story Type Analysis
**Primary Type**: Deployment/CI-CD
**Secondary Type(s)**: Configuration, Documentation
**Complexity**: Medium

### Specialized Agent Assignment

**Primary Agents**:
- @devops (Gage): CI/CD workflow creation, GitHub Actions configuration
- @dev (Dex): Configuration file creation (.releaserc.json)

**Supporting Agents**:
- @qa (Quinn): E2E validation, NPM publish verification

### Quality Gate Tasks
- [ ] Pre-Commit (@dev): Validate `.releaserc.json` JSON syntax and plugin order
- [ ] Pre-PR (@devops): Verify workflow YAML syntax with `actionlint`
- [ ] Pre-Deployment (@devops): Confirm `NPM_TOKEN` secret is active and valid

### Self-Healing Configuration

**Expected Self-Healing**:
- Primary Agent: @devops (check mode)
- Max Iterations: N/A (report only for deployment stories)
- Timeout: N/A
- Severity Filter: report_only

**Predicted Behavior**:
- CRITICAL issues: Report and block merge
- HIGH issues: Report only, document in PR

### CodeRabbit Focus Areas

**Primary Focus**:
- CI/CD pipeline configuration best practices
- Secrets management (no hardcoded tokens, proper env usage)
- GitHub Actions workflow syntax and permissions
- semantic-release plugin configuration order

**Secondary Focus**:
- YAML syntax validation
- JSON configuration validity
- Conventional commit message format documentation

---

## Acceptance Criteria

### AC1: Configuração Semantic-Release
- [x] Criar arquivo `.releaserc.json` com configuração completa
- [x] Configurar plugins: changelog, npm, git, github
- [x] Definir branches de release (`main`)
- [x] Configurar mensagens de commit para release

### AC2: Workflow GitHub Actions
- [x] Criar `.github/workflows/semantic-release.yml`
- [x] Trigger: push para branch `main`
- [x] Executar após CI passar (lint, test, typecheck)
- [x] Usar secrets: `NPM_TOKEN`, `GITHUB_TOKEN`

### AC3: Conventional Commits → Releases
- [ ] `fix:` commit → patch release (2.2.2 → 2.2.3)
- [ ] `feat:` commit → minor release (2.2.2 → 2.3.0)
- [ ] `feat!:` ou `BREAKING CHANGE:` → major release (2.2.2 → 3.0.0)
- [ ] `chore:`, `docs:`, `style:`, `test:` → NO release

### AC4: Automações no Release
- [ ] Criar git tag (v2.3.0)
- [ ] Criar GitHub Release com notas automáticas
- [ ] Publicar no NPM com tag `latest`

> **Nota:** `package.json` e `CHANGELOG.md` no repositório NÃO são atualizados automaticamente devido à branch protection. A versão é atualizada apenas no NPM registry.

### AC5: Validação End-to-End
- [ ] Testar com PR de teste (fix: test commit)
- [ ] Verificar versão atualizada no NPM
- [ ] Verificar `npx aios-core --version` retorna nova versão
- [ ] Verificar instalador usa arquivos atualizados

## Technical Design

### 1. Arquivo `.releaserc.json`

> **Nota:** Plugins `@semantic-release/git` e `@semantic-release/changelog` foram removidos porque o branch `main` está protegido e não permite pushes diretos, mesmo do GitHub Actions. A versão em `package.json` é atualizada apenas no NPM registry, não no repositório.

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    ["@semantic-release/npm", {
      "npmPublish": true
    }],
    "@semantic-release/github"
  ]
}
```

### 2. Workflow `.github/workflows/semantic-release.yml`

```yaml
name: Semantic Release

on:
  push:
    branches:
      - main

permissions:
  contents: write
  issues: write
  pull-requests: write
  id-token: write

jobs:
  # Run CI first
  ci:
    uses: ./.github/workflows/ci.yml

  release:
    name: Release
    needs: ci
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          persist-credentials: false

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Semantic Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release
```

### 3. Fluxo Completo

```
Developer                    GitHub                      NPM
    │                           │                         │
    │  git push (feat: ...)     │                         │
    ├──────────────────────────►│                         │
    │                           │                         │
    │                    [CI Workflow]                    │
    │                    - lint ✓                         │
    │                    - test ✓                         │
    │                    - typecheck ✓                    │
    │                           │                         │
    │               [Semantic Release Workflow]           │
    │                    - analyze commits                │
    │                    - determine version bump         │
    │                    - update package.json            │
    │                    - generate CHANGELOG             │
    │                    - create git tag                 │
    │                    - create GitHub Release          │
    │                    - npm publish ──────────────────►│
    │                           │                         │
    │                           │                   [Registry Updated]
    │                           │                         │
User: npx aios-core ◄───────────────────────────────────────┘
         (gets new version)
```

## Tasks

### Task 1: Criar Configuração Semantic-Release
- [x] 1.1 Criar `.releaserc.json` na raiz do projeto
- [x] 1.2 Configurar plugins na ordem correta
- [x] 1.3 Testar localmente com `npm run release:test`

### Task 2: Criar Workflow GitHub Actions
- [x] 2.1 Criar `.github/workflows/semantic-release.yml`
- [x] 2.2 Configurar trigger para push em main
- [x] 2.3 Adicionar job de CI como dependência
- [x] 2.4 Configurar permissões necessárias

### Task 3: Configurar Secrets
- [ ] 3.1 Verificar `NPM_TOKEN` está configurado no repositório
- [ ] 3.2 Verificar `GITHUB_TOKEN` tem permissões adequadas
- [ ] 3.3 Documentar processo de rotação de tokens

### Task 4: Atualizar Documentação
- [x] 4.1 Atualizar `docs/versioning-and-releases.md` (remover "não implementado")
- [x] 4.2 Adicionar seção de troubleshooting
- [x] 4.3 Documentar como pular releases (`[skip ci]`)

### Task 5: Validação E2E
- [ ] 5.1 Criar PR de teste com commit `fix: test semantic release`
- [ ] 5.2 Merge e verificar workflow executa
- [ ] 5.3 Verificar nova versão no NPM
- [ ] 5.4 Testar `npx aios-core --version`

## Dependencies

### Dependências já instaladas (package.json)
```json
{
  "semantic-release": "^25.0.2",
  "@semantic-release/changelog": "^6.0.3",
  "@semantic-release/git": "^10.0.1"
}
```

### Secrets necessários
- `NPM_TOKEN` - Token de publicação NPM (já deve existir)
- `GITHUB_TOKEN` - Automático do GitHub Actions

## Risks & Mitigations

| Risco | Probabilidade | Impacto | Mitigação |
|-------|---------------|---------|-----------|
| NPM_TOKEN expirado | Média | Alto | Verificar antes de implementar |
| Commits não seguem conventional | Alta | Médio | Adicionar commitlint (futuro) |
| Release duplicado | Baixa | Médio | `[skip ci]` no commit de release |
| Falha parcial (tag sem NPM) | Baixa | Alto | Retry manual, documentar rollback |

## Success Metrics

| Métrica | Target | Como Medir |
|---------|--------|------------|
| Releases automáticos | 100% dos merges com fix:/feat: | GitHub Actions logs |
| Tempo merge → NPM | < 5 minutos | Timestamps |
| Zero releases manuais | 0 por sprint | Audit releases |

## Out of Scope

- Commitlint enforcement (pode ser story futura)
- Releases para branches de feature
- Canary/beta releases automáticos
- Multi-package monorepo releases

## Related Documentation

- [Versioning and Releases Guide](../../versioning-and-releases.md)
- [DevOps Version Management Task](../../../.aios-core/development/tasks/github-devops-version-management.md)
- [Release Checklist](../../../.aios-core/product/checklists/release-checklist.md)
- [Semantic Release Docs](https://semantic-release.gitbook.io/)

## File List

Files to create/modify:

| File | Action | Description |
|------|--------|-------------|
| `.releaserc.json` | CREATE | Semantic release configuration |
| `.github/workflows/semantic-release.yml` | CREATE | GitHub Actions workflow |
| `docs/versioning-and-releases.md` | UPDATE | Mark as implemented |
| `CHANGELOG.md` | AUTO | Will be auto-generated |

## QA Checklist

- [x] `.releaserc.json` válido (JSON syntax)
- [x] Workflow syntax válida (YAML lint)
- [x] Dry-run local passa (`npm run release:test`)
- [ ] CI passa antes do release
- [ ] NPM publish funciona
- [ ] GitHub Release criado com notas
- [ ] CHANGELOG.md atualizado
- [ ] Tag git criada corretamente

## Notes

### Conventional Commits Reference

```bash
# Patch release (bug fixes)
fix: resolve CLI argument parsing bug
fix(installer): handle spaces in paths

# Minor release (new features)
feat: add team collaboration mode
feat(agents): implement memory persistence

# Major release (breaking changes)
feat!: redesign CLI interface
fix!: change default config location

BREAKING CHANGE: The config file location changed from ~/.aios to ~/.config/aios

# No release (maintenance)
chore: update dependencies
docs: fix typo in readme
style: format code
test: add unit tests
ci: update workflow
```

### Emergency Manual Release

Se o automático falhar:
```bash
npm run version:patch   # ou minor/major
git push && git push --tags
npm publish
```

---

## Dev Notes

### Plugin Order is Critical

The semantic-release plugin order in `.releaserc.json` is **critical**:

1. `commit-analyzer` - Must be first (analyzes commits)
2. `release-notes-generator` - Generates notes from analyzed commits
3. `changelog` - Writes to CHANGELOG.md
4. `npm` - Publishes to NPM registry
5. `git` - Commits package.json + CHANGELOG.md changes
6. `github` - Creates GitHub Release

**Wrong order = failed releases.** For example, if `git` runs before `npm`, the version bump commit may not include the actual NPM publish.

### GitHub Actions Permissions

The workflow requires these permissions:
- `contents: write` - For git commits and tags
- `issues: write` - For release notes linking
- `pull-requests: write` - For PR comments
- `id-token: write` - For NPM provenance (optional but recommended)

### NPM Token Requirements

The `NPM_TOKEN` secret must:
- Be a granular access token (not legacy token)
- Have `Read and write` permissions for packages
- Be scoped to the `synkra-ai` organization (or appropriate scope)
- Not be expired

To create: NPM → Settings → Access Tokens → Generate New Token → Granular Access Token

### CI Job Dependency

The workflow uses `uses: ./.github/workflows/ci.yml` to run CI first. This ensures:
- Lint passes before release
- Tests pass before release
- Type checking passes before release

If CI fails, release is automatically skipped.

### Skip CI Pattern

The release commit uses `[skip ci]` to prevent infinite loops:
```
chore(release): 2.3.0 [skip ci]
```

Without this, the release commit would trigger another CI run, which could trigger another release attempt.

---

## Testing

### Unit Tests
N/A - This story creates configuration files, not code.

### Integration Tests

| Test | Command | Expected |
|------|---------|----------|
| JSON validity | `node -e "require('./.releaserc.json')"` | No errors |
| YAML validity | `npx yaml-lint .github/workflows/semantic-release.yml` | Valid |
| Dry-run | `npm run release:test` | Shows next version without publishing |

### E2E Tests

| Scenario | Steps | Expected Result |
|----------|-------|-----------------|
| Patch release | 1. Merge PR with `fix:` commit<br>2. Wait for workflow | Version bumps from 2.2.x to 2.2.x+1 |
| Minor release | 1. Merge PR with `feat:` commit<br>2. Wait for workflow | Version bumps from 2.x.y to 2.x+1.0 |
| Major release | 1. Merge PR with `feat!:` or `BREAKING CHANGE:`<br>2. Wait for workflow | Version bumps from x.y.z to x+1.0.0 |
| No release | 1. Merge PR with `chore:` or `docs:` commit<br>2. Wait for workflow | No version change, no NPM publish |
| NPM verification | 1. After release completes<br>2. Run `npm view aios-core version` | Shows new version |
| Installer verification | 1. After NPM update<br>2. Run `npx aios-core --version` | Shows new version |

### Smoke Tests

After implementation, perform these manual smoke tests:

1. **Workflow Trigger Test**
   - Create branch `test/semantic-release`
   - Add commit `fix: test semantic release automation`
   - Create PR and merge to main
   - Verify workflow runs in GitHub Actions

2. **Version Bump Test**
   - Check `package.json` version updated
   - Check git tag created
   - Check GitHub Release exists

3. **NPM Publish Test**
   - Run `npm view aios-core version`
   - Compare with GitHub Release version
   - Run `npx aios-core@latest --version`

---

## Change Log

| Date | Author | Change |
|------|--------|--------|
| 2025-12-18 | @devops (Gage) | Initial story creation |
| 2025-12-18 | @po (Pax) | Added CodeRabbit Integration, Dev Notes, Testing, Change Log sections |

---

**Story Status:** Ready for Implementation
**Assignee:** @devops (Gage)
**Reviewer:** @qa (Quinn)
