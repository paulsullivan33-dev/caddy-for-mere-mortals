# Troubleshooting

**Scope:** the failures you'll actually meet, symptom →
fix. Messages are verbatim from real runs (Caddy 2.11.7)
unless marked (documented).

## "unrecognized directive: bogus_directive"

```
adapting config using caddyfile: Caddyfile.bad:2: unrecognized directive: bogus_directive
```

**Symptom:** Caddy won't start or reload; `caddy
validate` reports the same.

**Fix:** a directive name Caddy doesn't know — a typo
(`file_sever`), a plugin directive in a Caddy built
without that plugin, or a directive in the wrong place
(not everything is legal inside a site block; `tls`, for
instance, has both global and per-site forms). The
error names the file and line; start there.

## "syntax error: unexpected token ..., expecting '}'"

```
syntax error: unexpected token '200', expecting '}', at Caddyfile.unclosed:2
```

**Symptom:** validate/adapt fails on structure.

**Fix:** a brace problem — a missing close, or a block
opened on a line the parser reads as a directive with
arguments. Caddyfile blocks must open `{` at the *end*
of the site/directive line. `caddy fmt` on the file
often makes the structural slip visible in its diff.

## "bind: address already in use"

```
loading initial config: ... listening on :8091: listen tcp :8091: bind: address already in use
```

**Symptom:** startup dies with a listen error in the
JSON log.

**Fix:** something already owns the port — usually a
previous Caddy you forgot was running (check
`systemctl status caddy` / `ss`/`lsof` for the owner),
or another server entirely. Note the mirror-image trap:
two Caddies *can* share a port on Linux in some
setups, which hides the duplicate until behavior gets
weird. One Caddy per machine is the sane rule.

## 502 Bad Gateway

**Symptom:** visitors get `502 Bad Gateway` with
`Server: Caddy`.

**Fix:** Caddy is healthy; the upstream isn't
answering. Confirm the app listens where the config
points (`curl 127.0.0.1:8080` from the Caddy machine),
then read Caddy's log for the dial error: connection
refused (app down), timeout (app wedged or a firewall),
TLS errors (you proxied to an HTTPS port with an HTTP
upstream line — `reverse_proxy` speaks plain HTTP to
upstreams unless the upstream is named with
`https://`). Full discussion in
[Reverse proxying](reverse-proxying.md).

## Certificate issuance fails (documented)

**Symptom:** a public site stays unreachable over
HTTPS, and the server log shows ACME errors — challenge
timeouts, "connection" refusals from the CA, or rate
limit notices.

**Fix, in order:** DNS actually points at this machine
(check from outside, not from the machine itself —
split-brain DNS lies); ports 80 and 443 are reachable
from the internet (router NAT, cloud firewall, ISP
blocking inbound 80 is a real thing on residential
connections); nothing else already squats 80/443.
Fix the cause, restart, and Caddy retries. If you hit
a Let's Encrypt rate limit by flapping, stop flapping:
the limit window is hours-to-days, and the staging
endpoint exists for exactly this testing phase.

## "It serves the wrong site"

**Symptom:** a name you configured shows a different
site, or an unconfigured name shows a real one.

**Fix:** site selection is by the request's Host (and
SNI for TLS). Overlapping blocks — a `:443` catch-all
plus named sites, or a wildcard — match in Caddy's
documented specificity order, not file order. List what
the running instance thinks it has: the admin API's
`/config/` (or `caddy adapt` on the file) shows the
compiled truth. (Documented mechanics; the API itself
was verified in the Running Caddy chapter.)

## Internal-CA certificates rejected by other devices

**Symptom:** `localhost` HTTPS works on the Caddy
machine, but your phone/laptop shows a trust warning
for the LAN site.

**Fix:** expected — those devices don't know Caddy's
private CA. Install its root certificate
(`~/.local/share/caddy/pki/authorities/local/root.crt`,
or `/var/lib/caddy/...` for the service install) into
each device's trust store, or accept the warning for
LAN-only tools. Do not "fix" this by disabling
verification in anything that matters.
