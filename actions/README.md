# Composite Actions per Build e Deployment

Questo directory contiene **composite actions riutilizzabili** per il build e deployment dei microservizi DiemmeGroup.

## 📋 Indice

- [Panoramica](#panoramica)
- [Build Actions](#build-actions)
- [Deployment Actions](#deployment-actions)
- [Esempi di Utilizzo](#esempi-di-utilizzo)
- [Migrazione dei Workflow Esistenti](#migrazione-dei-workflow-esistenti)
- [Troubleshooting](#troubleshooting)

## 🎯 Panoramica

Le composite actions centralizzano la logica di build e deployment in componenti riutilizzabili, eliminando duplicazioni e garantendo consistenza.

### Vantaggi

✅ **DRY (Don't Repeat Yourself)**: La logica esiste una volta sola
✅ **Manutenibilità**: Modifiche in un solo posto per tutti i servizi
✅ **Consistenza**: Tutti i servizi usano la stessa logica
✅ **Testabilità**: Actions testate indipendentemente
✅ **Scalabilità**: Facile aggiungere nuovi servizi

### Architettura

```
.github/actions/
├── version-management/    # Gestione versioni semantiche
├── build-dotnet/          # Build progetti .NET con ABP
├── build-docker-image/    # Build immagini Docker multi-platform
├── create-github-release/ # Creazione GitHub releases
├── deploy-systemd/        # Deploy servizi .NET con systemd
├── deploy-docker/         # Deploy container Docker
├── deploy-kubernetes/     # Deploy su cluster Kubernetes
└── deploy-nginx/          # Configurazione Nginx + SSL
```

## 🔨 Build Actions

### 1. version-management

Gestione semantica delle versioni con standard-version, changelog e git tags.

**Caratteristiche:**
- Cleanup automatico tag non pushati
- Semantic versioning con standard-version
- Generazione changelog automatica
- Estrazione note di rilascio
- Push tags opzionale
- Dry-run mode per testing

**Input principali:**
```yaml
- node_version: Versione Node.js (default: 16.x)
- release_as: Tipo release (major/minor/patch o versione specifica)
- prerelease: Identificatore prerelease (alpha/beta/rc)
- dry_run: Esegui senza effettuare modifiche
- cleanup_unpushed_tags: Pulisci tag non pushati (default: true)
```

**Output:**
```yaml
- version: Versione generata (es. 1.2.3)
- version_tag: Tag versione con prefisso v (es. v1.2.3)
- changelog: Estratto changelog per questa versione
- previous_version: Versione precedente al bump
```

### 2. build-dotnet

Build di progetti .NET con supporto ABP Framework e configurazione multi-source NuGet.

**Caratteristiche:**
- Configurazione NuGet multi-source (ABP, GitHub Packages, Telerik, DevExpress)
- Supporto ABP CLI e install-libs
- Gestione versione in common.props
- Supporto npm/Node.js per frontend ABP
- Build, test e publish
- Creazione artifact per deployment

**Input principali:**
```yaml
- dotnet_version: Versione .NET SDK (default: 8.0.x)
- project_path: Path directory progetto
- csproj_pattern: Pattern file .csproj
- version: Versione da impostare in common.props
- artifact_name: Nome artifact di build
- install_abp_cli: Installa ABP CLI (default: false)
- skip_tests: Salta esecuzione test (default: true)
- nuget_feed_username: Username GitHub Packages
- nuget_feed_password: Token GitHub Packages
- abp_nuget_api_key: API key ABP Commercial
```

**Output:**
```yaml
- artifact_name: Nome artifact creato
- publish_path: Path file pubblicati
```

### 3. build-docker-image

Build e push di immagini Docker con supporto multi-platform.

**Caratteristiche:**
- Build multi-platform (linux/amd64, linux/arm64)
- Login automatico registry (GHCR, Docker Hub)
- Supporto build args e labels
- Cache ottimizzate per build veloci
- Metadata automatiche OCI
- Push opzionale al registry

**Input principali:**
```yaml
- image_name: Nome immagine (org/image-name)
- image_tags: Tag separati da virgola (latest,v1.0.0)
- registry_url: URL registry (default: ghcr.io)
- registry_username: Username registry
- registry_password: Password/token registry
- platforms: Piattaforme target (default: linux/amd64)
- build_args: Build arguments come JSON
- push: Push al registry (default: true)
```

**Output:**
```yaml
- image_digest: Digest immagine (sha256:...)
- image_tags: Riferimenti completi immagine
- image_metadata: Metadata build JSON
```

### 4. create-github-release

Creazione GitHub release con artifact, changelog e riferimenti Docker.

**Caratteristiche:**
- Creazione release automatica
- Allegato artifact da workflow
- Inclusione riferimenti Docker images
- Generazione note rilascio
- Supporto draft e prerelease
- Link confronto versioni

**Input principali:**
```yaml
- github_token: Token GitHub per release
- tag_name: Nome tag release (v1.0.0)
- release_name: Nome release (default: tag_name)
- body: Descrizione release (markdown)
- draft: Crea come draft (default: false)
- prerelease: Marca come prerelease (default: false)
- artifacts: Lista artifact da allegare
- docker_images: Array JSON immagini Docker
```

**Output:**
```yaml
- release_id: ID release creata
- release_url: URL release
- release_upload_url: URL upload asset
```

## 🚀 Deployment Actions

### 1. deploy-systemd

Deploy di applicazioni .NET come servizi systemd su server Linux.

**Caratteristiche:**
- Verifica prerequisiti (.NET runtime, systemd)
- Backup automatico deployment precedente
- Gestione systemd unit file
- Health check con retry logic
- Cleanup automatico vecchi backup

**Input principali:**
```yaml
- ssh_key: Chiave SSH per autenticazione
- ssh_host: Host target
- service_name: Nome servizio systemd
- deploy_path: Path di deployment
- artifact_name: Nome artifact da scaricare
- appsettings_json: Configurazione JSON
- service_port: Porta applicazione
```

**Output:**
```yaml
- deployment_status: success/failed
- service_url: URL locale servizio
- health_check_passed: true/false
```

### 2. deploy-docker

Deploy di container Docker su host remoto.

**Caratteristiche:**
- Verifica installazione Docker
- Login registry (GHCR, Docker Hub, etc.)
- Gestione container esistenti
- Environment variables da JSON
- Volume mounts configurabili
- Network mode configurabile
- Health check con retry logic
- Cleanup immagini vecchie

**Input principali:**
```yaml
- ssh_key: Chiave SSH
- ssh_host: Docker host
- docker_image: Immagine completa (registry/image:tag)
- container_name: Nome container
- host_port: Porta host
- container_port: Porta container
- environment_vars: JSON object con variabili ambiente
- volume_mounts: JSON array di mount points
```

**Output:**
```yaml
- deployment_status: success/failed
- container_id: ID container Docker
- health_check_passed: true/false
```

### 3. deploy-nginx

Configurazione Nginx reverse proxy con gestione SSL/TLS.

**Caratteristiche:**
- Installazione/verifica Nginx
- Configurazione reverse proxy
- SSL/TLS con Let's Encrypt
- SSL/TLS con certificati self-signed (OpenSSL)
- Auto-renewal certificati
- Backup configurazioni
- Force HTTPS redirect
- Health check HTTP/HTTPS

**Input principali:**
```yaml
- ssh_key: Chiave SSH
- ssh_host: Server Nginx
- service_name: Nome servizio (per config file)
- backend_host: Host backend
- backend_port: Porta backend
- domain: FQDN dominio
- ssl_mode: auto/renew/disable/self-signed
- self_signed_passphrase: Passphrase per .pfx (se self-signed)
```

**Output:**
```yaml
- configuration_status: success/failed
- ssl_enabled: true/false
- certificate_expiry_days: Giorni alla scadenza
```

### 4. deploy-kubernetes

Deploy di applicazioni su cluster Kubernetes con orchestrazione completa.

**Caratteristiche:**
- Verifica installazione kubectl
- Setup kubeconfig opzionale
- Gestione namespace automatica
- ConfigMap per environment variables
- Secret per dati sensibili
- Deployment con rolling updates
- Service (ClusterIP/NodePort/LoadBalancer)
- Ingress con supporto TLS
- PersistentVolumeClaim per storage
- Resource limits/requests (CPU/Memory)
- Liveness e readiness probes
- Health check post-deployment
- Rollback automatico su failure
- ImagePullSecret per private registries

**Input principali:**
```yaml
- ssh_key: Chiave SSH per nodo kubectl
- ssh_host: Host con accesso kubectl
- kubeconfig: Config K8s (base64) - opzionale
- namespace: Namespace Kubernetes
- docker_image: Immagine Docker completa
- app_name: Nome applicazione
- replicas: Numero repliche pod (default: 1)
- container_port: Porta container
- service_type: Tipo service (ClusterIP/NodePort/LoadBalancer)
- environment_vars: JSON variabili ambiente (→ ConfigMap)
- secrets: JSON valori sensibili (→ K8s Secret)
- ingress_enabled: Abilita Ingress (default: false)
- ingress_host: Hostname Ingress
- ingress_tls_enabled: Abilita TLS (default: false)
- cpu_request/cpu_limit: Resource limits CPU
- memory_request/memory_limit: Resource limits memoria
- rollback_on_failure: Rollback automatico (default: true)
```

**Output:**
```yaml
- deployment_status: success/failed/rolled_back
- rollout_status: Stato rollout
- service_endpoint: Endpoint servizio
- health_check_passed: true/false
```

## 📚 Esempi di Utilizzo

### Esempio Completo Build Workflow

```yaml
name: Build - Factory Service

on:
  push:
    branches: [main, update]
  workflow_dispatch:
    inputs:
      release_type:
        description: 'Release type'
        required: false
        type: choice
        options:
          - patch
          - minor
          - major
        default: patch

env:
  SERVICE_NAME: factory-service
  DOTNET_VERSION: '8.0.x'
  REGISTRY: ghcr.io

jobs:
  # =====================================================
  # VERSION MANAGEMENT
  # =====================================================
  set-version:
    name: Set Version
    runs-on: [self-hosted, Linux, X64]
    outputs:
      version: ${{ steps.versioning.outputs.version }}
      version_tag: ${{ steps.versioning.outputs.version_tag }}
      changelog: ${{ steps.versioning.outputs.changelog }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Necessario per standard-version

      - name: Manage version
        id: versioning
        uses: ./.github/actions/version-management
        with:
          release_as: ${{ github.event.inputs.release_type || 'patch' }}
          cleanup_unpushed_tags: 'true'

  # =====================================================
  # BUILD .NET PROJECT
  # =====================================================
  build-dotnet:
    name: Build .NET Project
    needs: set-version
    runs-on: [self-hosted, Linux, X64]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build .NET project
        uses: ./.github/actions/build-dotnet
        with:
          dotnet_version: ${{ env.DOTNET_VERSION }}
          project_path: services/factory
          csproj_pattern: 'services/factory/**/*.HttpApi.Host.csproj'
          version: ${{ needs.set-version.outputs.version }}
          artifact_name: ${{ env.SERVICE_NAME }}-${{ needs.set-version.outputs.version }}-${{ github.run_number }}
          install_abp_cli: 'true'
          skip_tests: 'false'
          nuget_feed_username: ${{ github.actor }}
          nuget_feed_password: ${{ secrets.GITHUB_TOKEN }}
          abp_nuget_api_key: ${{ secrets.ABP_NUGET_API_KEY }}
          devexpress_nuget_key: ${{ secrets.DEVEXPRESS_NUGET_KEY }}
          telerik_username: ${{ secrets.TELERIK_USERNAME }}
          telerik_password: ${{ secrets.TELERIK_PASSWORD }}

  # =====================================================
  # BUILD DOCKER IMAGE
  # =====================================================
  build-docker:
    name: Build Docker Image
    needs: [set-version, build-dotnet]
    runs-on: [self-hosted, Linux, X64]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build and push Docker image
        uses: ./.github/actions/build-docker-image
        with:
          context_path: services/factory
          dockerfile_path: services/factory/Dockerfile
          image_name: ${{ github.repository }}/${{ env.SERVICE_NAME }}
          image_tags: |
            latest,
            ${{ needs.set-version.outputs.version }},
            sha-${{ github.sha }}
          registry_url: ${{ env.REGISTRY }}
          registry_username: ${{ github.actor }}
          registry_password: ${{ secrets.GITHUB_TOKEN }}
          platforms: linux/amd64
          build_args: |
            {
              "VERSION": "${{ needs.set-version.outputs.version }}",
              "BUILD_NUMBER": "${{ github.run_number }}"
            }
          push: 'true'

  # =====================================================
  # CREATE GITHUB RELEASE
  # =====================================================
  create-release:
    name: Create GitHub Release
    needs: [set-version, build-dotnet, build-docker]
    runs-on: [self-hosted, Linux, X64]
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Create release
        uses: ./.github/actions/create-github-release
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          tag_name: ${{ needs.set-version.outputs.version_tag }}
          release_name: ${{ env.SERVICE_NAME }} ${{ needs.set-version.outputs.version }}
          body: ${{ needs.set-version.outputs.changelog }}
          artifacts: ${{ env.SERVICE_NAME }}-${{ needs.set-version.outputs.version }}-${{ github.run_number }}
          docker_images: |
            [
              "${{ env.REGISTRY }}/${{ github.repository }}/${{ env.SERVICE_NAME }}:${{ needs.set-version.outputs.version }}",
              "${{ env.REGISTRY }}/${{ github.repository }}/${{ env.SERVICE_NAME }}:latest"
            ]
          prerelease: ${{ contains(needs.set-version.outputs.version, '-') }}

  # =====================================================
  # SUMMARY
  # =====================================================
  summary:
    name: Build Summary
    needs: [set-version, build-dotnet, build-docker, create-release]
    if: always()
    runs-on: [self-hosted, Linux, X64]

    steps:
      - name: Generate summary
        run: |
          echo "## 🔨 Build Complete" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Service:** ${{ env.SERVICE_NAME }}" >> $GITHUB_STEP_SUMMARY
          echo "**Version:** ${{ needs.set-version.outputs.version }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY

          if [ "${{ needs.build-dotnet.result }}" = "success" ]; then
            echo "✅ .NET Build: Success" >> $GITHUB_STEP_SUMMARY
          else
            echo "❌ .NET Build: Failed" >> $GITHUB_STEP_SUMMARY
          fi

          if [ "${{ needs.build-docker.result }}" = "success" ]; then
            echo "✅ Docker Build: Success" >> $GITHUB_STEP_SUMMARY
          else
            echo "❌ Docker Build: Failed" >> $GITHUB_STEP_SUMMARY
          fi

          if [ "${{ needs.create-release.result }}" = "success" ]; then
            echo "✅ GitHub Release: Created" >> $GITHUB_STEP_SUMMARY
          elif [ "${{ needs.create-release.result }}" = "skipped" ]; then
            echo "⊘ GitHub Release: Skipped (not main branch)" >> $GITHUB_STEP_SUMMARY
          else
            echo "❌ GitHub Release: Failed" >> $GITHUB_STEP_SUMMARY
          fi
```

### Esempio Completo Deployment Workflow: Factory Realtime Service

```yaml
name: Deploy - Factory Realtime Service

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - factory_staging
          - factory_prod
        default: factory_staging

      deployment_type:
        description: 'Deployment type'
        required: true
        type: choice
        options:
          - systemd
          - docker
        default: docker

      configure_nginx:
        description: 'Configure Nginx after deployment'
        required: false
        type: boolean
        default: true

      ssl_mode:
        description: 'SSL mode'
        required: true
        type: choice
        options:
          - auto
          - renew
          - disable
          - self-signed
        default: self-signed

env:
  SERVICE_NAME: factory-realtime-service
  SERVICE_PORT: 8894

jobs:
  prepare:
    name: Prepare Deployment
    runs-on: [self-hosted, Linux, X64]
    outputs:
      environment: ${{ github.event.inputs.environment }}
      deployment_type: ${{ github.event.inputs.deployment_type }}
      configure_nginx: ${{ github.event.inputs.configure_nginx }}
    steps:
      - run: echo "Preparing deployment..."

  build:
    name: Build Application
    runs-on: [self-hosted, Linux, X64]
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '7.0.x'

      - name: Build and publish
        run: |
          dotnet restore
          dotnet publish -c Release -o ./publish

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: factory-realtime-service-build
          path: ./publish

  deploy-systemd:
    name: Deploy Systemd Service
    needs: [prepare, build]
    if: needs.prepare.outputs.deployment_type == 'systemd'
    runs-on: [self-hosted, Linux, X64]
    environment: ${{ needs.prepare.outputs.environment }}

    steps:
      - name: Deploy with systemd
        uses: ./.github/actions/deploy-systemd
        with:
          ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
          ssh_host: ${{ vars.DEPLOY_HOST }}
          ssh_user: ${{ vars.SSH_DEPLOY_USER }}
          service_name: ${{ env.SERVICE_NAME }}
          deploy_path: /opt/${{ env.SERVICE_NAME }}
          artifact_name: factory-realtime-service-build
          appsettings_json: ${{ secrets.APPSETTINGS_JSON }}
          service_port: ${{ env.SERVICE_PORT }}
          dotnet_version: '7.0'

  deploy-docker:
    name: Deploy Docker Container
    needs: [prepare]
    if: needs.prepare.outputs.deployment_type == 'docker'
    runs-on: [self-hosted, Linux, X64]
    environment: ${{ needs.prepare.outputs.environment }}

    steps:
      - name: Deploy with Docker
        uses: ./.github/actions/deploy-docker
        with:
          ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
          ssh_host: ${{ vars.DOCKER_HOST }}
          ssh_user: ${{ vars.SSH_DEPLOY_USER }}
          docker_image: ghcr.io/${{ github.repository }}/factory-realtime-service:latest
          container_name: ${{ env.SERVICE_NAME }}
          host_port: ${{ env.SERVICE_PORT }}
          container_port: 80
          registry_url: ghcr.io
          registry_username: ${{ github.actor }}
          registry_password: ${{ secrets.GITHUB_TOKEN }}
          environment_vars: |
            {
              "ASPNETCORE_ENVIRONMENT": "Production",
              "ConnectionStrings__Default": "${{ secrets.DB_CONNECTION_STRING }}"
            }

  configure-nginx:
    name: Configure Nginx
    needs: [prepare, deploy-systemd, deploy-docker]
    if: |
      always() &&
      needs.prepare.outputs.configure_nginx == 'true' &&
      (needs.deploy-systemd.result == 'success' || needs.deploy-docker.result == 'success')
    runs-on: [self-hosted, Linux, X64]
    environment: ${{ needs.prepare.outputs.environment }}

    steps:
      - name: Configure Nginx and SSL
        uses: ./.github/actions/deploy-nginx
        with:
          ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
          ssh_host: ${{ vars.NGINX_HOST }}
          ssh_user: ${{ vars.SSH_DEPLOY_USER }}
          service_name: ${{ env.SERVICE_NAME }}
          backend_host: localhost
          backend_port: ${{ env.SERVICE_PORT }}
          domain: ${{ vars.FQDN_DOMAIN }}
          ssl_mode: ${{ github.event.inputs.ssl_mode }}
          self_signed_passphrase: ${{ secrets.SSL_PASSPHRASE }}
          force_ssl: 'true'

  summary:
    name: Deployment Summary
    needs: [prepare, deploy-systemd, deploy-docker, configure-nginx]
    if: always()
    runs-on: [self-hosted, Linux, X64]
    steps:
      - name: Generate summary
        run: |
          echo "## 🚀 Deployment Complete" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Service:** ${{ env.SERVICE_NAME }}" >> $GITHUB_STEP_SUMMARY
          echo "**Environment:** ${{ needs.prepare.outputs.environment }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY

          if [ "${{ needs.deploy-systemd.result }}" = "success" ]; then
            echo "✅ Systemd deployment: Success" >> $GITHUB_STEP_SUMMARY
          elif [ "${{ needs.deploy-docker.result }}" = "success" ]; then
            echo "✅ Docker deployment: Success" >> $GITHUB_STEP_SUMMARY
          fi

          if [ "${{ needs.configure-nginx.result }}" = "success" ]; then
            echo "✅ Nginx configuration: Success" >> $GITHUB_STEP_SUMMARY
          fi
```

### Esempio: Solo Nginx (Nginx-only Mode)

```yaml
jobs:
  configure-nginx-only:
    name: Update Nginx Configuration
    runs-on: [self-hosted, Linux, X64]
    environment: production

    steps:
      - name: Configure Nginx
        uses: ./.github/actions/deploy-nginx
        with:
          ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
          ssh_host: ${{ vars.NGINX_HOST }}
          ssh_user: ${{ vars.SSH_DEPLOY_USER }}
          service_name: my-service
          backend_host: localhost
          backend_port: 8080
          domain: api.example.com
          ssl_mode: renew  # Force certificate renewal
          force_reconfigure: 'true'  # Force complete reconfiguration
```

### Esempio: Deploy Docker con Volume Mounts

```yaml
steps:
  - name: Deploy with Docker
    uses: ./.github/actions/deploy-docker
    with:
      ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
      ssh_host: ${{ vars.DOCKER_HOST }}
      ssh_user: ${{ vars.SSH_DEPLOY_USER }}
      docker_image: ghcr.io/myorg/myapp:v1.2.3
      container_name: myapp
      host_port: 8080
      container_port: 80
      registry_url: ghcr.io
      registry_username: ${{ github.actor }}
      registry_password: ${{ secrets.GITHUB_TOKEN }}
      environment_vars: |
        {
          "DATABASE_URL": "${{ secrets.DATABASE_URL }}",
          "API_KEY": "${{ secrets.API_KEY }}",
          "LOG_LEVEL": "info"
        }
      volume_mounts: |
        [
          "/opt/myapp/config:/app/config:ro",
          "/var/log/myapp:/app/logs",
          "/opt/myapp/data:/app/data"
        ]
      network_mode: bridge
      restart_policy: unless-stopped
```

## 🔄 Migrazione dei Workflow Esistenti

### Build Workflow: Prima (monolitico)

```yaml
# auth-server-build.yml - 644 linee di codice duplicato
jobs:
  set-version:
    steps:
      # 50+ linee per cleanup tag e versioning
      # 30+ linee per standard-version
      # 20+ linee per changelog extraction

  build-dotnet:
    steps:
      # 80+ linee per NuGet configuration (5 sources)
      # 40+ linee per ABP CLI installation
      # 30+ linee per npm/Node.js setup
      # 50+ linee per build/test/publish
      # 25+ linee per artifact creation

  build-docker:
    steps:
      # 60+ linee per Docker buildx setup
      # 40+ linee per multi-platform builds
      # 30+ linee per registry login e push

  create-release:
    steps:
      # 70+ linee per artifact download
      # 50+ linee per release creation
      # 40+ linee per asset upload
```

### Build Workflow: Dopo (con composite actions)

```yaml
# factory-service-build.yml - ~150 linee, logica riutilizzabile
jobs:
  set-version:
    steps:
      - uses: ./.github/actions/version-management
        with:
          release_as: patch

  build-dotnet:
    steps:
      - uses: ./.github/actions/build-dotnet
        with:
          version: ${{ needs.set-version.outputs.version }}
          install_abp_cli: 'true'
          # ... credentials

  build-docker:
    steps:
      - uses: ./.github/actions/build-docker-image
        with:
          image_tags: latest,${{ needs.set-version.outputs.version }}

  create-release:
    steps:
      - uses: ./.github/actions/create-github-release
        with:
          tag_name: ${{ needs.set-version.outputs.version_tag }}
```

**Risultato**: Da ~644 linee a ~150 linee (-77% di codice)

### Deployment Workflow: Prima (monolitico)

```yaml
# 1500+ linee di codice duplicato
jobs:
  deploy:
    steps:
      # 100+ linee per SSH setup
      # 200+ linee per systemd deployment
      # 300+ linee per Nginx configuration
      # 150+ linee per SSL management
      # ...
```

### Deployment Workflow: Dopo (con composite actions)

```yaml
# 200-300 linee, logica riutilizzabile
jobs:
  deploy-systemd:
    steps:
      - uses: ./.github/actions/deploy-systemd
        with: { ... }

  configure-nginx:
    steps:
      - uses: ./.github/actions/deploy-nginx
        with: { ... }
```

**Risultato**: Da ~1500 linee a ~250 linee (-83% di codice)

### Checklist di Migrazione Build Workflows

- [ ] Identificare parametri specifici del servizio (nome, versione .NET, progetto)
- [ ] Verificare necessità ABP CLI (install_abp_cli: true/false)
- [ ] Configurare secrets NuGet (ABP, Telerik, DevExpress, GitHub Packages)
- [ ] Sostituire job `set-version` con `version-management` action
- [ ] Sostituire job `build-dotnet` con `build-dotnet` action
- [ ] Sostituire job `build-docker` con `build-docker-image` action
- [ ] Sostituire job `create-release` con `create-github-release` action
- [ ] Testare build completo con tutti i job
- [ ] Verificare artifact e Docker images
- [ ] Verificare GitHub release creation

### Checklist di Migrazione Deployment Workflows

- [ ] Identificare parametri specifici del servizio (nome, porte, paths)
- [ ] Estrarre secrets e variables in GitHub Environment
- [ ] Sostituire job `deploy` con `deploy-systemd` o `deploy-docker`
- [ ] Sostituire job inline Nginx con `configure-nginx`
- [ ] Testare in staging prima di produzione
- [ ] Aggiornare documentazione del servizio

## 🔧 Troubleshooting

### Build Issues

#### Problema: NuGet Package Restore Failed

**Sintomo:** Errore durante il restore dei pacchetti NuGet

**Soluzione:**
```bash
# Verificare configurazione NuGet.Config
cat NuGet.Config

# Testare connessione alle source
dotnet nuget list source --configfile NuGet.Config

# Verificare credentials
echo ${{ secrets.ABP_NUGET_API_KEY }} | wc -c  # Should be > 0

# Test restore manuale
dotnet restore --configfile NuGet.Config --verbosity detailed
```

#### Problema: ABP CLI Installation Failed

**Sintomo:** ABP CLI non viene installato o non è disponibile

**Soluzione:**
```bash
# Verificare installazione
dotnet tool list -g | grep volo.abp.studio.cli

# Installare manualmente
dotnet tool install -g Volo.Abp.Studio.Cli

# Verificare PATH
echo $PATH | grep .dotnet/tools

# Aggiungere al PATH se necessario
export PATH="$PATH:$HOME/.dotnet/tools"
```

#### Problema: Docker Build Multi-Platform Failed

**Sintomo:** Build multi-platform fallisce

**Soluzione:**
```bash
# Verificare Docker Buildx
docker buildx version

# Verificare builder
docker buildx ls

# Creare nuovo builder se necessario
docker buildx create --name multiplatform --use

# Verificare piattaforme supportate
docker buildx inspect --bootstrap
```

#### Problema: Version Tag Already Exists

**Sintomo:** standard-version fallisce perché il tag esiste già

**Soluzione:**
```bash
# Lista tag locali
git tag -l

# Lista tag remoti
git ls-remote --tags origin

# Rimuovere tag locale (se non pushato)
git tag -d v1.0.0

# Rimuovere tag remoto (ATTENZIONE!)
git push --delete origin v1.0.0

# Usare cleanup_unpushed_tags: 'true' nell'action
```

### Deployment Issues

#### Problema: SSH Connection Failed

**Sintomo:** Errore di connessione SSH durante deployment

**Soluzione:**
```bash
# Verificare SSH key
ssh-keygen -y -f ~/.ssh/deploy_key

# Testare connessione manualmente
ssh -i ~/.ssh/deploy_key user@host

# Verificare known_hosts
ssh-keyscan -H hostname >> ~/.ssh/known_hosts
```

### Problema: Health Check Failed

**Sintomo:** Health check fallisce dopo deployment

**Soluzione:**
1. Aumentare retry count/delay se il servizio è lento ad avviarsi
2. Verificare che il servizio sia in ascolto sulla porta corretta
3. Controllare logs del servizio:
   ```bash
   # Systemd
   sudo journalctl -u service-name -n 100

   # Docker
   sudo docker logs container-name --tail 100
   ```

### Problema: SSL Certificate Renewal Failed

**Sintomo:** Errore durante il renewal del certificato

**Soluzione:**

**Let's Encrypt:**
```bash
# Verificare DNS
dig domain.com

# Test manuale renewal
sudo certbot renew --dry-run

# Check logs
sudo tail -f /var/log/letsencrypt/letsencrypt.log
```

**Self-signed:**
```bash
# Verificare certificato esistente
openssl pkcs12 -in cert.pfx -passin pass:'passphrase' -nokeys | openssl x509 -noout -dates

# Rigenerare manualmente
cd /opt/service-name/certs
sudo /opt/service-name/scripts/renew-cert.sh
```

### Problema: Nginx Configuration Test Failed

**Sintomo:** `nginx -t` fallisce dopo configurazione

**Soluzione:**
```bash
# Test configurazione
sudo nginx -t

# Visualizzare configurazione completa
sudo nginx -T

# Verificare sintassi file specifico
sudo nginx -t -c /etc/nginx/sites-available/service-name

# Restore backup se necessario
sudo cp /etc/nginx/sites-available/service-name.backup.YYYYMMDD_HHMMSS /etc/nginx/sites-available/service-name
sudo systemctl reload nginx
```

### Problema: Docker Container Not Starting

**Sintomo:** Container si ferma subito dopo l'avvio

**Soluzione:**
```bash
# Controllare logs
sudo docker logs container-name

# Controllare stato
sudo docker inspect container-name

# Verificare immagine
sudo docker images
sudo docker pull image:tag

# Test manuale
sudo docker run -it --rm image:tag /bin/sh
```

## 📖 Riferimenti

- [Documentazione Architettura](../../docs/workflows/COMPOSITE_ACTIONS_ARCHITECTURE.md)
- [Guida Standardizzazione Nginx/SSL](../../docs/workflows/NGINX_SSL_STANDARDIZATION_GUIDE.md)
- [GitHub Actions - Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

## 🤝 Contribuire

Per miglioramenti alle composite actions:

1. Testare modifiche localmente
2. Aggiornare esempi in questo README
3. Aggiornare documentazione relativa
4. Testare in staging
5. Creare PR con descrizione dettagliata

---

**Ultima modifica:** 2025-10-17
**Maintainer:** DiemmeGroup DevOps Team
