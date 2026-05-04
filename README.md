# claudefirm/dev-sites

GitOps registry for **`*-dev.foursli.de`** Coder workspaces.

Push a YAML manifest into `sites/<name>.yaml`. Within ~5 min, a Coder
workspace named `<name>` is provisioned and its dev server published at
`https://<name>-dev.foursli.de/` (auth gated by `auth.claudefirm.com`).
The aggregator at **`https://dev.foursli.de/`** lists every active site.

## Manifest schema

```yaml
# sites/blog.yaml  →  https://blog-dev.foursli.de/
template: dev-site-vite-react        # see "Templates" below
git_repo_id: claudefirm/blog          # optional; pre-clones the repo
expose_port: 5173                      # optional; default depends on template
description: ""                        # optional; aggregator falls back to <title>
```

## Templates

| name | what it runs | default port |
|---|---|---|
| `dev-site-vite-react` | `pnpm dev` (Vite + React + Tailwind) | 5173 |
| `dev-site-static-html` | Caddy serving `/srv/site/index.html` from PVC | 80 |
| `dev-site-mintlify-docs` | `mintlify dev` | 3000 |
| `paperclip-fork-dev` *(v1.1)* | clone `fourslide/paperclip` + `skaffold dev` | 8080 |
| `claudefirm-platform-dev` *(v1.1)* | clone `claudefirm/platform` + `skaffold dev` | 3100 |

## Naming

- `[a-z][a-z0-9-]*[a-z0-9]`, max 30 chars
- Reserved names (refused at reconcile): `auth, admin, api, www, dev, sites, index, app, status, paperclip, marketing-dev, private, local, sovereign, foursli`
- First-come-first-served. Collisions (a YAML for an already-taken `<name>`) get rejected; the reconciler comments on the offending PR (or commit) explaining why.

## Lifecycle

- Coder workspaces stop after 1 day idle (URL still resolves, returns 503 until restarted).
- Workspaces stopped >7 days are auto-destroyed; the manifest's subdomain frees up.
- Removing the manifest (`rm sites/blog.yaml && git push`) destroys the workspace immediately.

## Auth

Every `*-dev.foursli.de` URL goes through Authentik at `auth.claudefirm.com`. You sign in once; the cookie covers the whole `foursli.de` namespace.

## Where the moving parts live

- Reconciler: `iac/clusters/etcdrich/tenants/fourslide/dev-sites-reconciler.yaml` (CronJob, every 5 min, in cluster `etcdrich/fourslide` ns)
- Aggregator: `iac/clusters/etcdrich/tenants/fourslide/dev-sites-aggregator.yaml`
- Coder templates: `iac/tofu/coder/dev_site_*` directories (TF-managed)
