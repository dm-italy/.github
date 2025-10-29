# Create GitHub Release Action

GitHub Action composita per creare release GitHub con artifacts, changelog automatico e riferimenti a Docker images.

## ✨ Features

- 📅 **Release date automatica** (formato UTC)
- 📝 **Estrazione CHANGELOG.md** automatica
- 🏷️ **Template strutturato** con metadata (commit, branch, date)
- 🐳 **Supporto Docker images** con pull commands
- 📦 **Upload artifacts** automatico
- 🔗 **Link comparison** con tag precedente
- 📊 **Release summary** dettagliato
- ✅ **softprops/action-gh-release** (action aggiornata, non deprecata)

## 🚀 Quick Start

### ⚠️ Requisiti

Il workflow che usa questa action **DEVE** avere i permessi di scrittura:

```yaml
permissions:
  contents: write  # Required for creating releases
```

### Release base con changelog

```yaml
- name: Create Release
  uses: dm-italy/github-dm-italy/actions/create-github-release@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    tag_name: v1.0.0
```

Questo genererà una release con:
- Titolo: `v1.0.0`
- Data release (UTC)
- Commit SHA e branch
- Contenuto estratto da `CHANGELOG.md`

### Release con artifacts

```yaml
- name: Build artifacts
  run: |
    # Build your artifacts
    mkdir -p dist
    cp build/*.zip dist/

- name: Create Release
  uses: dm-italy/github-dm-italy/actions/create-github-release@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    tag_name: gateway-v${{ env.VERSION }}
    release_name: Gateway v${{ env.VERSION }}
    artifacts: 'windows-x64,linux-x64,arm-packages'
```

### Release con Docker images

```yaml
- name: Create Release
  uses: dm-italy/github-dm-italy/actions/create-github-release@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    tag_name: v1.0.0
    docker_images: |
      [
        "ghcr.io/dm-italy/app:1.0.0",
        "ghcr.io/dm-italy/app:latest"
      ]
```

### Release completa (come machine.git)

```yaml
- name: Create GitHub Release
  uses: dm-italy/github-dm-italy/actions/create-github-release@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    tag_name: gateway-v${{ env.PACKAGE_VERSION }}
    release_name: Gateway v${{ env.PACKAGE_VERSION }}
    artifacts: 'windows-x64,linux-x64,arm-packages,drivers-windows'
    docker_images: |
      [
        "ghcr.io/dm-italy/gateway:${{ env.PACKAGE_VERSION }}",
        "ghcr.io/dm-italy/gateway:latest"
      ]
    previous_tag: 'gateway-v1.0.0'
```

## 📋 Inputs

| Input | Descrizione | Required | Default |
|-------|-------------|----------|---------|
| `github_token` | GitHub token per creare release | ✅ Yes | - |
| `tag_name` | Nome del tag release (es. v1.0.0) | ✅ Yes | - |
| `release_name` | Nome della release (default: tag_name) | No | `''` |
| `body` | Testo custom aggiuntivo (markdown) | No | `''` |
| `draft` | Crea come draft release | No | `'false'` |
| `prerelease` | Marca come prerelease | No | `'false'` |
| `generate_release_notes` | Auto-genera note da commits | No | `'true'` |
| `artifacts` | Lista artifacts separati da virgola | No | `''` |
| `docker_images` | Array JSON di Docker images | No | `'[]'` |
| `previous_tag` | Tag precedente per comparison link | No | `''` |

## 📤 Outputs

| Output | Descrizione | Esempio |
|--------|-------------|---------|
| `release_id` | ID della release creata | `123456` |
| `release_url` | URL della release | `https://github.com/org/repo/releases/tag/v1.0.0` |
| `release_upload_url` | URL per upload assets | (upload URL) |

## 📝 Template Release

La release generata segue questo formato:

