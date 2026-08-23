# Bunny Catalog Extras

Extra package manifests for [bunny](https://github.com/cristatus/bunny), read alongside the
[public catalog](https://github.com/cristatus/bunny-catalog) rather than replacing it. The two share
no package ids: everything here is a package upstream does not carry.

## Scope

Packages that fall outside the public catalog's Java and Node workstation scope: clients for the
infrastructure the team runs, ops and remote-access tooling, internal or vendored builds. Everything
a general Java or Node developer would want lives upstream; this catalog holds only what upstream
would close as out-of-scope.

Current packages:

- **teleport** — `tsh` and `tctl`, the client tools for Teleport-protected SSH, Kubernetes, and
  database infrastructure.
- **teleport-connect** — the Teleport Connect desktop app.
- **restic** — deduplicating, encrypted backups.
- **rclone** — file sync and transfer against cloud and remote storage.
- **kubectl**, **helm**, **k9s** — the Kubernetes client set.
- **kind**, **minikube** — local clusters. Both need a container runtime from your distro:
  `download.docker.com` publishes no digests for its static binaries, so bunny cannot ship Docker.

## Layout

```
.
├── index.json                  # Top-level catalog index (summary metadata per id)
├── tags.yaml                   # The tag vocabulary manifests may draw from
└── packages/{id}/manifest.yaml # One directory per package id
```

`index.json` is the entry point: bunny fetches it first to learn what packages exist and where to
find their manifests. Where a manifest sits does not decide where the package lands on disk: `kind:`
does, and bunny reads it from the index.

## Adding a package

1. Confirm it does not belong upstream — if a general Java/Node developer would want it, send it there.
2. Create `packages/{id}/manifest.yaml`. Copy the closest existing manifest as a template.
3. Append a corresponding entry to `index.json`, and declare any tag it uses in `tags.yaml`.
4. Point `catalog.local` at your checkout and run `bunny dev validate`.

The full manifest reference — field-by-field, with formatting conventions — is in the
[upstream README](https://github.com/cristatus/bunny-catalog/blob/main/README.md).

## Automatic version bumps

A daily GitHub Actions cron runs `bunny dev update` and pushes any new versions it finds (sha256 and
size are refreshed alongside; a separate CI job validates every push). Don't bump `version:` manually
unless upstream's archive layout also changed and you need to adjust `prepare:` to match.

For packages where you want manual review, omit the `update:` block from the source: the cron will
skip it.

## Using this catalog

`catalogs:` is exhaustive: writing it replaces the default chain instead of extending it, so list
the public catalog too or it goes away with `jdk-21`, `maven`, and everything else in it.

```yaml
# ~/.config/bunny/config.yaml
catalogs:
  - name: extras
    remote: https://raw.githubusercontent.com/<org>/bunny-catalog-extras/main
  - name: bunny
    remote: https://raw.githubusercontent.com/cristatus/bunny-catalog/main
```

Order is priority: the first catalog that carries a package id serves it. Nothing here shares an id
with upstream today, so the order changes nothing yet — it matters the day this catalog wants to
override a package rather than add one.

For a local checkout, swap `remote:` for `local: /path/to/bunny-catalog-extras`. See
[team deployment](https://github.com/cristatus/bunny/blob/main/docs/teams.md) in the bunny repo for
the full pattern.

## License

MIT
