# CLAUDE.md

## Repository

`signoz-vultr` is a **deployment**: desired state only, no source code. It runs
the `signoz` Package Skill (`../signoz`) against one Vultr instance serving
`signoz.bigconfig.online`.

`colors.yml` is the only file to edit. Everything under `.colors/` is generated
and must never be edited, read as source, or committed. `.envrc.private` holds
every credential and must never be read, printed, or copied.

## Commands

```sh
./green build              # render .colors/signoz-vultr/ — contacts nothing
./green create --dry-run   # walk the workflow, skip every side effect
./green create             # converge for real — requires explicit authorization
./green delete             # guarded and destructive — separately authorized
```

`build` and `--dry-run` work with an empty environment and are the safe way to
check a `colors.yml` edit.

## The launcher is a copy

The root `./green` is a **copy** of `.agents/skills/package-signoz-green/green`,
not a symlink. `npx skills update -p` rewrites the payload and leaves the root
file alone, so the project would keep running the old pin while the lockfile
claimed the new one. After every update:

```sh
npx skills update -p
cp .agents/skills/package-signoz-green/green green
```

## What this deployment assumes

- The Cloudflare token can edit the `bigconfig.online` zone. Several other
  deployments share that zone; this package only reads the zone and writes one
  record, so it does not touch zone-level settings anything else depends on.
- The `signoz-backup` R2 bucket exists. This deployment owns only its
  `signoz-vultr/` prefix and must never delete the bucket or other objects.
- Keygen mode: `~/.ssh/signoz-vultr` is generated and owned by the package.
  Adding `vultr-ssh-keys` here would switch the deployment to opt-out mode.

## Secrets

`COLORS_PAR_SIGNOZ_ROOT_PASSWORD` is the root account's only record — the
account cannot be edited or deleted from the UI, and provisioning runs at
application startup only, so a change takes effect on the next recreate.

The OTLP ingestion bearer token and the Postgres password are generated on the
server and exist nowhere else. Read the token with
`ssh signoz-vultr cat /etc/signoz/ingestion.env`.

## Git

Work on the current branch. Do not commit or push unless explicitly authorized.
