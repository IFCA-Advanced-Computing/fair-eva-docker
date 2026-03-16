# fair-eva-docker
Docker compose to launch FAIR EVA

## Requirements
- Docker + Docker Compose v2 (`docker compose`)

## Start
1) Edit `.env` and set the refs you want to run:
   - FAIR EVA backend (`FAIR_EVA_REF`)
   - web UI source (`WEB_REPO`, `WEB_REF`)
   - plugins (`FAIR_EVA_PLUGINS`)

2) Build the images and start the services (API + web UI):

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

In this repo, plugins are installed at **build time** via variables read automatically from `.env` by `docker compose`.

The main variables are:

- `WEB_REPO`: Git URL used to clone the web interface (new UI version can be pointed here).
- `WEB_REF`: web branch/tag to build.
- `FAIR_EVA_PLUGINS`: space-separated pip specs or `git+https://...` URLs.
- `FAIR_EVA_PLUGIN_EXTRA_DEPS`: extra Python dependencies needed by plugins.
- `FAIR_EVA_API_EVAL_PATH`: API endpoint used by the web for evaluations. Default: `/v1.0/rda/rda_all`.
- `FAIR_EVA_API_PLUGINS_PATH`: API endpoint used by the web to discover installed plugins. Default: `/v1.0/endpoints`.
- `FAIR_EVA_PLUGINS_FILE`: optional local JSON file for hardcoded plugin choices in the web. Leave it empty to use the API-discovered list.

By default, this repo now builds with `oai-pmh + gbif`.

Example `.env` for **new web interface + gbif plugin**:

```dotenv
WEB_REPO="https://github.com/IFCA-Advanced-Computing/fair_eva_web_client.git"
WEB_REF="main"
FAIR_EVA_PLUGINS="git+https://github.com/IFCA-Advanced-Computing/fair-eva-plugin-oai-pmh@main git+https://github.com/ferag/fair-eva-plugin-gbif.git@main"
FAIR_EVA_PLUGIN_EXTRA_DEPS="python-dwca-reader geopandas idutils numpy pandas pycountry requests shapely"
```

After editing `.env`, rebuild and deploy:
```bash
docker compose -f compose.yml up -d --build
```

If you only changed plugins and want to rebuild only the backend:
```bash
docker compose -f compose.yml build --no-cache fair-eva
docker compose -f compose.yml up -d
```

The installed plugins are exposed by the API at `/v1.0/endpoints`, which is what the web UI uses to show the available repositories. The Docker image filters internal entries such as `__pycache__` so the interface only shows real plugins.
