# fair-eva-docker
Docker compose to launch FAIR EVA

## Requirements
- Docker + Docker Compose v2 (`docker compose`)

## Start
Build the images and start the services (API + web UI):

```bash
docker compose -f compose.yml up -d --build
```

URLs:
- Web UI: `http://localhost:8000`
- API: `http://localhost:9090` (for example `http://localhost:9090/v1.0/openapi.json`)

## Status and logs
```bash
docker compose -f compose.yml ps
docker compose -f compose.yml logs -f
```

Logs for a specific service:
```bash
docker compose -f compose.yml logs -f fair-eva
docker compose -f compose.yml logs -f fair-eva-web
```

## Stop / teardown
Stop and remove containers/network:
```bash
docker compose -f compose.yml down
```

Stop without removing containers:
```bash
docker compose -f compose.yml stop
```

## Plugins: how to choose what to install
FAIR EVA needs at least one plugin to run evaluations (in API requests it is selected with `repo`, e.g. `oai_pmh`).

In this repo, plugins are installed at **build time** (reproducible) via `FAIR_EVA_PLUGINS`, which is passed as a `--build-arg` to the backend image.

Examples:

1) Keep the default (installs `fair-eva-plugin-oai-pmh` from GitHub):
```bash
docker compose -f compose.yml up -d --build
```

2) Choose plugins explicitly (for example, a plugin from Git):
```bash
FAIR_EVA_PLUGINS="git+https://github.com/IFCA-Advanced-Computing/fair-eva-plugin-oai-pmh@main" \
  docker compose -f compose.yml up -d --build
```

2b) Install more than one plugin (space-separated pip specs):
```bash
FAIR_EVA_PLUGINS="git+https://github.com/IFCA-Advanced-Computing/fair-eva-plugin-oai-pmh@main git+https://github.com/<ORG>/<ANOTHER_PLUGIN>@<REF>" \
  docker compose -f compose.yml up -d --build
```

3) Change plugins and rebuild only the backend:
```bash
FAIR_EVA_PLUGINS="..." docker compose -f compose.yml build --no-cache fair-eva
docker compose -f compose.yml up -d
```

Note: `FAIR_EVA_PLUGINS` accepts any pip spec (PyPI packages, `pkg==x.y`, or `git+https://...`).
