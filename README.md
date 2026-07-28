# GeoServer

A standalone, containerized [GeoServer](https://geoserver.org/) instance for
publishing spatial data (PostGIS tables, files, and other sources) as OGC web
services — WFS, WMS, and OGC API - Features — to clients like QGIS and ArcGIS
Pro.

This is a general-purpose publishing service. A single instance can connect to
multiple databases and data sources at once, each organized under its own
workspace, so it can be reused across unrelated projects.

## Stack

- Official GeoServer image (`docker.osgeo.org/geoserver`), pinned to a specific
  version.
- Runs anywhere Docker or Podman is available.

## Layout

```
.
├── docker-compose.yml     # service definition
├── .env.example           # template for required environment variables
├── .env                    # real values (never committed)
└── data/                   # GeoServer data directory (never committed)
```

The `data/` directory is GeoServer's `GEOSERVER_DATA_DIR`. All runtime
configuration — workspaces, data stores, published layers, styles, and
security settings — is stored here. It is intentionally excluded from version
control because it contains connection credentials and is environment-specific.

**This means published layers and store configuration are not captured in git.**
The repository holds only the infrastructure (compose file and environment
template). Each deployment configures its own layers through the GeoServer admin
UI, and the `data/` directory should be backed up separately as its own source
of truth.

## Setup

1. Copy the environment template and fill in real values:

   ```bash
   cp .env.example .env
   ```

   Set a strong admin password. Do not leave the GeoServer defaults in place.

2. Start the service:

   ```bash
   docker compose up -d
   ```

   (Or `podman-compose up -d` when using Podman.)

3. Open the web interface at `http://<host>:8080/geoserver` and log in with the
   admin credentials from `.env`.

## First-run hardening

GeoServer ships with well-known default credentials. On a new instance:

- The **admin** password is set from `GEOSERVER_ADMIN_PASSWORD` on first startup.
- The **root / master** password (used for keystore encryption) is separate and
  must be changed in the admin UI under **Security → Passwords**.

Change both before exposing the instance beyond localhost.

## Adding data

Data sources are configured through the admin UI, not in code:

1. **Workspaces** — create a workspace to group a project's layers.
2. **Stores** — add a store (e.g. PostGIS) with the connection details for the
   source database. Use a read-only database role where possible.
3. **Layers** — publish tables or views from the store as layers.

Published layers are then available as WFS/WMS/OGC API endpoints for GIS
clients.

## Notes

- The bundled demo data is skipped by default for a clean instance.
- On SELinux systems (e.g. Fedora), the data volume mount uses the `:Z` flag for
  relabeling. This flag is ignored on systems that do not enforce SELinux, so
  the same compose file works across environments.
- Default service port is `8080`.