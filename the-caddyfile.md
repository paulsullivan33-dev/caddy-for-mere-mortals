# The Caddyfile

**Scope:** Caddy's human config format — site blocks,
addresses, directives — and the two commands (`validate`,
`fmt`, `adapt`) that keep it honest.

## Anatomy

```
example.org {
	root * /var/www/example
	file_server
	encode gzip
}
```

- The first line is the **site address**: what Caddy
  answers for. A domain, `localhost`, `:8080` (any host,
  that port), or several comma-separated.
- Inside the braces, **directives**: one instruction
  per line. `root * ...` sets the site root (the `*` is
  a path matcher meaning "all requests"), `file_server`
  serves files, `encode gzip` compresses responses.
- Tabs or spaces for indentation — Caddy doesn't care,
  but `caddy fmt` has opinions (below).

Directives run in a fixed, sensible order built into
Caddy (routing before handling, for instance), *not*
strictly top-to-bottom like a script. Mostly you never
think about it; when two directives seem to ignore each
other's order, that's why. **Matchers** narrow a
directive to some requests: `respond /health "ok" 200`
answers only the `/health` path — you used one already
if you ran the examples.

## The three honesty commands

```
caddy validate --config Caddyfile
```

```
Valid configuration
```

Validation parses the whole file and reports problems
with file, line, and cause — verbatim from the lab:

```
adapting config using caddyfile: Caddyfile.bad:2: unrecognized directive: bogus_directive
```

```
caddy fmt --diff Caddyfile
```

shows what the canonical formatting would change (tabs,
alignment) and `--overwrite` applies it. Teams fight
about formatting so computers don't have to; Caddy ships
the referee.

## Underneath: it's JSON all the way down

The Caddyfile is a friendly skin. `caddy adapt` compiles
it to the JSON Caddy actually runs — the same JSON the
admin API serves about a live instance (fetched for real
from the running proxy):

```json
{"apps":{"http":{"servers":{"srv0":{
  "listen":[":8081"],
  "routes":[{"handle":[{"handler":"reverse_proxy",
  "upstreams":[{"dial":"127.0.0.1:8080"}]}]}]
}}}}}
```

You can configure Caddy in raw JSON if you enjoy that.
Nobody enjoys that. But knowing the Caddyfile *adapts*
explains two things: why `validate` says "adapting
config" in its errors, and why a running Caddy can be
reconfigured through its API — the config is data, not
a program. Chapter 8 returns to that.
