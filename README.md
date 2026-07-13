# hackaton

DataHub ingestion project.

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) running (with at least 2 CPUs and 8 GB RAM allocated)
- [uv](https://docs.astral.sh/uv/) installed
- Python 3.11 (managed via the project `.venv`)

## Setup

Install the project dependencies (creates/uses `.venv`):

```powershell
uv sync
```

This installs `acryl-datahub[sqlalchemy,datahub-rest]`, which provides the `datahub` CLI.

## Run DataHub (project docker-compose)

This project ships a self-contained [docker-compose.yml](docker-compose.yml) and
[.env](.env). Docker Compose loads `.env` automatically (same directory), so the
whole stack — GMS, frontend, MySQL, Kafka, OpenSearch, system-update, actions —
comes up with a single command:

```powershell
docker compose up -d
```

When GMS is healthy, DataHub is available at:

- **Web UI:** http://localhost:9002
- **Default login:** `datahub` / `datahub`

Everything is driven declaratively from `.env` — image versions, ports, and the
metadata-service auth settings. No flags needed: `COMPOSE_PROFILES=quickstart`
in `.env` activates the services.

### Configuration ([.env](.env))

```env
DATAHUB_VERSION=v1.5.0.6              # image tag (reuses locally-pulled images)
COMPOSE_PROFILES=quickstart          # auto-activates the stack
HOME=C:/Users/lupev                  # host path for bind mounts (change per machine)

# Metadata Service authentication
METADATA_SERVICE_AUTH_ENABLED=true
TOKEN_SERVICE_ENABLED=true
DATAHUB_TOKEN_SERVICE_SIGNING_KEY=...   # required when auth is on
DATAHUB_TOKEN_SERVICE_SALT=...          # required when auth is on
```

> `.env` holds the token signing key/salt — it is gitignored. Keep the key/salt
> stable across restarts or existing access tokens are invalidated.

### Managing the stack

```powershell
docker compose ps          # status of all services
docker compose logs -f datahub-gms-quickstart   # follow GMS logs
docker compose down        # stop (volumes/data preserved)
docker compose down -v     # stop AND delete all data volumes
```

> **Note:** this compose reuses the same project name (`datahub`), network, and
> volumes as `datahub docker quickstart`, so the two are interchangeable against
> the same data. Prefer this project file — the CLI-generated one under
> `~/.datahub/quickstart/` gets overwritten on every `datahub docker quickstart`.

### Common commands

```powershell
# Check the CLI version
datahub version

# Verify the local instance is healthy
datahub docker check

# Load the sample metadata (optional, great for a first look)
datahub docker ingest-sample-data

# Stop DataHub (containers stay, data preserved)
datahub docker quickstart --stop

# Tear everything down and remove all local DataHub data
datahub docker nuke
```

### Notes

- First startup can take several minutes while images download.
- If containers fail to start, confirm Docker Desktop has enough resources
  allocated (Settings → Resources).
- This project pins **Python 3.11**, the version DataHub officially supports.
  `uv sync` provisions it automatically; if you need it explicitly, run
  `uv venv --python 3.11`.

## References

See [DataHub Quickstart docs](https://docs.datahub.com/docs/quickstart) for the
full guide.
