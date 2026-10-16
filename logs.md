# Logs

**Scope:** Caddy's own logging (always on) and per-site
access logs (opt-in) — what they look like and where
they go.

## The server log: JSON, to stderr

Caddy logs its own life — startups, certificate events,
proxy errors — as one JSON object per line on stderr.
From the lab's journal:

```json
{"level":"error","ts":1792194958.2938142,"msg":"loading initial config: loading new config: http app module: start: listening on :8091: listen tcp :8091: bind: address already in use"}
```

Under systemd, stderr *is* the journal, so
`journalctl -u caddy` collects all of it (the systemd
guide's model, again). Fields are consistent: `level`,
`ts`, `msg`, plus structured extras like `error` and
`logger`. JSON annoys some humans at first glance; it
pays off the first time you filter a week of logs with
`jq` instead of regex archaeology. (`caddy run` in a
terminal shows the same JSON. There is no "pretty mode"
switch — piping through `jq` or a log viewer is the
idiom.)

Level discipline: certificate issuance, renewal, and
reload outcomes all appear here. A Caddy you can't see
the logs of is a Caddy whose certificate expiry you'll
learn about from your visitors.

## Access logs: opt-in, per site (documented)

Request logging is off until a site asks:

```
example.org {
	log {
		output file /var/log/caddy/access.log
		format json
	}
	reverse_proxy 127.0.0.1:8080
}
```

`log` inside a site block logs that site's requests —
method, path, status, size, duration, the visitor's
address as Caddy saw it. Formats: `json` (structured,
the default recommendation), `console` (human-readable),
and `single_field` for piping elsewhere. Output can be
a file (with size/time-based rolling built in), stdout,
or the systemd journal via `output stderr` conventions
in service setups.

(Documented: the lab verified Caddy's server/error
logging and its journal trail directly; per-site access
log blocks follow Caddy's documented format.)

## Reading a proxy failure in the log

The 502 from the proxying chapter leaves a line with
`"logger":"http.log.error"` (or in the server log)
naming the upstream and the refusal — `dial tcp
127.0.0.1:8080: connect: connection refused` is the
signature of "app is down." Learn to tell its two
neighbors apart: a *timeout* dialing means the app (or
the network to it) is wedged, and no error at all means
Caddy relayed the app's own response — the failure, if
any, is inside the app and inside its logs, not
Caddy's.
