# clified-catalog

Catálogo **live** do [Clified](https://github.com/maikramer/clified) — o ficheiro
[`registry.yaml`](registry.yaml) mapeia um nome curto (ex.: `denv`) ao
repositório git + ferramenta registada no `tools.yaml` desse repositório.

O `clified-install` lê este catálogo **por defeito** (com cache local de 1h e
fallback ao catálogo embutido no pacote quando está offline). Adicionar ou
alterar uma ferramenta aqui fica imediatamente disponível via
`clified-install --get <tool>` / `--catalog` — **sem** novo release do clified.

A partir do clified **0.8**, os subcomandos `clified` também consomem este
catálogo: `clified search <termo>` filtra entradas (marca `(privado)` /
`[instalado]`) e `clified get <tool>` instala; `clified list` / `clified update` /
`clified uninstall` gerem o que já está instalado (state em
`~/.config/clified/state.json`). Ver
[Managing installed tools](https://github.com/maikramer/clified#managing-installed-tools).

## Esquema (`tools:`)

```yaml
tools:
  <nome>:
    repo:        <url git>            # obrigatório; público ou privado
    tool:        <nome em tools.yaml> # default: <nome>
    tools_yaml:  <path do tools.yaml> # default: tools.yaml
    description: <texto curto>        # apresentado em --catalog
    access:      public | private     # default: public (informational)
```

- `access: private` é **informational**: o clified marca a ferramenta como
  `(privado)` em `--catalog` e avisa antes de clonar. O clone só funciona se o
  utilizador tiver permissão de leitura no git da organização (chave SSH ou
  token HTTPS via git credential manager). Sem acesso, o clified **falha de
  forma suave** com uma mensagem clara (não crasha).

## Adicionar uma ferramenta

1. Acrescenta uma entrada em `tools:` em [`registry.yaml`](registry.yaml).
2. `access: public` para repos públicos; `access: private` para privados.
3. Abre um PR / push para `main`. O clified passa a usar a nova entrada assim
   que a cache expira (TTL 1h); para forçar refresh imediato:
   `clified-install --refresh-catalog --catalog` ou `CLIFIED_CATALOG_TTL=0`.

## Publicar uma ferramenta (3 caminhos)

- **Sem catálogo** (qualquer repo público): `clified-install --get <nome> --repo https://github.com/.../x.git`.
- **No catálogo público** (este repo): abre um PR com `access: public`.
- **Self-host privado** (empresa): mantém o teu próprio `registry.yaml` (privado
  ou local) e aponta o clified com `CLIFIED_CATALOG=<url-ou-path>`.

## Variáveis de ambiente

| Env | Default | Efeito |
|-----|---------|--------|
| `CLIFIED_CATALOG` | *(unset)* | Override do catálogo: URL (`http(s)://...`) ou path local. |
| `CLIFIED_CATALOG_TTL` | `3600` | TTL da cache em segundos. `0` = sempre fetch; `-1` = só bundled (offline). |

## Notas

- Este repo é **público** de propósito (o fetch via `raw.githubusercontent.com`
  não usa token). Contém apenas metadados — não código nem segredos.
- Repos privados (ex.: `LocatelliSupermercados/*`) são referenciados aqui por
  URL; o clone depende das creds git do utilizador.

## GameDev (público)

O monorepo [`maikramer/GameDev`](https://github.com/maikramer/GameDev) expõe
15 ferramentas + meta `gamedev` (instala `all`):

| Chave | Ferramenta | Tipo |
|-------|------------|------|
| `text2d`, `text3d`, `texture2d`, … | Python GPU tools | Python 3.13 |
| `materialize` | PBR material CLI | Rust (`cargo`) |
| `vibegame` | Browser 3D engine | Bun |
| `gamedev` | Todas as ferramentas presentes no checkout | meta → `all` |

One-liner (sem clone):

```bash
curl -fsSL https://raw.githubusercontent.com/maikramer/clified/main/install.sh | bash -s -- --get text2d
```

Chaves = entradas em `GameDev/tools.yaml`. Com clone: `./install.sh text2d` na raiz do repo.
