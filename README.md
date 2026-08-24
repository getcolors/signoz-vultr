# signoz-vultr

Desired state for a single-node [SigNoz](https://signoz.io/) observability
stack on Vultr, serving **https://signoz.bigconfig.online**.

Built by the [`signoz`](https://github.com/getcolors/signoz) Package Skill:
OpenTofu manages the instance, its firewall and a proxied Cloudflare `A`
record; Ansible converges ClickHouse, ClickHouse Keeper, a Postgres metastore,
the schema migrator, the SigNoz application, the `signoz-otel-collector`
ingester and Caddy.

## Use

```sh
direnv allow               # once, after cloning
./green build              # render .colors/signoz-vultr/
./green create --dry-run   # walk the workflow, no side effects
./green create             # converge
```

## Sending telemetry

SigNoz community edition has no ingestion keys, so Caddy gates the OTLP paths
with a bearer token generated on the server:

```sh
token=$(ssh signoz-vultr sed -n 's/^SIGNOZ_INGEST_TOKEN=//p' /etc/signoz/ingestion.env)

OTEL_EXPORTER_OTLP_ENDPOINT=https://signoz.bigconfig.online
OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer ${token}"
```

Only OTLP/HTTP is published. gRPC on 4317 stays on loopback.

## Signing in

`https://signoz.bigconfig.online` — the root account is the
`signoz-root-email` in `colors.yml`, with `COLORS_PAR_SIGNOZ_ROOT_PASSWORD`
from `.envrc.private`. That variable is the account's only record: the root
user cannot be edited or deleted from the UI.

## Recovery

A daily timer dumps the Postgres metastore to `r2:signoz-backup/signoz-vultr/`.
Telemetry is deliberately not backed up. Restore is `./green create` plus:

```sh
gunzip -c metastore-<stamp>.sql.gz | \
  ssh signoz-vultr docker compose -f /opt/signoz/compose.yml exec -T metastore \
    psql -U signoz -d signoz
```

## Credentials

Seven `COLORS_PAR_*` variables in the gitignored `.envrc.private`; the header
of `colors.yml` lists them. Never export `COLORS_PAR_PROFILE`.
