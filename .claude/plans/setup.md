# Plano: Deploy automático para DreamHost via GitHub Actions

## Objetivo

Configurar deploy automático para DreamHost via SSH usando GitHub Actions.
Cada repositório (web app) copia o workflow e muda apenas o `APP_DIR`.

- Push em `main` → deploy para **desenvolvimento** (`dev.site.com/APP_DIR/`)
- Tag `v*` → deploy para **produção** (`site.com/APP_DIR/`)

---

## Arquitetura multi-app

```
Secrets/Variables (compartilhados)     Cada repo define
─────────────────────────────────     ─────────────────
DH_SSH_KEY  (secret)                  APP_DIR: dh
DH_HOST     (variable)               APP_DIR: blog
DH_USER     (variable)               APP_DIR: api
DH_DIR_DEV  (variable)               ...
DH_DIR_PRO  (variable)
```

O deploy vai para `{DH_DIR_DEV ou DH_DIR_PRO}/{APP_DIR}/`.
Cada app ocupa sua subpasta, sem interferir nas outras.

```
dev.ceu-lang.org/
├── dh/          ← repo dreamhost  (APP_DIR: dh)
├── blog/        ← repo blog       (APP_DIR: blog)
└── api/         ← repo api        (APP_DIR: api)

ceu-lang.org/
├── dh/
├── blog/
└── api/
```

---

## Setup único (uma vez por conta DreamHost)

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

| Tipo     | Nome         | Valor                              | Exemplo                    |
|----------|--------------|------------------------------------|----------------------------|
| Secret   | `DH_SSH_KEY` | Chave privada (`dreamhost_deploy`) | `-----BEGIN OPENSSH...`    |
| Variable | `DH_HOST`    | Hostname do servidor               | `servidor.dreamhost.com`   |
| Variable | `DH_USER`    | Usuário SSH                        | `meuusuario`               |
| Variable | `DH_DIR_DEV` | Diretório raiz do domínio dev      | `~/dev.meusite.com`        |
| Variable | `DH_DIR_PRO` | Diretório raiz do domínio produção | `~/meusite.com`            |

> Para compartilhar entre repos, use **Organization secrets/variables**
> ou configure os mesmos valores em cada repo.

---

## Setup por app (cada novo repo)

### 1. Copiar o workflow

Copiar `.github/workflows/deploy.yml` para o novo repo.

### 2. Mudar o `APP_DIR`

No topo do workflow, alterar:

```yaml
env:
  APP_DIR: minha-app  # nome da subpasta no DreamHost
```

### 3. Colocar arquivos em `web/`

O rsync envia o conteúdo de `web/` para o servidor.
Colocar todos os arquivos do site dentro de `web/`.

### 4. Adaptar os testes

Editar a seção de testes no workflow conforme o projeto.

---

## Checklist

### Uma vez (conta DreamHost)
- [ ] Criar subdomínio dev no painel DreamHost
- [ ] Gerar par de chaves SSH (ed25519)
- [ ] Adicionar chave pública no `authorized_keys` do DreamHost
- [ ] Testar SSH: `ssh -i dreamhost_deploy usuario@servidor`
- [ ] Criar Secret `DH_SSH_KEY` no GitHub
- [ ] Criar Variables: `DH_HOST`, `DH_USER`, `DH_DIR_DEV`, `DH_DIR_PRO`

### Cada novo repo
- [ ] Copiar `.github/workflows/deploy.yml`
- [ ] Definir `APP_DIR` no workflow
- [ ] Colocar arquivos em `web/`
- [ ] Push em `main` e verificar deploy em dev
- [ ] Tag `v*` e verificar deploy em produção

---

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

---

## Notas

- O `rsync --delete` remove do servidor arquivos que não existem mais no repo.
- Exclusões padrão: `node_modules`, `.env`.
- Cada app é independente: o `--delete` só afeta a subpasta `APP_DIR/` daquela app.
