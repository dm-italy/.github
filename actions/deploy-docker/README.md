# Deploy Docker Container - Composite Action

GitHub Actions composite action per il deployment di container Docker su server remoti tramite SSH con supporto per docker-compose, gestione avanzata della rete e normalizzazione automatica dei path delle immagini.

## Caratteristiche Principali

✅ **Deploy con docker-compose** - Configurazione dichiarativa e riproducibile
✅ **Normalizzazione Automatica Path** - Risolve automaticamente duplicazioni nel path delle immagini
✅ **Gestione Reti Docker** - Creazione automatica di reti Docker per comunicazione inter-container
✅ **Health Check Integrati** - Verifica automatica dello stato del container dopo il deploy
✅ **Cleanup Automatico** - Rimozione immagini obsolete e dangling
✅ **Variabili d'Ambiente** - Configurazione flessibile tramite JSON
✅ **Volume Mounting** - Supporto per volumi persistenti e bind mounts
✅ **Rollback Automatico** - Ripristino automatico in caso di failure

## Utilizzo Base

```yaml
- name: Deploy Docker Container
  uses: ./.github/actions/deploy-docker
  with:
    ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
    ssh_host: ${{ vars.SSH_HOST }}
    ssh_user: ${{ vars.SSH_USER }}
    docker_image: ghcr.io/dm-italy/myservice:1.0.0
    container_name: myservice
    host_port: 8080
    container_port: 80
    registry_username: ${{ github.actor }}
    registry_password: ${{ secrets.GITHUB_TOKEN }}
```

## Input Parameters

### Obbligatori

| Parameter | Descrizione | Esempio |
|-----------|-------------|---------|
| `ssh_key` | Chiave privata SSH per autenticazione | `${{ secrets.DEPLOY_SSH_KEY }}` |
| `ssh_host` | Hostname o IP del server target | `production.example.com` |
| `ssh_user` | Username SSH per il deployment | `deployer` |
| `docker_image` | Immagine Docker completa con tag | `ghcr.io/org/image:tag` |
| `container_name` | Nome del container Docker | `myservice` |
| `host_port` | Porta host per il port mapping | `8080` |
| `container_port` | Porta interna del container | `80` |

### Opzionali

| Parameter | Descrizione | Default | Esempio |
|-----------|-------------|---------|---------|
| `environment_vars` | Variabili d'ambiente (JSON) | `{}` | `{"KEY":"value"}` |
| `volume_mounts` | Mount dei volumi (JSON array) | `[]` | `["/host:/container:ro"]` |
| `registry_url` | URL del Docker registry | `ghcr.io` | `docker.io` |
| `registry_username` | Username per il registry | - | `${{ github.actor }}` |
| `registry_password` | Password per il registry | - | `${{ secrets.TOKEN }}` |
| `restart_policy` | Policy di restart del container | `unless-stopped` | `always` |
| `skip_health_check` | Salta l'health check | `false` | `true` |
| `health_endpoint` | Endpoint per l'health check | `/` | `/health` |
| `cleanup_old_images` | Rimuovi immagini vecchie | `true` | `false` |
| `deploy_path` | Path di deployment sul server | `/opt/docker/{container_name}` | `/srv/apps/myservice` |
| `docker_network` | Nome della rete Docker | `factory-network` | `services-net` |

## Output

| Output | Descrizione | Valori |
|--------|-------------|--------|
| `deployment_status` | Stato del deployment | `success`, `failed` |
| `container_id` | ID del container Docker | Es. `a1b2c3d4e5f6` |
| `health_check_passed` | Risultato dell'health check | `true`, `false` |
| `normalized_image` | Path normalizzato dell'immagine | Es. `ghcr.io/org/image:tag` |

## Docker Image Path Normalization

### Problema

Nei workflow GitHub Actions, le immagini Docker possono essere referenziate in due modi che a volte generano duplicazioni:

