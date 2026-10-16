# Running Caddy

**Scope:** operating Caddy day to day — foreground vs.
service, the zero-downtime reload, and the admin API
underneath both.

## Ways to run it

- **`caddy run --config Caddyfile`** — foreground.
  Logs stream to your terminal. This is how you develop
  a config, and — importantly — how a service manager
  runs it in production (the process *is* the daemon;
  no forking games).
- **`caddy start` / `caddy stop`** — daemonizes via the
  admin API and returns your prompt. Fine for
  experiments; the service manager is the better keeper
  for anything permanent.
- **As a systemd service** — the packaged installs ship
  an official `caddy.service` unit; the shape will be
  familiar from this series' systemd guide:
  `ExecStart=/usr/bin/caddy run --environ --config
  /etc/caddy/Caddyfile`, `ExecReload=` pointing at
  `caddy reload`. (Documented — the unit ships with
  Caddy's packages; this guide's lab ran Caddy under
  `systemd-run` transient units instead, same
  supervision model.)

## Reload: new config, no restart

Edit the Caddyfile, then:

```
caddy reload --config Caddyfile
```

Caddy *adapts* the file to JSON and posts it to the
running instance's admin API, which swaps the config
atomically: listeners stay open, in-flight connections
finish on the old config, new ones get the new. Verified
live in the lab — a `respond` string changed
("all good" → "reloaded and live") and the next request,
a second later, served the new text with the process
never having stopped.

Two consequences worth internalizing:

- A **bad** reload is rejected and the old config keeps
  serving — failure is safe by default. (Still
  `caddy validate` first; belt and suspenders.)
- Anything that *can't* be hot-swapped (the listen
  addresses of the admin API itself, storage locations)
  is the rare exception that needs a real restart.

## The admin API

A running Caddy exposes a small REST API, by default on
`localhost:2019` — localhost only, deliberately. The
running proxy's live config, fetched in the lab:

```
curl http://localhost:2019/config/
```

```json
{"apps":{"http":{"servers":{"srv0":{"listen":[":8091"],
"routes":[{"handle":[{"handler":"reverse_proxy",
"upstreams":[{"dial":"127.0.0.1:8090"}]}]}]}}}}}
```

That JSON is the adapted Caddyfile, alive. `reload`,
`start`, and `stop` are all just API clients; so is
anything you write. Two cautions: an exposed admin API
is total control of the server (keep it on localhost —
that's an architectural rule, not a preference), and
API-made changes are **not** written back to your
Caddyfile — the file remains the source of truth only
if all changes flow through it. Pick one direction and
stay with it: mortals pick the file.

## Upgrades and the version

```
caddy version
```

```
v2.11.7 h1:yj0Y4fYZGPkSvibBJ1sTWE33xC0fxztVyXEW5iIdUT4=
```

A single static binary: upgrading is replacing the file
and restarting the service. No runtime, no dependency
drift. Check the release notes for config-affecting
changes; Caddyfiles have been admirably stable across
v2.
