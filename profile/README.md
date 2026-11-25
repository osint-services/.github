# Open Source Intelligence Platform

The Open Source Intelligence Platform is a modular investigation workspace for discovering public profiles, inspecting public X metadata, resolving caller information, searching authorized local datasets, and mapping explicit geospatial evidence. The [`platform`](https://github.com/osint-services/platform) repository brings live and imported sources together behind one nginx endpoint and the [`whoisit`](https://github.com/osint-services/whoisit) Electron UI.

## Install the current release

1. Install Git, Docker, and Docker Compose v2. Clone the platform at v1.1.0 with its pinned submodules, then start the backend:

   ```bash
   git clone --branch v1.1.0 --recurse-submodules https://github.com/osint-services/platform.git
   cd platform
   cp .env.example .env
   docker compose up --build -d
   ```

2. On Debian or Ubuntu x86-64, [download Whoisit v1.0.0](https://github.com/osint-services/whoisit/releases/download/v1.0.0/whoisit_1.0.0_amd64.deb), install it with `sudo apt install ./whoisit_1.0.0_amd64.deb`, and launch Whoisit. The desktop package connects to the running backend at `http://127.0.0.1:80`. Manage the Compose stack and its settings from the platform checkout.

3. To use **Certificates**, set `CERTIFICATE_INVENTORY_DOMAINS` in the platform `.env` for domains whose ownership you have verified. Recreate the dataset service with `docker compose up -d dataset_service`. See the [certificate inventory setup guide](https://github.com/osint-services/platform/blob/v1.1.0/docs/certificate-inventory.md#using-the-inventory).

## Architecture

```mermaid
flowchart LR
    UI["whoisit<br/>Electron + React"] -->|HTTP| Proxy["nginx proxy<br/>localhost:80"]

    subgraph Platform["platform · Docker Compose"]
        Proxy -->|/scan/*| Checker["profile_checker<br/>FastAPI"]
        Proxy -->|/focus*| Search["profile_search<br/>FastAPI + X API"]
        Proxy -->|/phone_search*| Phone["phone_search<br/>FastAPI + Twilio Lookup v2"]
        Proxy -->|/datasets*| Data["dataset_service<br/>FastAPI + SQLite"]
        Proxy -->|/map/*| Mapping["social_mapping<br/>FastAPI + GeoJSON"]
        Data --> Volume[("dataset_data<br/>persistent volume")]
        Mapping --> Data
        Mapping --> Cache[("geocoder cache<br/>persistent volume")]
    end

    Search --> X["X API"]
    Mapping --> X
    Mapping --> Nominatim["Nominatim<br/>search-origin geocoding"]
    Phone --> Twilio["Twilio Lookup API"]
```

The services remain independently runnable, while the platform repository provides:

- Docker Compose orchestration and a shared bridge network
- nginx routing through port `80`
- environment-variable injection for external APIs
- health and readiness endpoints
- persistent normalized profile and phone datasets
- explicit geospatial dataset evidence, radius search, and combined GeoJSON responses
- a durable, rate-limited search-origin geocoder cache
- helper scripts for starting, stopping, rebuilding, and viewing logs
- a unified Electron interface with identity search, native map, dataset import, integration status, and local history workspaces

## Workflow

1. **Enter a username or phone number** in the shared Search workspace. Auto mode routes clearly formatted phone numbers to phone services and other values to profile services; explicit modes handle ambiguous identifiers.
2. **Choose sources** across all connected services, live APIs only, or imported datasets only. Excluded providers are not called.
3. **Inspect a profile** to retrieve public X identity fields, media, verification state, dates, pinned/recent post IDs, entities, and metrics. Public scan evidence remains available when paid X inspection is unavailable.
4. **Review phone results** from Twilio Lookup v2 and exact normalized matches in imported phone datasets.
5. **Import authorized data** from CSV, JSON, JSONL, or NDJSON through one sparse entity pipeline. Each row can contain a profile identifier, a phone identifier, or both; matching either identifier returns the stored association.
6. **Explore the Map workspace** by entering a place, radius, optional keyword, and source selection. Results retain source, accuracy, confidence, timestamps, provenance, and associated identity metadata.
7. **Manage integrations** by reviewing the configured and reachable state of X/Tweepy, Twilio Lookup, local datasets, and Social Mapping, or replacing provider credentials from the desktop UI.

Live providers and imported sources are queried independently. A provider error is shown with that source while successful results from other sources remain available.

## Repositories

| Repository | Role |
| --- | --- |
| [`platform`](https://github.com/osint-services/platform) | Top-level Docker Compose orchestration, nginx proxy, shared configuration, and service submodules. |
| [`whoisit`](https://github.com/osint-services/whoisit) | Electron and React workspace for identity search, native mapping, dataset imports, provenance, and local history. |
| [`social_mapping`](https://github.com/osint-services/social_mapping) | FastAPI geospatial aggregator and standalone React/Leaflet client for imported datasets and provider-supplied X locations. |
| [`profile_checker`](https://github.com/osint-services/profile_checker) | FastAPI service that validates configured public profile locations for a username. |
| [`profile_search`](https://github.com/osint-services/profile_search) | FastAPI and Tweepy service that retrieves public X profile metadata from a discovered profile URL. |
| [`phone_search`](https://github.com/osint-services/phone_search) | FastAPI service that resolves caller metadata through Twilio Lookup v2. |

## API routes

All routes are exposed through nginx at `http://127.0.0.1:80`.

| Method and route | Service | Purpose |
| --- | --- | --- |
| `GET /scan/{username}` | `profile_checker` | Scan configured platforms for a username. |
| `GET /focus?url={profile_url}` | `profile_search` | Inspect an X profile and return public metadata. |
| `GET /phone_search?phone_number={e164_number}` | `phone_search` | Look up phone and caller metadata. |
| `GET /datasets` | `dataset_service` | List imported datasets. |
| `GET /datasets/schema/entity` | `dataset_service` | Describe the unified sparse entity fields. |
| `POST /datasets/import` | `dataset_service` | Import mapped entities with profile identifiers, phone identifiers, or both. |
| `GET /datasets/search/profiles?query=...&fuzzy=true` | `dataset_service` | Search imported profile records. |
| `GET /datasets/search/phones?phone_number=...` | `dataset_service` | Search imported phone records by normalized E.164 number. |
| `GET /datasets/search/geo?latitude=...&longitude=...` | `dataset_service` | Radius-search records that contain an explicit valid coordinate pair. |
| `GET /map/search?place=...&radius_miles=...` | `social_mapping` | Return combined evidence-oriented GeoJSON from selected sources. |
| `DELETE /datasets/{dataset_id}` | `dataset_service` | Delete a dataset and its records. |
| `GET /scan/healthz` | `profile_checker` | Username-scan service liveness. |
| `GET /focus/readyz` | `profile_search` | Profile service readiness, including X credential configuration. |
| `GET /phone_search/healthz` | `phone_search` | Phone service liveness. |
| `GET /datasets/healthz` | `dataset_service` | Dataset service liveness. |
| `GET /map/healthz` | `social_mapping` | Map service liveness. |
| `GET /map/readyz` | `social_mapping` | Map cache and optional-provider readiness. |

## Configuration

Copy the example file before starting the stack:

```bash
cp .env.example .env
```

Set these values in `.env`:

| Variable | Used by | Meaning |
| --- | --- | --- |
| `TWEEPY_BEARER_TOKEN` | `profile_search`, `social_mapping` | X API app-only bearer token used for public profile and recent geospatial post lookup. |
| `TWILIO_ACCOUNT_SID` | `phone_search` | Identifier for the Twilio account making Lookup API requests. |
| `TWILIO_AUTH_TOKEN` | `phone_search` | Secret used to authenticate Twilio API requests. |
| `DATASET_DB_PATH` | `dataset_service` | SQLite path; Compose sets `/data/datasets.db` in a persistent named volume. |
| `NOMINATIM_BASE_URL` | `social_mapping` | Server-side geocoder used only for the investigator's entered search origin. |
| `NOMINATIM_USER_AGENT` | `social_mapping` | Identifying User-Agent with a project or contact URL. |
| `GEOCODER_CACHE_DAYS` | `social_mapping` | Durable normalized place-cache refresh interval; default 30 days. |

Never place real credentials in `.env.example`, source files, logs, issues, or commits. The local `.env` file is intentionally ignored by Git. The desktop Integrations workspace never displays existing values; replacements are handled by the Electron main process, written atomically with owner-only permissions, and applied by recreating only the affected API services when Docker is accessible.

## Development setup

### Prerequisites

- Git, including configured access to the service submodules
- Docker Engine
- Docker Compose through the `docker compose` command
- Node.js 22 or newer and npm to run the Electron UI
- Optional X API and Twilio credentials for their corresponding live features

### Start the platform

```bash
git clone https://github.com/osint-services/platform.git
cd platform
git submodule update --init --recursive
cp .env.example .env
# Add your credentials to .env
docker compose up --build -d
```

Or use the provided helper:

```bash
./scripts/start.sh
```

Verify the routed services:

```bash
curl http://127.0.0.1/scan/healthz
curl http://127.0.0.1/focus/readyz
curl http://127.0.0.1/phone_search/healthz
curl http://127.0.0.1/datasets/healthz
curl http://127.0.0.1/map/readyz
```

### Start the desktop UI

With the platform stack running:

```bash
cd whoisit
npm install
npm start
```

The UI checks platform availability, can attempt to start the parent Compose stack, and provides Search, Map, Datasets, Integrations, History, and Certificates workspaces. The native Map workspace calls the shared `/map/search` API rather than embedding the standalone client. Credential-free dataset mapping remains available when X is not configured. Imported records preserve their dataset, source, observation time, optional confidence, normalized fields, associations, coordinates when explicitly supplied, and original row for audit.

## Operations

From the `platform` repository:

```bash
./scripts/start.sh    # Build and start the stack
./scripts/stop.sh     # Stop the stack
./scripts/build.sh    # Rebuild service images
./scripts/logs.sh     # Stream service logs
```

## Responsible use

Use the platform only for lawful access to public or properly authorized data. Import only data you are permitted to retain and search. A map marker is source evidence, not proof of a person's present location. Free-text profile locations are never converted into claimed post coordinates, and associated phone data never creates or implies a phone or device location. Protect API credentials and dataset volumes, respect provider and dataset license terms, apply appropriate retention controls, and consider the privacy and safety impact of collecting or sharing identifying information. The local API currently has no authentication layer and should not be exposed to an untrusted network without access controls.