```yaml
# Metodo 1: github.repository (include owner/repo)
ghcr.io/${{ github.repository }}/service:tag
# → ghcr.io/dm-italy/microservices-1/service:tag

# Metodo 2: github.repository_owner (solo owner)
ghcr.io/${{ github.repository_owner }}/microservices/service:tag
# → ghcr.io/dm-italy/microservices/service:tag

# Problema: doppia interpolazione
ghcr.io/dm-italy/dm-italy/service:tag ❌
```

### Soluzione Automatica

L'action **normalizza automaticamente** il path rimuovendo segmenti consecutivi duplicati:

```bash
# Input (con duplicazione)
ghcr.io/dm-italy/dm-italy/repository:1.0.0

# Output (normalizzato automaticamente)
ghcr.io/dm-italy/repository:1.0.0 ✓
```

### Esempi di Normalizzazione

| Input | Output |
|-------|--------|
| `ghcr.io/dm-italy/dm-italy/repo:tag` | `ghcr.io/dm-italy/repo:tag` |
| `ghcr.io/owner/owner/path/service:tag` | `ghcr.io/owner/path/service:tag` |
| `ghcr.io/org/org/org/image:latest` | `ghcr.io/org/image:latest` |
| `ghcr.io/owner/repo:tag` | `ghcr.io/owner/repo:tag` *(già corretto)* |

### Logging

Quando viene applicata la normalizzazione, l'action logga:

```
⚠️  Docker image path normalized:
   Original:    ghcr.io/dm-italy/dm-italy/service:1.0.0
   Normalized:  ghcr.io/dm-italy/service:1.0.0
```

Se il path è già corretto:
```
✓ Docker image path: ghcr.io/dm-italy/service:1.0.0
```

## Esempi Avanzati

### Con Variabili d'Ambiente e Volumi

```yaml
- name: Deploy with environment and volumes
  uses: ./.github/actions/deploy-docker
  with:
    ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
    ssh_host: ${{ vars.SSH_HOST }}
    ssh_user: ${{ vars.SSH_USER }}
    docker_image: ghcr.io/dm-italy/api:1.0.0
    container_name: api-service
    host_port: 8080
    container_port: 80
    environment_vars: |
      {
        "ASPNETCORE_ENVIRONMENT": "Production",
        "ConnectionStrings__Default": "${{ secrets.DB_CONNECTION }}",
        "Redis__Configuration": "${{ secrets.REDIS_CONNECTION }}"
      }
    volume_mounts: |
      [
        "/data/logs:/app/logs:rw",
        "/data/uploads:/app/uploads:rw",
        "/etc/ssl/certs:/etc/ssl/certs:ro"
      ]
    docker_network: production-network
```

### Deploy con Health Check Personalizzato

```yaml
- name: Deploy with custom health check
  uses: ./.github/actions/deploy-docker
  with:
    ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
    ssh_host: ${{ vars.SSH_HOST }}
    ssh_user: ${{ vars.SSH_USER }}
    docker_image: ghcr.io/dm-italy/service:2.0.0
    container_name: health-service
    host_port: 9000
    container_port: 80
    health_endpoint: /api/health
    skip_health_check: false
```

### Deploy Senza Cleanup (Development)

```yaml
- name: Deploy without cleanup
  uses: ./.github/actions/deploy-docker
  with:
    ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
    ssh_host: ${{ vars.DEV_HOST }}
    ssh_user: ${{ vars.SSH_USER }}
    docker_image: ghcr.io/dm-italy/dev-service:latest
    container_name: dev-service
    host_port: 3000
    container_port: 80
    cleanup_old_images: false
    restart_policy: always
```

## Testing

### Test della Normalizzazione

