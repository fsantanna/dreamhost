# dreamhost

Deploy automático via GitHub Actions para DreamHost.

## Como funciona

1. Push no branch `master` dispara o workflow
2. GitHub Actions roda os testes
3. Se os testes passam, faz deploy via rsync/SSH para o DreamHost
4. Pull requests rodam os testes mas **não** fazem deploy

## Setup

### 1. Gerar chave SSH

```bash
ssh-keygen -t ed25519 -f dreamhost_deploy -C "github-actions-deploy" -N ""
```

### 2. Adicionar chave pública no DreamHost

Copie o conteúdo de `dreamhost_deploy.pub` para o DreamHost:

```bash
ssh-copy-id -i dreamhost_deploy.pub usuario@servidor.dreamhost.com
```

Ou adicione manualmente em `~/.ssh/authorized_keys` no servidor.

### 3. Configurar GitHub Secrets

No repositório GitHub, vá em **Settings > Secrets and variables > Actions** e adicione:

| Secret | Valor | Exemplo |
|--------|-------|---------|
| `DREAMHOST_SSH_KEY` | Conteúdo da chave privada (`dreamhost_deploy`) | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `DREAMHOST_HOST` | Hostname do servidor DreamHost | `servidor.dreamhost.com` |
| `DREAMHOST_USER` | Usuário SSH no DreamHost | `meuusuario` |
| `DREAMHOST_PATH` | Caminho do site no servidor | `~/meusite.com/` |

### 4. Adaptar os testes

Edite `.github/workflows/deploy.yml` e descomente/configure a seção de testes conforme sua stack (Node.js, PHP, Python, etc.).

## Estrutura do workflow

```
push master → [test] → testes passam? → [deploy] → rsync para DreamHost
                              ↓ falham
                         ❌ deploy não acontece
```
