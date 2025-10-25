# Build Docker Image - Composite Action

GitHub Actions composite action per il build e push di immagini Docker multi-platform con supporto GitHub Container Registry e normalizzazione automatica dei path.

## Caratteristiche Principali

✅ **Multi-Platform Build** - Supporto per linux/amd64, linux/arm64, ecc.
✅ **Normalizzazione Automatica Path** - Risolve duplicazioni nel path delle immagini
✅ **GitHub Container Registry** - Integrazione nativa con ghcr.io
✅ **Build Arguments & Secrets** - Configurazione flessibile tramite JSON
✅ **Image Labels** - Metadata OCI standard automatici
✅ **Build Cache** - Ottimizzazione tempi di build con layer caching
✅ **Build Summary** - Report dettagliato in GitHub Actions

## Utilizzo Base

```yaml
- name: Build and Push Docker Image
  uses: ./.github/actions/build-docker-image
  with:
    context_path: '.'
    dockerfile_path: './Dockerfile'
    image_name: microservices/myservice
    image_tags: '1.0.0,latest'
    registry_url: 'ghcr.io'
    repository_owner: 'dm-italy'
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}
    platforms: 'linux/amd64'
```

## Docker Image Path Normalization

### ⚠️ Problema Comune

Quando usi `github.repository` nell'`image_name`, potresti creare duplicazioni:

```yaml
# ❌ GENERA DUPLICAZIONE
env:
  IMAGE_NAME: ${{ github.repository }}/service  # = dm-italy/microservices-1/service

jobs:
  build:
    steps:
      - uses: ./.github/actions/build-docker-image
        with:
          image_name: ${{ env.IMAGE_NAME }}      # dm-italy/microservices-1/service
          repository_owner: ${{ github.repository_owner }}  # dm-italy

# Risultato: ghcr.io/dm-italy/dm-italy/microservices-1/service ❌
```

### ✅ Soluzione Automatica

L'action **normalizza automaticamente** il path rimuovendo duplicazioni:

```bash
# Input (con duplicazione)
ghcr.io/dm-italy/dm-italy/microservices-1/dashboard

# Output (normalizzato automaticamente)
ghcr.io/dm-italy/microservices-1/dashboard ✓
```

### Logging della Normalizzazione

Quando viene applicata la normalizzazione, l'action logga:

```
⚠️  Docker image path normalized:
   Original:    ghcr.io/dm-italy/dm-italy/microservices-1/service
   Normalized:  ghcr.io/dm-italy/microservices-1/service
```

Se il path è già corretto:
```
Full image path (with owner): ghcr.io/dm-italy/microservices-1/service
```

## Input Parameters

### Obbligatori

| Parameter | Descrizione | Esempio |
|-----------|-------------|---------|
| `image_name` | Nome immagine (senza registry e owner) | `microservices/identity` |
| `image_tags` | Lista di tag separati da virgola | `1.0.0,latest,sha-abc123` |
| `registry_username` | Username per il registry | `${{ github.actor }}` |
| `registry_password` | Password/token per il registry | `${{ secrets.GITHUB_TOKEN }}` |

### Opzionali

| Parameter | Descrizione | Default | Esempio |
|-----------|-------------|---------|---------|
| `context_path` | Path del build context | `.` | `./services/api` |
| `dockerfile_path` | Path del Dockerfile | `./Dockerfile` | `./docker/Dockerfile.prod` |
| `registry_url` | URL del Docker registry | `ghcr.io` | `docker.io` |
| `repository_owner` | Owner del repository | `''` | `dm-italy` |
| `platforms` | Piattaforme target | `linux/amd64` | `linux/amd64,linux/arm64` |
| `build_args` | Build arguments (JSON) | `{}` | `{"ARG1":"value"}` |
| `secrets` | Build secrets (JSON) | `{}` | `{"TOKEN":"xxx"}` |
| `labels` | Image labels (JSON) | `{}` | `{"version":"1.0"}` |
| `push` | Push al registry | `true` | `false` |
| `load` | Load in Docker locale | `false` | `true` |
| `cache_from` | Cache source | `''` | `type=registry,ref=...` |
| `cache_to` | Cache destination | `''` | `type=registry,ref=...` |

## Output

