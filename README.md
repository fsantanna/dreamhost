# dreamhost

Deploy automático via GitHub Actions para DreamHost com dois ambientes.

## Como funciona

```
push master → testes → passam? → deploy em dev.meusite.com
                           ↓ falham
                      ❌ nada acontece

git tag v1.0.0 → testes → passam? → deploy em meusite.com (produção)
                              ↓ falham
                         ❌ nada acontece
```

| Evento | Testes | Deploy |
|--------|--------|--------|
| PR para master | Roda | Nenhum |
| Push no master | Roda | Dev |
| Tag `v*` | Roda | Produção |

## Setup

### 1. Criar subdomínio dev no DreamHost

No painel do DreamHost, adicione o domínio `dev.meusite.com` apontando para
um diretório separado (ex: `~/dev.meusite.com/`).

### 2. Gerar chave SSH

```bash
ssh-keygen -t ed25519 -f dreamhost_deploy -C "github-actions-deploy" -N ""
```

### 3. Adicionar chave pública no DreamHost

```bash
ssh-copy-id -i dreamhost_deploy.pub usuario@servidor.dreamhost.com
```

### 4. Configurar GitHub Secrets

Em **Settings > Secrets and variables > Actions**, adicione:

| Secret | Valor | Exemplo |
|--------|-------|---------|
| `DREAMHOST_SSH_KEY` | Chave privada (conteúdo de `dreamhost_deploy`) | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `DREAMHOST_HOST` | Hostname do servidor | `servidor.dreamhost.com` |
| `DREAMHOST_USER` | Usuário SSH | `meuusuario` |
| `DREAMHOST_DEV_PATH` | Diretório dev | `~/dev.meusite.com/` |
| `DREAMHOST_PRODUCTION_PATH` | Diretório de produção | `~/meusite.com/` |

### 5. Adaptar os testes

Edite `.github/workflows/deploy.yml` e descomente/configure a seção de testes.

## Uso no dia a dia

```bash
# Desenvolver e testar em dev
git add . && git commit -m "nova feature"
git push                          # → dev

# Verificar em dev.meusite.com ...

# Promover para produção
git tag v1.0.0
git push origin v1.0.0            # → produção
```

### Convenção de tags (semver)

- **Patch** (bugfix): `v1.0.0` → `v1.0.1`
- **Minor** (feature): `v1.0.0` → `v1.1.0`
- **Major** (breaking): `v1.0.0` → `v2.0.0`
