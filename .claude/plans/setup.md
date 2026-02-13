# Plano: Deploy automático para DreamHost via GitHub Actions

## Objetivo

Configurar deploy automático do site para DreamHost via SSH usando GitHub Actions.
- Push em `main` → deploy para **produção**
- Push em `dev` → deploy para **desenvolvimento**

---

## 1. Gerar par de chaves SSH (na máquina local)

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/dreamhost_deploy
```

Isso gera dois arquivos:
- `~/.ssh/dreamhost_deploy` — chave **privada** (vai pro GitHub Secret)
- `~/.ssh/dreamhost_deploy.pub` — chave **pública** (vai pro DreamHost)

---

## 2. Adicionar chave pública no DreamHost

Copiar o conteúdo da chave pública para o servidor:

```bash
ssh-copy-id -i ~/.ssh/dreamhost_deploy.pub usuario@servidor.dreamhost.com
```

Ou manualmente adicionar o conteúdo de `dreamhost_deploy.pub` no arquivo
`~/.ssh/authorized_keys` do servidor DreamHost.

---

## 3. Configurar GitHub — Secrets e Variables

### Secrets (Settings → Secrets and variables → Actions → Secrets)

| Secret                | Valor                                      |
|-----------------------|--------------------------------------------|
| `DREAMHOST_SSH_KEY`   | Conteúdo da chave privada (`dreamhost_deploy`) |

### Variables (Settings → Secrets and variables → Actions → Variables)

| Variable                   | Valor                                        |
|----------------------------|----------------------------------------------|
| `DREAMHOST_HOST`           | `servidor.dreamhost.com`                     |
| `DREAMHOST_USER`           | `usuario_ssh`                                |
| `DREAMHOST_DEV_DIR`        | `/home/usuario/dev.seusite.com`              |
| `DREAMHOST_PRODUCTION_DIR` | `/home/usuario/seusite.com`                  |

> **Nota:** Só a chave SSH precisa ser Secret (é sensível). Os outros valores
> são configuração e podem ficar em Variables (visíveis, mais fáceis de editar).

---

## 4. Criar o workflow do GitHub Actions

Arquivo: `.github/workflows/deploy.yml`

```yaml
name: Deploy to DreamHost

on:
  push:
    branches: [main, dev]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.DREAMHOST_SSH_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H ${{ vars.DREAMHOST_HOST }} >> ~/.ssh/known_hosts

      - name: Set deploy directory
        id: target
        run: |
          if [ "${{ github.ref_name }}" = "main" ]; then
            echo "dir=${{ vars.DREAMHOST_PRODUCTION_DIR }}" >> "$GITHUB_OUTPUT"
          else
            echo "dir=${{ vars.DREAMHOST_DEV_DIR }}" >> "$GITHUB_OUTPUT"
          fi

      - name: Deploy via rsync
        run: |
          rsync -avz --delete \
            -e "ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no" \
            --exclude '.git' \
            --exclude '.github' \
            --exclude '.claude' \
            ./ ${{ vars.DREAMHOST_USER }}@${{ vars.DREAMHOST_HOST }}:${{ steps.target.outputs.dir }}/
```

---

## 5. Checklist de execução

- [ ] Gerar par de chaves SSH (ed25519)
- [ ] Adicionar chave pública no `authorized_keys` do DreamHost
- [ ] Testar conexão SSH manualmente: `ssh -i ~/.ssh/dreamhost_deploy usuario@servidor.dreamhost.com`
- [ ] Criar Secret `DREAMHOST_SSH_KEY` no GitHub (conteúdo da chave privada)
- [ ] Criar Variables no GitHub: `DREAMHOST_HOST`, `DREAMHOST_USER`, `DREAMHOST_DEV_DIR`, `DREAMHOST_PRODUCTION_DIR`
- [ ] Criar arquivo `.github/workflows/deploy.yml` no repositório
- [ ] Fazer push e verificar se o Action executa com sucesso
- [ ] Verificar se os arquivos chegaram no servidor DreamHost

---

## Notas

- O `rsync --delete` remove do servidor arquivos que não existem mais no repo.
  Se não quiser esse comportamento, remover a flag `--delete`.
- Os diretórios `.git`, `.github` e `.claude` são excluídos do deploy.
- Para adicionar mais exclusões, adicionar linhas `--exclude 'pasta'` no rsync.
