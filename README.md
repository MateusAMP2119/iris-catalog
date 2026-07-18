# iris-catalog

The public pack catalog for [Iris](https://github.com/MateusAMP2119/iris-lakehouse):
prebuilt pipeline packs an engine installs by name through `iris catalog install <pack>`
(or the `:catalog` overlay in `iris ps`).

## How an engine uses this repo

Add the index URL to `iris.toml`:

```toml
catalogs = ["https://raw.githubusercontent.com/MateusAMP2119/iris-catalog/main/catalog.json"]
```

The daemon fetches `catalog.json`, resolves each pack's tarball against the index URL,
and verifies the pinned `sha256` before a single byte is parsed — a tampered tarball
refuses, naming both digests. Clients name packs, never URLs: all catalog egress is
daemon-side.

## Index schema (`catalog.json`)

- `format` — integer, versioned from day one; engines refuse unknown formats.
- `packs[]` — `name`, `description`, `tags`, `path` (tarball relative to the index URL),
  `sha256` (over the tarball), `requires` (minimum engine release, mandatory).

## Pack layout

A pack tarball carries workspace-relative paths:

- `pipelines/<lane>/iris-declare.yaml` — the lane composer.
- `pipelines/<lane>/<pipeline>/` — one declaration plus one script per pipeline.
- `schemas/<schema>/<table>/table.yaml` — the declared tables.
- `README.md` — pack documentation (shown in previews, never materialized).

## Pack conventions

- No secrets. No `env_file` entries pointing at files the pack does not carry.
- Workers speak turn-protocol frames on stdout; stderr is free-form log.
- Declared tables live under `schemas/`; packs never touch engine-owned surfaces.
- `requires` is mandatory and names the oldest engine release the pack works on.

## Founding packs

| pack | what it shows |
| --- | --- |
| `quake-monitor` | USGS earthquake feed into `demo.quakes` plus a derived report lane |
| `dlq-demo` | a lane that fails on purpose: dead letters, replay, drain |

## Publishing a pack

Tar the pack directory (sorted paths, no leading `./`), gzip it, drop it under `packs/`,
and add its index entry with the tarball's `sha256`. Engines cap tarballs at 64 MiB and
single files at 8 MiB, and refuse non-regular entries and path escapes.
