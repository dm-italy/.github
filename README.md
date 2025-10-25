# dm-italy Organization GitHub Actions

Composite actions and workflow templates riutilizzabili per l'intera organizzazione dm-italy.

## 📋 Indice

- [Composite Actions Disponibili](#composite-actions-disponibili)
- [Workflow Templates](#workflow-templates)
- [Quick Start](#quick-start)
- [Configurazione Secrets](#configurazione-secrets)
- [Esempi Completi](#esempi-completi)
- [Troubleshooting](#troubleshooting)

---

## Composite Actions Disponibili

### 🚀 Deployment Actions

#### 1. `deploy-systemd`
Deploy di applicazioni .NET come servizi systemd su Linux con gestione completa del ciclo di vita.

**Caratteristiche:**
- ✅ Backup automatico prima del deploy
- ✅ Health check post-deployment
- ✅ Rollback automatico in caso di fallimento
- ✅ Gestione appsettings.json con merge delle configurazioni
- ✅ Supporto per configurazioni environment-specific

**Usage:**
```yaml
- name: Deploy to systemd
  uses: dm-italy/.github/actions/deploy-systemd@main
  with:
    ssh_key: ${{ secrets.SSH_KEY }}
    ssh_host: ${{ secrets.SSH_HOST }}
    ssh_user: ${{ secrets.SSH_USER }}
    service_name: 'my-service'
    deploy_path: '/opt/services/my-service'
    artifact_name: 'my-service-build'
    service_port: 8080
    appsettings_json: |
      {
        "ConnectionStrings": {
          "Default": "${{ secrets.DB_CONNECTION_STRING }}"
        }
      }
```

**Input principali:**
- `ssh_key`, `ssh_host`, `ssh_user` - Credenziali SSH per accesso al server
- `service_name` - Nome del servizio systemd
- `deploy_path` - Percorso di installazione sul server
- `artifact_name` - Nome dell'artifact GitHub Actions da deployare
- `service_port` - Porta su cui il servizio è in ascolto
- `appsettings_json` - JSON con configurazioni da applicare
- `dotnet_version` - Versione .NET (default: '7.0')
- `environment_name` - Environment name (Development/Staging/Production)
- `backup_previous` - Crea backup prima del deploy (default: true)
- `skip_health_check` - Salta il controllo di salute (default: false)
- `health_endpoint` - Endpoint per health check (default: '/health')

---

#### 2. `deploy-docker`
Deploy di applicazioni come container Docker usando docker-compose con supporto completo per configurazioni avanzate.

**Caratteristiche:**
- ✅ Normalizzazione automatica nomi immagini (rimuove segmenti duplicati)
- ✅ Supporto per volume mounts (JSON)
- ✅ Environment variables (JSON)
- ✅ Docker networks
- ✅ Health check post-deployment
- ✅ Cleanup immagini vecchie
- ✅ Login automatico a registry privati

**Usage:**
```yaml
- name: Deploy Docker Container
  uses: dm-italy/.github/actions/deploy-docker@main
  with:
    ssh_key: ${{ secrets.SSH_KEY }}
    ssh_host: ${{ secrets.SSH_HOST }}
    ssh_user: ${{ secrets.SSH_USER }}
    docker_image: 'ghcr.io/dm-italy/my-service:latest'
    container_name: 'my-service'
    host_port: 8080
    container_port: 80
    environment_vars: |
      {
        "ASPNETCORE_ENVIRONMENT": "Production",
        "ConnectionStrings__Default": "${{ secrets.DB_CONNECTION_STRING }}"
      }
    registry_url: 'ghcr.io'
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}
    docker_network: 'my-network'
```

**Input principali:**
- `docker_image` - Immagine Docker completa (registry/org/image:tag)
- `container_name` - Nome del container
- `host_port` - Porta esposta sull'host
- `container_port` - Porta del container
- `environment_vars` - Variabili d'ambiente in formato JSON
- `volume_mounts` - Mount di volumi in formato JSON (es: `{"./data": "/app/data"}`)
- `registry_url`, `registry_username`, `registry_password` - Credenziali registry
- `restart_policy` - Politica di restart (default: 'unless-stopped')
- `docker_network` - Nome della network Docker
- `cleanup_old_images` - Rimuovi immagini vecchie (default: true)

**Normalizzazione Immagini:**
L'action rimuove automaticamente segmenti duplicati nei path:
- `ghcr.io/dm-italy/dm-italy/repo:tag` → `ghcr.io/dm-italy/repo:tag`
- `registry.io/org/org/app:v1` → `registry.io/org/app:v1`

---

#### 3. `deploy-nginx`
Configurazione Nginx come reverse proxy con gestione automatica certificati SSL/TLS.

**Caratteristiche:**
- ✅ Configurazione automatica Let's Encrypt
- ✅ Certificati self-signed con auto-renewal (cron job)
- ✅ Force HTTPS redirect
- ✅ Configurazione reverse proxy ottimizzata
- ✅ Health check del servizio backend

**Usage:**
```yaml
- name: Configure Nginx with SSL
  uses: dm-italy/.github/actions/deploy-nginx@main
  with:
    ssh_key: ${{ secrets.SSH_KEY }}
    ssh_host: ${{ secrets.SSH_HOST }}
    ssh_user: ${{ secrets.SSH_USER }}
    service_name: 'my-service'
    backend_host: 'localhost'
    backend_port: 8080
    domain: 'service.example.com'
    ssl_mode: 'auto'
    force_ssl: 'true'
```

**SSL Modes:**
- `auto` - Certbot Let's Encrypt automatico (production)
- `self-signed` - Certificati self-signed con auto-renewal via cron
- `renew` - Forza rinnovo certificato esistente
- `disable` - Nessun SSL, solo HTTP

**Input principali:**
- `service_name` - Nome del servizio (per file nginx conf)
- `backend_host` - Host del backend (default: 'localhost')
- `backend_port` - Porta del backend
- `domain` - Dominio per il quale configurare Nginx
- `ssl_mode` - Modalità SSL (auto/self-signed/renew/disable)
- `force_ssl` - Forza redirect HTTP → HTTPS (default: true)
- `self_signed_passphrase` - Passphrase per certificati self-signed (opzionale)
- `force_reconfigure` - Forza riconfigurazione anche se esistente (default: false)

---

#### 4. `deploy-kubernetes`
Deploy completo su Kubernetes con creazione di tutte le risorse necessarie.

**Caratteristiche:**
- ✅ Deployment con rolling update strategy
- ✅ Service (ClusterIP, NodePort, LoadBalancer)
- ✅ ConfigMap per configurazioni
- ✅ Secret per dati sensibili
- ✅ Ingress con TLS
- ✅ PersistentVolumeClaim
- ✅ Health probes (liveness, readiness, startup)
- ✅ Resource limits e requests
- ✅ Rollback automatico su fallimento

**Usage:**
```yaml
- name: Deploy to Kubernetes
  uses: dm-italy/.github/actions/deploy-kubernetes@main
  with:
    kubeconfig: ${{ secrets.KUBECONFIG }}
    namespace: 'production'
    deployment_name: 'my-service'
    image: 'ghcr.io/dm-italy/my-service:v1.0.0'
    replicas: 3
    container_port: 80
    service_type: 'ClusterIP'
    ingress_enabled: 'true'
    ingress_host: 'service.example.com'
    config_map_data: |
      {
        "ASPNETCORE_ENVIRONMENT": "Production",
        "Logging__LogLevel__Default": "Information"
      }
    secret_data: |
      {
        "ConnectionStrings__Default": "${{ secrets.DB_CONNECTION_STRING }}",
        "ApiKey": "${{ secrets.API_KEY }}"
      }
```

**Input principali:**
- `kubeconfig` - Contenuto del file kubeconfig (base64 o plain)
- `namespace` - Kubernetes namespace
- `deployment_name` - Nome del deployment
- `image` - Immagine Docker completa
- `replicas` - Numero di repliche (default: 1)
- `container_port` - Porta del container
- `service_type` - Tipo di servizio (ClusterIP/NodePort/LoadBalancer)
- `ingress_enabled` - Abilita Ingress (default: false)
- `ingress_host` - Hostname per Ingress
- `ingress_tls_enabled` - Abilita TLS su Ingress (default: false)
- `config_map_data` - Dati ConfigMap in formato JSON
- `secret_data` - Dati Secret in formato JSON (saranno codificati base64)
- `pvc_enabled` - Abilita PersistentVolumeClaim (default: false)
- `resource_limits_cpu/memory` - Limiti di risorse
- `resource_requests_cpu/memory` - Richieste di risorse

---

### 🏗️ Build Actions

#### 5. `build-dotnet`
Build di progetti .NET con supporto completo per ABP Framework e NuGet feed privati.

**Caratteristiche:**
- ✅ Supporto multi-source NuGet (ABP, DevExpress, Telerik, GitHub Packages)
- ✅ Configurazione automatica NuGet.Config
- ✅ ABP CLI integration
- ✅ npm/yarn per progetti frontend integrati
- ✅ Cache delle dipendenze NuGet
- ✅ Output come artifact GitHub Actions

**Usage:**
```yaml
- name: Build .NET Project
  uses: dm-italy/.github/actions/build-dotnet@main
  with:
    dotnet_version: '8.0.x'
    project_path: '.'
    csproj_pattern: '**/*.HttpApi.Host.csproj'
    build_configuration: 'Release'
    version: '1.0.0'
    artifact_name: 'my-service-build'
    nuget_feed_password: ${{ secrets.NUGET_FEED_PASSWORD }}
    abp_nuget_api_key: ${{ secrets.ABP_NUGET_API_KEY }}
    devexpress_nuget_key: ${{ secrets.DEVEXPRESS_NUGET_KEY }}
    telerik_username: ${{ secrets.TELERIK_USERNAME }}
    telerik_password: ${{ secrets.TELERIK_PASSWORD }}
```

**Input principali:**
- `dotnet_version` - Versione .NET SDK (es: '8.0.x')
- `project_path` - Path del progetto
- `csproj_pattern` - Pattern per trovare .csproj (glob pattern)
- `build_configuration` - Configurazione build (Release/Debug)
- `version` - Versione da assegnare al build
- `artifact_name` - Nome dell'artifact di output
- `nuget_feed_password`, `abp_nuget_api_key`, etc. - Credenziali NuGet feeds

---

#### 6. `build-docker-image`
Build e push di immagini Docker con supporto multi-platform e caching ottimizzato.

**Caratteristiche:**
- ✅ Docker Buildx con multi-platform support
- ✅ Normalizzazione path immagini (rimuove duplicati)
- ✅ Build args e secrets
- ✅ Caching layer Docker
- ✅ OCI image labels
- ✅ Push automatico a registry

**Usage:**
```yaml
- name: Build and Push Docker Image
  uses: dm-italy/.github/actions/build-docker-image@main
  with:
    context_path: '.'
    dockerfile_path: './Dockerfile'
    image_name: 'my-service'
    image_tags: 'latest,v1.0.0,${{ github.sha }}'
    registry_url: 'ghcr.io'
    repository_owner: 'dm-italy'
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}
    platforms: 'linux/amd64,linux/arm64'
    build_args: |
      VERSION=1.0.0
      BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
    push: 'true'
```

**Input principali:**
- `context_path` - Docker build context
- `dockerfile_path` - Path al Dockerfile
- `image_name` - Nome dell'immagine
- `image_tags` - Tags separati da virgola (latest,v1.0.0,commit-sha)
- `registry_url` - URL del registry
- `repository_owner` - Owner dell'organizzazione/repository
- `platforms` - Piattaforme per build multi-arch (default: 'linux/amd64')
- `build_args` - Build arguments (formato KEY=VALUE, uno per riga)
- `push` - Esegui push al registry (default: true)

---

### 📦 Release Actions

#### 7. `create-github-release`
Creazione release GitHub con artifact, changelog e riferimenti Docker images.

**Caratteristiche:**
- ✅ Release notes automatiche da commit
- ✅ Upload automatico artifact (.zip)
- ✅ Documentazione immagini Docker nella release
- ✅ Supporto draft e pre-release
- ✅ Changelog integration

**Usage:**
```yaml
- name: Create GitHub Release
  uses: dm-italy/.github/actions/create-github-release@main
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    tag_name: 'v1.0.0'
    release_name: 'My Service v1.0.0'
    body: 'Release notes for version 1.0.0'
    generate_release_notes: 'true'
    draft: 'false'
    prerelease: 'false'
    artifact_path: './dist'
    artifact_name: 'my-service-v1.0.0.zip'
    docker_images: |
      [
        "ghcr.io/dm-italy/my-service:v1.0.0",
        "ghcr.io/dm-italy/my-service:latest"
      ]
```

**Input principali:**
- `github_token` - Token GitHub (GITHUB_TOKEN secret)
- `tag_name` - Tag Git per la release
- `release_name` - Nome della release
- `body` - Corpo della release (markdown)
- `generate_release_notes` - Genera note automatiche (default: true)
- `draft` - Crea come draft (default: false)
- `prerelease` - Marca come pre-release (default: false)
- `artifact_path` - Path all'artifact da allegare
- `artifact_name` - Nome del file .zip artifact
- `docker_images` - Array JSON con immagini Docker da documentare

---

#### 8. `version-management`
Gestione automatica versioning semantico con conventional commits e changelog.

**Caratteristiche:**
- ✅ Semantic versioning automatico (major.minor.patch)
- ✅ Conventional commits parsing
- ✅ Changelog generation
- ✅ Git tag creation e push
- ✅ Cleanup di tag non pushati
- ✅ Supporto tag prefix personalizzati

**Usage:**
```yaml
- name: Semantic Versioning
  uses: dm-italy/.github/actions/version-management@main
  with:
    working_directory: '.'
    tag_prefix: 'my-service@'
    cleanup_unpushed_tags: 'true'
    git_branch: 'main'
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

**Conventional Commits Format:**
```
feat: nuova funzionalità     → MINOR version bump
fix: correzione bug          → PATCH version bump
BREAKING CHANGE: ...         → MAJOR version bump
chore: aggiornamento deps    → Nessun version bump
docs: documentazione         → Nessun version bump
```

**Input principali:**
- `working_directory` - Directory del progetto
- `tag_prefix` - Prefisso per i tag (es: 'service@' → 'service@1.2.3')
- `cleanup_unpushed_tags` - Rimuovi tag locali non pushati (default: false)
- `git_branch` - Branch su cui operare
- `github_token` - Token GitHub

**Outputs:**
- `new_version` - Nuova versione calcolata (es: '1.2.3')
- `version_changed` - 'true' se la versione è cambiata

---

## Workflow Templates

I workflow templates sono disponibili nell'UI di GitHub quando crei un nuovo workflow. Si trovano nella sezione "By dm-italy".

### 1. Deploy .NET Service to systemd

Template completo per deploy di servizi .NET su Linux con systemd e Nginx.

**Include:**
- Build .NET con ABP Framework
- Deploy systemd con backup e rollback
- Configurazione Nginx con SSL

**File:** `workflow-templates/deploy-dotnet-systemd.yml`

---

### 2. Deploy Docker Service

Template per deploy di servizi tramite Docker con reverse proxy Nginx.

**Include:**
- Build immagine Docker multi-platform
- Deploy container con docker-compose
- Configurazione Nginx con SSL

**File:** `workflow-templates/deploy-docker-service.yml`

---

### 3. Semantic Versioning and Release

Template per gestione automatica versioning e release.

**Include:**
- Semantic versioning con conventional commits
- Changelog generation
- GitHub Release creation con artifact

**File:** `workflow-templates/versioning-and-release.yml`

---

## Quick Start

### 1. Setup Secrets Organization-Level

Configura i seguenti secrets a livello di organizzazione (`Settings` → `Secrets and variables` → `Actions`):

**Deployment Secrets:**
```
SSH_KEY          # Chiave SSH privata per accesso ai server
SSH_HOST         # Hostname/IP del server di deploy
SSH_USER         # Username SSH (es: deploy)
```

**NuGet Feeds:**
```
NUGET_FEED_PASSWORD    # Password per feed NuGet privato
ABP_NUGET_API_KEY      # API key ABP Commercial
DEVEXPRESS_NUGET_KEY   # NuGet key DevExpress
TELERIK_USERNAME       # Username Telerik
TELERIK_PASSWORD       # Password Telerik
```

**Database:**
```
DB_CONNECTION_STRING   # Connection string database
```

**Kubernetes (se usato):**
```
KUBECONFIG            # File kubeconfig completo
```

---

### 2. Esempio Workflow Completo

Crea `.github/workflows/deploy.yml` nel tuo repository:

```yaml
name: Build and Deploy

on:
  push:
    branches: [main, update]
  workflow_dispatch:

jobs:
  # Step 1: Versioning
  version:
    runs-on: ubuntu-latest
    outputs:
      new_version: ${{ steps.versioning.outputs.new_version }}
      version_changed: ${{ steps.versioning.outputs.version_changed }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Semantic Versioning
        id: versioning
        uses: dm-italy/.github/actions/version-management@main
        with:
          working_directory: '.'
          tag_prefix: 'my-service@'
          cleanup_unpushed_tags: 'true'
          git_branch: ${{ github.ref_name }}
          github_token: ${{ secrets.GITHUB_TOKEN }}

  # Step 2: Build
  build:
    needs: version
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build .NET Project
        uses: dm-italy/.github/actions/build-dotnet@main
        with:
          dotnet_version: '8.0.x'
          project_path: 'src'
          csproj_pattern: '**/*.HttpApi.Host.csproj'
          build_configuration: 'Release'
          version: ${{ needs.version.outputs.new_version }}
          artifact_name: 'my-service-build'
          nuget_feed_password: ${{ secrets.NUGET_FEED_PASSWORD }}
          abp_nuget_api_key: ${{ secrets.ABP_NUGET_API_KEY }}

  # Step 3: Deploy
  deploy:
    needs: [version, build]
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to systemd
        uses: dm-italy/.github/actions/deploy-systemd@main
        with:
          ssh_key: ${{ secrets.SSH_KEY }}
          ssh_host: ${{ secrets.SSH_HOST }}
          ssh_user: ${{ secrets.SSH_USER }}
          service_name: 'my-service'
          deploy_path: '/opt/services/my-service'
          artifact_name: 'my-service-build'
          service_port: 8080
          environment_name: 'Production'
          appsettings_json: |
            {
              "ConnectionStrings": {
                "Default": "${{ secrets.DB_CONNECTION_STRING }}"
              }
            }

      - name: Configure Nginx
        uses: dm-italy/.github/actions/deploy-nginx@main
        with:
          ssh_key: ${{ secrets.SSH_KEY }}
          ssh_host: ${{ secrets.SSH_HOST }}
          ssh_user: ${{ secrets.SSH_USER }}
          service_name: 'my-service'
          backend_port: 8080
          domain: 'myservice.example.com'
          ssl_mode: 'auto'

  # Step 4: Release
  release:
    needs: [version, build, deploy]
    if: needs.version.outputs.version_changed == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Create GitHub Release
        uses: dm-italy/.github/actions/create-github-release@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          tag_name: 'my-service@${{ needs.version.outputs.new_version }}'
          release_name: 'My Service v${{ needs.version.outputs.new_version }}'
          generate_release_notes: 'true'
```

---

## Esempi Completi

### Deploy Docker con Multi-Container

```yaml
- name: Deploy Multi-Container Application
  uses: dm-italy/.github/actions/deploy-docker@main
  with:
    ssh_key: ${{ secrets.SSH_KEY }}
    ssh_host: ${{ secrets.SSH_HOST }}
    ssh_user: ${{ secrets.SSH_USER }}
    docker_image: 'ghcr.io/dm-italy/my-app:v1.0.0'
    container_name: 'my-app'
    host_port: 8080
    container_port: 80
    environment_vars: |
      {
        "ASPNETCORE_ENVIRONMENT": "Production",
        "Redis__Configuration": "redis:6379",
        "RabbitMQ__Host": "rabbitmq"
      }
    volume_mounts: |
      {
        "./data": "/app/data",
        "./logs": "/app/logs"
      }
    docker_network: 'app-network'
    registry_url: 'ghcr.io'
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}
```

---

### Deploy Kubernetes con Ingress e TLS

```yaml
- name: Deploy to Kubernetes with Ingress
  uses: dm-italy/.github/actions/deploy-kubernetes@main
  with:
    kubeconfig: ${{ secrets.KUBECONFIG }}
    namespace: 'production'
    deployment_name: 'my-api'
    image: 'ghcr.io/dm-italy/my-api:v2.0.0'
    replicas: 3
    container_port: 80
    service_type: 'ClusterIP'
    service_port: 80
    ingress_enabled: 'true'
    ingress_host: 'api.example.com'
    ingress_tls_enabled: 'true'
    ingress_tls_secret_name: 'api-tls'
    config_map_data: |
      {
        "ASPNETCORE_ENVIRONMENT": "Production",
        "Logging__LogLevel__Default": "Information"
      }
    secret_data: |
      {
        "ConnectionStrings__Default": "${{ secrets.DB_CONNECTION_STRING }}",
        "Redis__Configuration": "${{ secrets.REDIS_CONNECTION }}",
        "RabbitMQ__Password": "${{ secrets.RABBITMQ_PASSWORD }}"
      }
    resource_limits_cpu: '1000m'
    resource_limits_memory: '1Gi'
    resource_requests_cpu: '500m'
    resource_requests_memory: '512Mi'
    liveness_probe_path: '/health'
    readiness_probe_path: '/ready'
```

---

### Build e Push Multi-Platform

```yaml
- name: Build Multi-Platform Docker Image
  uses: dm-italy/.github/actions/build-docker-image@main
  with:
    context_path: '.'
    dockerfile_path: './Dockerfile'
    image_name: 'my-microservice'
    image_tags: 'latest,v1.0.0,${{ github.sha }}'
    registry_url: 'ghcr.io'
    repository_owner: 'dm-italy'
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}
    platforms: 'linux/amd64,linux/arm64'
    build_args: |
      DOTNET_VERSION=8.0
      BUILD_CONFIGURATION=Release
      VERSION=1.0.0
    secrets: |
      NUGET_PASSWORD=${{ secrets.NUGET_FEED_PASSWORD }}
    push: 'true'
```

---

## Troubleshooting

### Problemi Comuni

#### 1. **Errore SSH: "Permission denied (publickey)"**

**Causa:** Chiave SSH non configurata correttamente.

**Soluzione:**
```bash
# Genera nuova coppia chiavi SSH
ssh-keygen -t ed25519 -C "github-actions" -f github_deploy_key -N ""

# Copia la chiave pubblica sul server
ssh-copy-id -i github_deploy_key.pub user@server

# Aggiungi la chiave privata come secret GitHub
# Settings → Secrets → New repository secret
# Name: SSH_KEY
# Value: [contenuto di github_deploy_key]
```

---

#### 2. **Deploy Nginx: "Certificate verification failed"**

**Causa:** Let's Encrypt non riesce a verificare il dominio.

**Soluzione:**
- Verifica che il dominio punti all'IP corretto
- Porta 80 deve essere accessibile pubblicamente
- Firewall deve permettere traffico HTTP/HTTPS
- Per testing, usa `ssl_mode: 'self-signed'`

---

#### 3. **Docker: "Error response from daemon: manifest not found"**

**Causa:** Immagine Docker non trovata o problemi di autenticazione.

**Soluzione:**
```yaml
# Verifica login al registry
- name: Login to GitHub Container Registry
  run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

# Verifica nome immagine (case-sensitive!)
docker_image: 'ghcr.io/dm-italy/my-service:latest'  # Corretto
docker_image: 'ghcr.io/DM-Italy/My-Service:latest'  # Sbagliato
```

---

#### 4. **NuGet Restore Failed: 401 Unauthorized**

**Causa:** Credenziali NuGet feed non valide.

**Soluzione:**
```yaml
# Verifica che i secret siano configurati
- ABP_NUGET_API_KEY
- DEVEXPRESS_NUGET_KEY
- TELERIK_USERNAME / TELERIK_PASSWORD

# Testa credenziali manualmente
dotnet nuget add source "https://nuget.abp.io/..." \
  --name ABP \
  --username abp \
  --password ${{ secrets.ABP_NUGET_API_KEY }}
```

---

#### 5. **Health Check Failed: Service unhealthy**

**Causa:** Il servizio non risponde all'endpoint di health check.

**Soluzione:**
```yaml
# Aumenta timeout health check
skip_health_check: 'false'
health_endpoint: '/health'  # Verifica endpoint corretto

# Oppure disabilita temporaneamente
skip_health_check: 'true'

# Debug: accedi al server e controlla
ssh user@server
systemctl status my-service
journalctl -u my-service -f
curl http://localhost:8080/health
```

---

#### 6. **Kubernetes: ImagePullBackOff**

**Causa:** Kubernetes non riesce a scaricare l'immagine Docker.

**Soluzione:**
```yaml
# Crea image pull secret
kubectl create secret docker-registry regcred \
  --docker-server=ghcr.io \
  --docker-username=${{ github.actor }} \
  --docker-password=${{ secrets.GITHUB_TOKEN }} \
  --namespace=production

# Aggiungi al deployment
spec:
  imagePullSecrets:
    - name: regcred
```

---

#### 7. **Version Management: No new version created**

**Causa:** Commit messages non seguono conventional commits format.

**Soluzione:**
```bash
# Formato corretto conventional commits
git commit -m "feat: add user authentication"     # MINOR bump
git commit -m "fix: resolve login timeout"        # PATCH bump
git commit -m "feat!: redesign API endpoints"     # MAJOR bump

# Formato sbagliato (nessun bump)
git commit -m "updated code"
git commit -m "changes"
```

---

## Configurazione Secrets

### Repository Secrets (per singolo repo)

`Settings` → `Secrets and variables` → `Actions` → `New repository secret`

### Organization Secrets (per tutti i repo)

`Organization Settings` → `Secrets and variables` → `Actions` → `New organization secret`

**Repository access:** Seleziona "Private repositories" o "Selected repositories"

---

### Lista Completa Secrets Raccomandati

```bash
# SSH Deployment
SSH_KEY                 # Chiave privata SSH per deploy
SSH_HOST                # Server hostname/IP
SSH_USER                # Username SSH

# Container Registries
GHCR_TOKEN              # GitHub Container Registry (alternativa a GITHUB_TOKEN)
DOCKER_HUB_USERNAME     # Docker Hub username (se usato)
DOCKER_HUB_TOKEN        # Docker Hub token

# NuGet Feeds
NUGET_FEED_PASSWORD     # Password feed NuGet privato
ABP_NUGET_API_KEY       # ABP Commercial NuGet key
DEVEXPRESS_NUGET_KEY    # DevExpress NuGet key
TELERIK_USERNAME        # Telerik username
TELERIK_PASSWORD        # Telerik password

# Databases
DB_CONNECTION_STRING    # Connection string PostgreSQL/SQL Server
REDIS_CONNECTION        # Redis connection string
MONGODB_URI             # MongoDB connection URI

# Message Brokers
RABBITMQ_HOST           # RabbitMQ hostname
RABBITMQ_USERNAME       # RabbitMQ username
RABBITMQ_PASSWORD       # RabbitMQ password

# Kubernetes
KUBECONFIG              # File kubeconfig completo (base64 encoded)
K8S_CLUSTER_URL         # Kubernetes cluster URL (alternativa)
K8S_TOKEN               # Kubernetes service account token

# Application Secrets
API_KEY                 # API keys per servizi esterni
JWT_SECRET              # Secret per JWT token generation
ENCRYPTION_KEY          # Chiave encryption dati sensibili

# Monitoring & Logging
SENTRY_DSN              # Sentry error tracking DSN
ELASTIC_CLOUD_ID        # Elasticsearch Cloud ID
ELASTIC_API_KEY         # Elasticsearch API key
```

---

## Best Practices

### 1. Security
- ✅ **Ruota i secret regolarmente** (ogni 90 giorni minimo)
- ✅ **Usa secret organization-level** per ridurre duplicazione
- ✅ **Non loggare mai secret nei workflow** (GitHub li maschera ma meglio evitare)
- ✅ **Usa least privilege** per service accounts

### 2. Versioning
- ✅ **Usa conventional commits** per versioning automatico
- ✅ **Tag format:** `service-name@1.2.3` per monorepo
- ✅ **CHANGELOG.md** generato automaticamente con `version-management`

### 3. Deployment
- ✅ **Sempre backup prima di deploy** (abilitato di default in deploy-systemd)
- ✅ **Health check obbligatori** in production
- ✅ **Rollback automatico** se health check fallisce
- ✅ **Blue-Green deployments** per zero-downtime

### 4. Docker
- ✅ **Multi-stage builds** per immagini più piccole
- ✅ **Non usare :latest in production** (usa version tag specifico)
- ✅ **Scan vulnerabilità** con Trivy o Snyk
- ✅ **Cleanup immagini vecchie** per risparmiare spazio

### 5. Monitoring
- ✅ **Configura health endpoints** (`/health`, `/ready`)
- ✅ **Centralizza logging** (Elasticsearch, CloudWatch)
- ✅ **Alert su deployment failure** (Slack, email)
- ✅ **Retention policy** per artifact e log

---

## Supporto e Contributi

### Problemi o Domande

Apri una issue nel repository: [dm-italy/.github/issues](https://github.com/dm-italy/.github/issues)

### Contribuire

1. Fork del repository
2. Crea branch feature (`git checkout -b feature/amazing-action`)
3. Commit con conventional commits (`git commit -m 'feat: add amazing action'`)
4. Push al branch (`git push origin feature/amazing-action`)
5. Apri Pull Request

### Testing Locale

Per testare le action localmente, usa [act](https://github.com/nektos/act):

```bash
# Installa act
brew install act

# Testa workflow
act -W .github/workflows/deploy.yml --secret-file .secrets

# Testa job specifico
act -j build --secret-file .secrets
```

---

## Licenza

Queste GitHub Actions sono di proprietà di **dm-italy** e sono destinate all'uso interno dell'organizzazione.

---

## Changelog

### v1.0.0 (2024)
- ✅ Initial release con 8 composite actions
- ✅ 3 workflow templates
- ✅ Documentazione completa
- ✅ Supporto .NET 7/8, Docker, Kubernetes
- ✅ Nginx reverse proxy con SSL automatico
- ✅ Semantic versioning integration

---

**Ultimo aggiornamento:** $(date -u +'%Y-%m-%d')