```markdown
# v1.0.0

**Release Date:** 2025-10-28 10:30:00 UTC
**Commit:** abc123def456
**Branch:** main

## [1.0.0] (2025-10-28)

### Features

* Add new authentication system
* Improve performance

### Bug Fixes

* Fix login timeout issue

## Build Artifacts

This release includes:
- Windows x64 builds
- Linux x64 builds
- ARM packages

## 🐳 Docker Images

This release includes the following Docker images:

- `ghcr.io/dm-italy/app:1.0.0`
- `ghcr.io/dm-italy/app:latest`

**Pull command:**
```bash
docker pull ghcr.io/dm-italy/app:1.0.0
```

---

**Full Changelog**: [v0.9.0...v1.0.0](https://github.com/org/repo/compare/v0.9.0...v1.0.0)
```

## 🎯 Workflow completo esempio

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write  # Required for creating releases

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: |
          # Your build steps
          mkdir -p dist
          # Build artifacts...

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-artifacts
          path: dist/

  release:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # For changelog

      - name: Get version
        id: version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

      - name: Create Release
        uses: dm-italy/github-dm-italy/actions/create-github-release@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          tag_name: ${{ github.ref_name }}
          release_name: Release ${{ steps.version.outputs.version }}
          artifacts: 'build-artifacts'
          docker_images: |
            [
              "ghcr.io/${{ github.repository }}:${{ steps.version.outputs.version }}",
              "ghcr.io/${{ github.repository }}:latest"
            ]
```

## 🔧 Integrazione con version-management

Combina con l'action version-management per workflow completo:

```yaml
- name: Version Management
  id: version
  uses: dm-italy/github-dm-italy/actions/version-management@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}

- name: Create Release
  if: steps.version.outputs.version_changed == 'true'
  uses: dm-italy/github-dm-italy/actions/create-github-release@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    tag_name: v${{ steps.version.outputs.new_version }}
    previous_tag: v${{ steps.version.outputs.current_version }}
```

## 📦 Artifacts

Gli artifacts vengono:
1. Scaricati da GitHub Actions artifacts
2. Compressi in `.zip`
3. Uploadati alla release automaticamente

Directory artifacts:
```
./release-artifacts/
├── windows-x64/
│   └── app.exe
├── linux-x64/
│   └── app
└── arm-packages/
    └── app.deb
```

Risultato:
- `windows-x64.zip`
- `linux-x64.zip`
- `arm-packages.zip`

## 🐳 Docker Images Format

L'input `docker_images` accetta un array JSON:

```yaml
docker_images: |
  [
    "ghcr.io/dm-italy/app:1.0.0",
    "ghcr.io/dm-italy/app:latest",
    "ghcr.io/dm-italy/app:stable"
  ]
```

Genera una sezione formattata con:
- Lista delle immagini
- Pull command per la prima immagine

## 📊 Release Summary

L'action genera un summary nel workflow con:
- ✅ Tag e nome release
- 📅 Data release
- 🔗 URL release
- 📝 Status (Draft/Prerelease/Published)
- 📦 Lista artifacts
- 🐳 Docker images

## 🔒 Security

### Command Injection Protection

Questa action è **protetta da command injection**. Changelog e contenuti custom con caratteri speciali non vengono eseguiti come comandi shell.

**Esempio problema risolto:**
```
# Changelog contenente: rawValue, MachineDeltaDataEto
# Prima: ❌ rawValue: command not found
# Ora: ✅ Contenuto visualizzato correttamente
```

Tutti i contenuti utente passano attraverso heredoc con quote (`<<'EOF'`) per prevenire l'esecuzione di codice.

## 🆚 Differenze da actions/create-release

| Feature | actions/create-release | Questa action |
|---------|----------------------|---------------|
| Status | ⚠️ Deprecata | ✅ Attiva |
| Action base | - | softprops/action-gh-release |
| Release date | ❌ | ✅ Auto |
| Changelog | ❌ | ✅ Auto da CHANGELOG.md |
| Template | ❌ Base | ✅ Strutturato |
| Docker images | ❌ | ✅ Con pull commands |
| Artifacts | ❌ Manuale | ✅ Automatico |
| Security | ⚠️ Command injection | ✅ Protetto |

## 🔗 Link utili

- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github)
- [Semantic Versioning](https://semver.org/)

## 📄 License

MIT © DiemmeGroup DevOps