```bash
# Esegui i test di normalizzazione
.github/actions/deploy-docker/test-normalize.sh

# Output atteso
Testing Docker image path normalization...

✓ PASS: ghcr.io/dm-italy/dm-italy/repository-name:1.0.0
  → ghcr.io/dm-italy/repository-name:1.0.0
✓ PASS: ghcr.io/dm-italy/repository-name:1.0.0
  → ghcr.io/dm-italy/repository-name:1.0.0
...
All tests passed! ✓
```

## Troubleshooting

### Problema: Health Check Failed

**Sintomo**: L'health check fallisce dopo il deploy
```
❌ Health check failed after 6 attempts
```

**Soluzioni**:
1. Verificare che l'endpoint sia corretto: `health_endpoint: /api/health`
2. Aumentare il timeout: aspettare più tempo per l'avvio del servizio
3. Controllare i log del container: `docker logs {container_name}`
4. Verificare la porta: `curl http://localhost:{host_port}{health_endpoint}`

### Problema: Duplicazione Path Immagine

**Sintomo**: Immagine non trovata con path duplicato
```
Error: Image not found: ghcr.io/dm-italy/dm-italy/service:1.0.0
```

**Soluzione**: Automatica! L'action normalizza il path. Verificare i log:
```
⚠️  Docker image path normalized:
   Original:    ghcr.io/dm-italy/dm-italy/service:1.0.0
   Normalized:  ghcr.io/dm-italy/service:1.0.0
```

### Problema: Permission Denied

**Sintomo**: Errori di permesso durante il deploy
```
Permission denied: cannot create directory /opt/docker/service
```

**Soluzioni**:
1. Verificare che l'utente SSH abbia permessi sudo
2. Aggiungere l'utente al gruppo docker: `sudo usermod -aG docker {ssh_user}`
3. Verificare i permessi della directory: `ls -la /opt/docker`

## Sicurezza

⚠️ **Best Practices**:

1. **Non esporre secrets nei log** - I valori sensibili vengono automaticamente mascherati
2. **Usare secrets GitHub** - Non hardcodare credenziali nei workflow
3. **Limitare permessi SSH** - Usare chiavi SSH dedicate per il deployment
4. **Validare inputs** - L'action valida automaticamente i parametri
5. **Usare registry privati** - Configurare autenticazione per registry privati

## Performance

### Ottimizzazioni Automatiche

- **Parallel execution**: Pull e cleanup eseguiti in parallelo quando possibile
- **Image caching**: Le immagini già presenti non vengono ri-scaricate
- **Minimal restarts**: Solo i container modificati vengono riavviati
- **Network reuse**: Le reti Docker vengono riutilizzate se esistenti

### Metriche Tipiche

- Deploy completo: ~30-60 secondi
- Pull immagine (nuova): ~10-30 secondi
- Pull immagine (cached): <5 secondi
- Health check: 10-30 secondi (6 tentativi con 5s di delay)

## Versioning

Questa action segue [Semantic Versioning](https://semver.org/):

- **Major**: Breaking changes (es. rimozione parametri)
- **Minor**: Nuove features backward-compatible
- **Patch**: Bug fixes e miglioramenti

## Changelog

### v1.1.0 (Current)
- ✨ Aggiunta normalizzazione automatica path immagini Docker
- 🐛 Fix gestione duplicazioni owner name (dm-italy/dm-italy)
- 📝 Documentazione estesa con esempi
- ✅ Aggiunto script di test per normalizzazione

### v1.0.0
- 🎉 Release iniziale
- ✨ Deploy con docker-compose
- ✨ Health check integrato
- ✨ Cleanup automatico immagini

## Contributing

Per contribuire a questa action:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests: `./test-normalize.sh`
5. Submit a pull request

## License

Copyright © 2025 DiemmeGroup
Internal use only.

## Support

Per supporto:
- 📖 Documentazione: `/docs/workflows/COMPOSITE_ACTIONS_ARCHITECTURE.md`
- 🐛 Issues: GitHub Issues
- 📧 Email: devops@diemmegroup.com
