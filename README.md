# dreamhost

[![Web](https://github.com/fsantanna/dreamhost/actions/workflows/deploy.yml/badge.svg)](https://github.com/fsantanna/dreamhost/actions/workflows/deploy.yml)

Deploy automático via GitHub Actions para DreamHost com dois ambientes.

## Como funciona

```
push master → testes → passam? → deploy em dev.meusite.com/dh/
                           ↓ falham
                      ❌ nada acontece

git tag v1.0.0 → testes → passam? → deploy em meusite.com/dh/
                              ↓ falham
                         ❌ nada acontece
```

| Evento | Testes | Deploy |
|--------|--------|--------|
| PR para master | Roda | Nenhum |
| Push no master | Roda | `dev.meusite.com/dh/` |
| Tag `v*` | Roda | `meusite.com/dh/` |

O `APP_DIR` é definido no topo do workflow (`dh` neste caso). Cada repo pode
ter seu próprio `APP_DIR`, enquanto os secrets do DreamHost são compartilhados.

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
| `DREAMHOST_DEV_DIR` | Diretório raiz do domínio dev | `~/dev.meusite.com` |
| `DREAMHOST_PRODUCTION_DIR` | Diretório raiz do domínio de produção | `~/meusite.com` |

Os secrets apontam para a **raiz do domínio**. Cada repo define seu `APP_DIR`
no workflow, e o deploy vai para `{DIR}/{APP_DIR}/`.

### 5. Adaptar os testes

Edite `.github/workflows/deploy.yml` e descomente/configure a seção de testes.

## Uso no dia a dia

```bash
# Desenvolver e testar em dev
git add . && git commit -m "nova feature"
git push                          # → dev.meusite.com/dh/

# Verificar em dev.meusite.com/dh/ ...

# Promover para produção
git tag v1.0.0
git push origin v1.0.0            # → meusite.com/dh/
```

### Convenção de tags (semver)

- **Patch** (bugfix): `v1.0.0` → `v1.0.1`
- **Minor** (feature): `v1.0.0` → `v1.1.0`
- **Major** (breaking): `v1.0.0` → `v2.0.0`