| Output | Descrizione |
|--------|-------------|
| `image_digest` | Image digest (sha256:...) |
| `image_tags` | Full image references con tag |
| `image_metadata` | Build metadata JSON |

## Esempi Completi

### Build con github.repository (normalizzazione automatica)

```yaml
env:
  # IMAGE_NAME includerà owner/repo/service
  IMAGE_NAME: ${{ github.repository }}/dashboard  # dm-italy/microservices-1/dashboard

jobs:
  build:
    steps:
      - uses: ./.github/actions/build-docker-image
        with:
          image_name: ${{ env.IMAGE_NAME }}
          image_tags: '1.0.0,latest'
          repository_owner: ${{ github.repository_owner }}  # dm-italy
          registry_username: ${{ github.actor }}
          registry_password: ${{ secrets.GITHUB_TOKEN }}

# L'action normalizza automaticamente:
# ghcr.io/dm-italy/dm-italy/microservices-1/dashboard
# → ghcr.io/dm-italy/microservices-1/dashboard ✓
```

### Build Semplice (senza owner duplicato)

```yaml
- uses: ./.github/actions/build-docker-image
  with:
    image_name: 'microservices/api'  # Path semplice
    image_tags: '2.0.0,latest'
    repository_owner: 'dm-italy'
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}

# Risultato: ghcr.io/dm-italy/microservices/api:2.0.0 ✓
```

### Build con Build Args e Secrets

```yaml
- uses: ./.github/actions/build-docker-image
  with:
    image_name: 'services/backend'
    image_tags: '${{ needs.version.outputs.version }},latest'
    repository_owner: 'dm-italy'
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}
    build_args: |
      {
        "DOTNET_VERSION": "8.0",
        "BUILD_CONFIGURATION": "Release"
      }
    secrets: |
      {
        "NUGET_TOKEN": "${{ secrets.NUGET_FEED_PASSWORD }}",
        "NPM_TOKEN": "${{ secrets.NPM_TOKEN }}"
      }
    labels: |
      {
        "org.opencontainers.image.title": "Backend Service",
        "org.opencontainers.image.version": "${{ needs.version.outputs.version }}"
      }
```

### Build Multi-Platform

```yaml
- uses: ./.github/actions/build-docker-image
  with:
    image_name: 'apps/frontend'
    image_tags: 'latest,v1.0.0'
    repository_owner: 'dm-italy'
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}
    platforms: 'linux/amd64,linux/arm64'
    cache_from: 'type=registry,ref=ghcr.io/dm-italy/apps/frontend:buildcache'
    cache_to: 'type=registry,ref=ghcr.io/dm-italy/apps/frontend:buildcache,mode=max'
```

### Build Locale (senza push)

```yaml
- uses: ./.github/actions/build-docker-image
  with:
    image_name: 'test/service'
    image_tags: 'test'
    repository_owner: 'dm-italy'
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}
    push: false
    load: true  # Carica in Docker locale per testing
```

## Testing

### Test della Normalizzazione

```bash
# Esegui i test
.github/actions/build-docker-image/test-normalize.sh

# Output atteso
Testing Docker build image path normalization...

✓ PASS: ghcr.io/dm-italy/dm-italy/microservices-1/dashboard
  → ghcr.io/dm-italy/microservices-1/dashboard
✓ PASS: ghcr.io/dm-italy/dm-italy/service
  → ghcr.io/dm-italy/service
...
All tests passed! ✓
```

## Best Practices

### Scelta di image_name

**Opzione 1: Path Semplice (Raccomandato)**
```yaml
env:
  IMAGE_NAME: microservices/dashboard  # Senza owner

jobs:
  build:
    steps:
      - uses: ./.github/actions/build-docker-image
        with:
          image_name: ${{ env.IMAGE_NAME }}
          repository_owner: ${{ github.repository_owner }}

# Risultato: ghcr.io/dm-italy/microservices/dashboard ✓
```

**Opzione 2: Con github.repository (Normalizzazione Automatica)**
```yaml
env:
  IMAGE_NAME: ${{ github.repository }}/service  # Include owner/repo

jobs:
  build:
    steps:
      - uses: ./.github/actions/build-docker-image
        with:
          image_name: ${{ env.IMAGE_NAME }}
          repository_owner: ${{ github.repository_owner }}

# Normalizzato automaticamente a: ghcr.io/dm-italy/microservices-1/service ✓
```

