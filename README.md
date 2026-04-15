# JellyRock API Docs

[![API Reference](https://img.shields.io/badge/docs-api.jellyrock.app-purple "API reference")](https://api.jellyrock.app/)
[![License](https://img.shields.io/github/license/jellyrock/api-docs "License")](./LICENSE)
[![build](https://img.shields.io/github/actions/workflow/status/jellyrock/api-docs/deploy.yml?logo=github&branch=main "build")](https://github.com/jellyrock/api-docs/actions/workflows/deploy.yml?query=branch%3Amain)

Auto-generated BrightScript API reference for
[JellyRock](https://github.com/jellyrock/jellyrock), published at
**[api.jellyrock.app](https://api.jellyrock.app/)**.

## Tech stack

| Piece | Choice | Why |
|---|---|---|
| Source extraction | [`brighterscript-jsdocs-plugin`](https://www.npmjs.com/package/brighterscript-jsdocs-plugin) | Parses BrightScript `'` comments into JSDoc-compatible AST. |
| Site generator | [JSDoc](https://github.com/jsdoc/jsdoc) | De facto standard for JS/BrightScript API sites. |
| Theme | [`clean-jsdoc-theme`](https://github.com/ankitskvmdam/clean-jsdoc-theme) | Dark mode, responsive, customizable header/footer. |
| Analytics | [Umami](https://analytics.jellyrock.app) (self-hosted) | Privacy-respecting; injected via theme `footer` option. |
| Hosting | Caddy `file_server` on the JellyRock VPS | Same pipeline as `jellyrock.app`, `docs.jellyrock.app`, `dev.jellyrock.app`. |

## How it works

```text
jellyrock/jellyrock (push to main)
    │
    └─▶ repository_dispatch → update.yml
            │
            ├─ clones jellyrock/jellyrock @ main
            ├─ runs `npm run build`  ─▶  writes docs/ output
            └─ commits & pushes docs/ back to this repo
                    │
                    └─▶ push event → deploy.yml
                            │
                            └─ rsync docs/ → VPS:/opt/jellyrock/api-docs/
                                    │
                                    └─ Caddy serves api.jellyrock.app
```

- [`.github/workflows/build.yml`](.github/workflows/build.yml) — PR sanity check (build succeeds).
- [`.github/workflows/update.yml`](.github/workflows/update.yml) — rebuild triggered by the app repo.
- [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml) — rsync to VPS.
- [`jsdoc.json`](jsdoc.json) — JSDoc + clean-jsdoc-theme config (title, menu, footer, analytics).

## Local build

```bash
npm ci
git clone --depth 1 https://github.com/jellyrock/jellyrock.git
npm run build            # writes docs/
npx http-server docs     # preview at http://localhost:8080
```

## Updating

| Task | Where |
|---|---|
| Change the site title / header menu | [`jsdoc.json`](jsdoc.json) → `opts.theme_opts` |
| Swap analytics website ID | [`jsdoc.json`](jsdoc.json) → `opts.theme_opts.footer` |
| Point at a different source repo / branch | [`update.yml`](.github/workflows/update.yml) → `Checkout jellyrock` step |
| Change deploy target | [`deploy.yml`](.github/workflows/deploy.yml) + [`jellyrock/infra`](https://github.com/jellyrock/infra) Caddy config |

## Deployment secrets

Required repo secrets (inherited from the `jellyrock` org):

- `DEPLOY_SSH_KEY` — private key authorized on VPS `jellyrock@` user
- `VPS_KNOWN_HOSTS` — pre-verified `ssh-keyscan` output (avoids MITM)
- `VPS_HOST`, `VPS_USER` — deploy target

## Credits

- [brighterscript-jsdocs-plugin](https://www.npmjs.com/package/brighterscript-jsdocs-plugin)
- [clean-jsdoc-theme](https://github.com/ankitskvmdam/clean-jsdoc-theme)
- [Brightscript Function Comment](https://marketplace.visualstudio.com/items?itemName=AliceBeckett.brightscriptcomment) VSCode extension
