# Semantic Version Management Action

GitHub Action composita per gestire il versioning semantico con standard-version, generazione changelog automatica e gestione tag git.

## ✨ Features

- ✅ **Versioning automatico** basato su Conventional Commits
- 📝 **Changelog automatico** con standard-version
- 🏷️ **Gestione tag git** con prefix personalizzabili
- 📦 **Supporto progetti non-npm** (crea package.json automaticamente)
- 🚀 **Nessuna installazione dipendenze** (usa npx)
- 🧹 **Cleanup tag non pushati** automatico
- 🔄 **Git history completa** (fetch automatico)
- 🎯 **Monorepo-ready** con tag prefix

## 🚀 Quick Start

### Configurazione base

```yaml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # IMPORTANTE per standard-version

      - name: Version Management
        uses: dm-italy/github-dm-italy/actions/version-management@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Configurazione per monorepo

```yaml
- name: Version Management - App Factory
  uses: dm-italy/github-dm-italy/actions/version-management@main
  with:
    working_directory: 'apps/factory'
    tag_prefix: 'factory@'
    git_branch: 'main'

- name: Version Management - Quality Service
  uses: dm-italy/github-dm-italy/actions/version-management@main
  with:
    working_directory: 'services/quality'
    tag_prefix: 'quality@'
    git_branch: 'main'
```

## 📋 Inputs

| Input | Descrizione | Required | Default |
|-------|-------------|----------|---------|
| `working_directory` | Directory di lavoro del progetto | No | `.` |
| `node_version` | Versione Node.js | No | `20.x` |
| `tag_prefix` | Prefix personalizzato per i tag | No | `''` |
| `skip_version_bump` | Salta il bump della versione | No | `false` |
| `skip_tag_push` | Salta il push dei tag | No | `false` |
| `git_branch` | Branch git per il push | No | `update` |
| `cleanup_unpushed_tags` | Pulisce tag non pushati | No | `true` |
| `git_user_name` | Nome utente git per i commit | No | `gitbot` |
| `git_user_email` | Email utente git per i commit | No | `gitbot@dmconsulting.it` |
| `github_token` | Token per npm registry auth | No | `''` |
| `release_script` | Comando custom release | No | `npx standard-version` |

## 📤 Outputs

| Output | Descrizione | Esempio |
|--------|-------------|---------|
| `new_version` | Nuova versione dopo il bump | `1.2.3` |
| `version_changed` | Se la versione è cambiata | `true`/`false` |
| `changelog` | Estratto changelog per questa versione | `## [1.2.3]...` |
| `current_version` | Versione prima del bump | `1.2.2` |

## 🎯 Come funziona

1. **Setup Git**: Configura utente e fetcha history completa
2. **Package.json Check**: Crea package.json minimo se mancante
3. **Cleanup Tags**: Rimuove tag locali non pushati (opzionale)
4. **Standard-version Dry-run**: Verifica se ci sono modifiche da versionare
5. **Version Bump**: Esegue standard-version se necessario
6. **Tag Creation**: Crea tag con prefix (se specificato)
7. **Push**: Pusha branch e tag al remote

## 📝 Conventional Commits

Standard-version analizza i commit message per determinare il bump:

### Bump Patch (0.0.X)
```bash
fix: corretto bug nel form di login
fix(auth): risolto problema sessione
```

### Bump Minor (0.X.0)
```bash
feat: aggiunto supporto dark mode
feat(ui): nuovo componente DatePicker
```

### Bump Major (X.0.0)
```bash
feat!: cambio API autenticazione

BREAKING CHANGE: endpoint /auth deprecato
```

### Nessun bump
```bash
chore: aggiornamento dipendenze
docs: migliorata documentazione
style: formattazione codice
refactor: refactoring funzione utils
```

## 🔧 Configurazione avanzata

### Custom release script

```yaml
- name: Version Management
  uses: dm-italy/github-dm-italy/actions/version-management@main
  with:
    release_script: 'yarn release'  # invece di npx standard-version
```

### Configurazione .versionrc

Crea `.versionrc.json` nel tuo progetto per personalizzare standard-version:

```json
{
  "types": [
    {"type": "feat", "section": "Features"},
    {"type": "fix", "section": "Bug Fixes"},
    {"type": "chore", "hidden": true},
    {"type": "docs", "hidden": true}
  ],
  "bumpFiles": [
    {
      "filename": "package.json",
      "type": "json"
    }
  ]
}
```

### GitHub Packages authentication

Per progetti che usano GitHub Packages npm:

```yaml
- name: Version Management
  uses: dm-italy/github-dm-italy/actions/version-management@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

L'action creerà automaticamente `.npmrc` con:
```
@dm-italy:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=***
```

## 🐛 Troubleshooting

### "No version bump needed" ma in locale funziona

**Causa**: Shallow clone di GitHub Actions non include history completa.

**Soluzione**: L'action ora fa automaticamente `git fetch --unshallow`, ma assicurati anche di usare:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0  # Fetch all history
```

### Progetti senza package.json

L'action crea automaticamente un package.json minimo:

```json
{
  "name": "nome-directory",
  "version": "1.0.0",
  "private": true
}
```

### Debug logs

L'action ora mostra:
- 📜 Ultimi 5 commits
- 🔍 Output completo di standard-version --dry-run
- 🚀 Comando eseguito e risultato

Vedi [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) per altri problemi comuni.

## 📊 Esempi di output

### Version bump riuscito

```
🔍 Checking if version will change...
Running: npx standard-version --dry-run
📋 Dry-run output:
✔ bumping version in package.json from 1.2.2 to 1.3.0
✓ Version will be bumped
🚀 Running: npx standard-version --skip.tag
✓ New version: 1.3.0
Creating tag: factory@1.3.0
✓ Tag created: factory@1.3.0
```

### Nessun bump necessario

```
🔍 Checking if version will change...
📋 Dry-run output:
[no changes to be bumped]
ℹ No version bump needed (no feat/fix/breaking changes)
```

## 🔗 Link utili

- [Conventional Commits](https://www.conventionalcommits.org/)
- [standard-version](https://github.com/conventional-changelog/standard-version)
- [Semantic Versioning](https://semver.org/)

## 📄 License

MIT © DiemmeGroup DevOps
