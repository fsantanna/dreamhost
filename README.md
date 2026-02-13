# dreamhost

[![Web](https://github.com/fsantanna/dreamhost/actions/workflows/deploy.yml/badge.svg)](https://github.com/fsantanna/dreamhost/actions/workflows/deploy.yml)

Deploy automático via GitHub Actions para DreamHost com dois ambientes.
Reutilizável por múltiplos repos — cada um define seu `APP_DIR`.

## Como funciona

| Evento | Testes | Deploy |
|--------|--------|--------|
| PR para main | Roda | Nenhum |
| Push no main | Roda | `dev.meusite.com/APP_DIR/` |
| Tag `v*` | Roda | `meusite.com/APP_DIR/` |

## Arquitetura multi-app

Os secrets/variables do DreamHost são compartilhados.
Cada repo define apenas o `APP_DIR` no workflow:

```
dev.ceu-lang.org/
├── dh/          ← repo dreamhost  (APP_DIR: dh)
├── blog/        ← repo blog       (APP_DIR: blog)
└── api/         ← repo api        (APP_DIR: api)
```

## Como usar em um novo repo

1. Copiar `.github/workflows/deploy.yml` para o novo repo
2. Mudar `APP_DIR` no topo do workflow
3. Colocar os arquivos do site em `web/`
4. Configurar os mesmos secrets/variables (ou usar Organization-level)

## Setup inicial (uma vez)

### 1. Criar subdomínio dev no DreamHost

No painel do DreamHost, adicione `dev.meusite.com` apontando para
um diretório separado (ex: `~/dev.meusite.com/`).

### 2. Gerar chave SSH

```bash
ssh-keygen -t ed25519 -f dreamhost_deploy -C "github-actions-deploy" -N ""
```

### 3. Adicionar chave pública no DreamHost

```bash
ssh-copy-id -i dreamhost_deploy.pub usuario@servidor.dreamhost.com
```

### 4. Configurar GitHub Secrets e Variables

Em **Settings > Secrets and variables > Actions**:

| Tipo     | Nome         | Valor                              |
|----------|--------------|------------------------------------|
| Secret   | `DH_SSH_KEY` | Chave privada (`dreamhost_deploy`) |
| Variable | `DH_HOST`    | Hostname do servidor               |
| Variable | `DH_USER`    | Usuário SSH                        |
| Variable | `DH_DIR_DEV` | Diretório raiz do domínio dev      |
| Variable | `DH_DIR_PRO` | Diretório raiz do domínio produção |

## Uso no dia a dia

```bash
# Desenvolver e testar em dev
git add . && git commit -m "nova feature"
git push                          # → dev.meusite.com/APP_DIR/

# Promover para produção
git tag v1.0.0
git push origin v1.0.0            # → meusite.com/APP_DIR/
```

### Convenção de tags (semver)

- **Patch** (bugfix): `v1.0.0` → `v1.0.1`
- **Minor** (feature): `v1.0.0` → `v1.1.0`
- **Major** (breaking): `v1.0.0` → `v2.0.0`
