# Version Management Action - Troubleshooting

## Common Issues

### ❌ "No version bump needed" ma in locale funziona

**Causa**: GitHub Actions fa un shallow clone (fetch-depth: 1) che non include la history completa necessaria a standard-version per analizzare i conventional commits.

**Soluzione**: L'action ora fetcha automaticamente la history completa con:
```bash
git fetch --unshallow --tags
```

Se il problema persiste, assicurati che nel workflow che usa questa action ci sia:
```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0  # Fetch all history for standard-version
```

### 🔍 Debug: Vedere l'output di standard-version

L'action ora mostra l'output completo di `--dry-run`:

```
🔍 Checking if version will change...
Running: npx standard-version --dry-run
📋 Dry-run output:
[output completo di standard-version]
```

Controlla nei logs dell'action per vedere cosa ha trovato standard-version.

### 📜 Verificare i commit

L'action ora mostra gli ultimi 5 commits:

```
📜 Recent commits:
abc1234 fix: correzione bug
def5678 feat: nuova funzionalità
```

Verifica che i commit seguano il formato Conventional Commits:
- `feat:` per nuove funzionalità (minor bump)
- `fix:` per bug fix (patch bump)
- `BREAKING CHANGE:` per breaking changes (major bump)

### ⚙️ npx non funziona

**Causa**: Node.js non è configurato nel workflow.

**Soluzione**: L'action ora configura sempre Node.js:
```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: ${{ inputs.node_version }}  # default: 20.x
```

### 📦 Progetti senza package.json

**Soluzione**: L'action ora crea automaticamente un package.json minimo:
```json
{
  "name": "nome-progetto",
  "version": "1.0.0",
  "private": true
}
```

Il nome viene estratto dalla directory corrente.

## Best Practices

### ✅ Configurazione workflow consigliata

```yaml
name: Release

on:
  push:
    branches: [main, master]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # IMPORTANTE: history completa
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Version Management
        uses: dm-italy/github-dm-italy/actions/version-management@main
        with:
          working_directory: '.'
          git_branch: 'main'
```

### 🎯 Conventional Commits

Standard-version analizza i commit per decidere il bump della versione:

| Tipo commit | Bump versione | Esempio |
|------------|---------------|---------|
| `fix:` | patch (0.0.X) | `fix: corretto bug login` |
| `feat:` | minor (0.X.0) | `feat: aggiunto dark mode` |
| `BREAKING CHANGE:` | major (X.0.0) | `feat!: cambio API` |
| `chore:`, `docs:` | nessuno | `chore: update deps` |

### 🏷️ Tag e Prefix

Per monorepo con prefix personalizzati:

```yaml
- name: Version Management - Factory
  uses: dm-italy/github-dm-italy/actions/version-management@main
  with:
    working_directory: 'apps/factory'
    tag_prefix: 'factory@'  # Genera tag: factory@1.2.3
```

## Logs e Debug

### Cosa cercare nei logs

1. **Git History**: Verifica che mostri più di 1 commit
   ```
   📜 Recent commits:
   abc1234 fix: bug
   def5678 feat: feature
   ```

2. **Dry-run Output**: Deve contenere "bumping version"
   ```
   📋 Dry-run output:
   ✔ bumping version in package.json from 1.0.0 to 1.1.0
   ```

3. **Version Changed**: Deve essere `true`
   ```
   version_changed=true
   new_version=1.1.0
   ```

### Script di test locale

Per testare in locale prima del push:

```bash
# Simula quello che fa l'action
git fetch --tags
npx standard-version --dry-run

# Se va bene, esegui
npx standard-version --skip.tag
```

## Errori comuni

### Error: rawValue: command not found / MachineDeltaDataEto: command not found

**Causa**: Il changelog contiene caratteri speciali che bash cerca di eseguire come comandi.

**Soluzione**: ✅ Risolto dalla versione corrente. L'action ora usa heredoc con quote per prevenire command injection.

Se usi una versione vecchia, aggiorna a:
```yaml
uses: dm-italy/github-dm-italy/actions/version-management@main
```

### Error: No commits found

**Causa**: Repository vuoto o nessun commit dopo l'ultimo tag.

**Soluzione**: Assicurati di avere commit con conventional format dopo l'ultimo tag.

### Error: ENOENT package.json

**Non dovrebbe più accadere**: L'action ora crea automaticamente package.json se mancante.

### Error: shallow update not allowed

**Causa**: Tentativo di push con shallow clone.

**Soluzione**: L'action ora fa `git fetch --unshallow` automaticamente.

## Supporto

Per problemi non risolti:
1. Controlla i logs completi dell'action
2. Verifica la configurazione del workflow
3. Testa in locale con `npx standard-version --dry-run`
4. Apri un issue con i logs completi