**Opzione 3: Path Completo (No Owner Parameter)**
```yaml
env:
  IMAGE_NAME: dm-italy/microservices/service  # Owner incluso

jobs:
  build:
    steps:
      - uses: ./.github/actions/build-docker-image
        with:
          image_name: ${{ env.IMAGE_NAME }}
          # repository_owner: '' (omesso)

# Risultato: ghcr.io/dm-italy/microservices/service ✓
```

### Tagging Strategy

```yaml
# Semantic versioning + latest + commit SHA
image_tags: '${{ needs.version.outputs.version }},latest,${{ github.ref_name }}-${{ github.sha }}'

# Esempi di tag generati:
# - 1.0.5
# - latest
# - update-431f900cc9a0eda3b64d688068378753a409c32e
```

### Build Secrets vs Build Args

**Build Args** - Per valori non sensibili (accessibili nel layer history):
```yaml
build_args: |
  {
    "DOTNET_VERSION": "8.0",
    "NODE_VERSION": "18"
  }
```

**Build Secrets** - Per valori sensibili (non salvati nei layer):
```yaml
secrets: |
  {
    "NUGET_TOKEN": "${{ secrets.NUGET_FEED_PASSWORD }}",
    "NPM_TOKEN": "${{ secrets.NPM_TOKEN }}"
  }

# Nel Dockerfile:
# RUN --mount=type=secret,id=NUGET_TOKEN \
#     dotnet restore --source https://nuget.pkg.github.com/...
```

## Troubleshooting

### Problema: Path Duplicato

**Sintomo**:
```
pushing manifest for ghcr.io/dm-italy/dm-italy/service:1.0.0
```

**Soluzione**: Automatica! L'action normalizza il path. Verifica i log:
```
⚠️  Docker image path normalized:
   Original:    ghcr.io/dm-italy/dm-italy/service
   Normalized:  ghcr.io/dm-italy/service
```

### Problema: Multi-Platform Build Failed

**Sintomo**:
```
ERROR: failed to solve: platform linux/arm64 not supported
```

**Soluzioni**:
1. Verifica che Docker buildx sia installato
2. Usa `platforms: 'linux/amd64'` se non serve ARM
3. Controlla che il runner supporti emulazione ARM

### Problema: Build Secrets Non Funzionano

**Sintomo**:
```
ERROR: secret "NUGET_TOKEN" not found
```

**Soluzione**:
```dockerfile
# ❌ SBAGLIATO - Il secret non esiste come ENV
RUN dotnet restore --source https://nuget.pkg.github.com/...

# ✅ CORRETTO - Monta il secret
RUN --mount=type=secret,id=NUGET_TOKEN \
    export TOKEN=$(cat /run/secrets/NUGET_TOKEN) && \
    dotnet restore --source https://nuget.pkg.github.com/...
```

## Performance

### Build Cache Optimization

```yaml
# Usa layer caching per build più veloci
cache_from: 'type=registry,ref=ghcr.io/dm-italy/service:buildcache'
cache_to: 'type=registry,ref=ghcr.io/dm-italy/service:buildcache,mode=max'

# mode=max: salva tutti i layer intermedi
# mode=min: salva solo i layer finali
```

### Metriche Tipiche

- Build da zero: ~5-15 minuti (dipende dal progetto)
- Build con cache: ~1-3 minuti
- Push immagine: ~30-60 secondi (dipende dalla size)
- Multi-platform: 2-3x più lento di single-platform

## Changelog

### v1.1.0 (Current)
- ✨ Aggiunta normalizzazione automatica path immagini
- 🐛 Fix gestione duplicazioni owner (dm-italy/dm-italy)
- 📝 Documentazione estesa
- ✅ Aggiunto script di test

### v1.0.0
- 🎉 Release iniziale
- ✨ Multi-platform support
- ✨ Build args e secrets via JSON
- ✨ Auto labels OCI

## Support

- 📖 Documentazione: `/docs/workflows/COMPOSITE_ACTIONS_ARCHITECTURE.md`
- 🐛 Issues: GitHub Issues
- 📧 Email: devops@diemmegroup.com
