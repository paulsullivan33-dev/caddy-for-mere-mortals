# Glossary

**ACME** — the protocol Caddy uses to obtain and renew
certificates from Let's Encrypt automatically: prove
control of a domain, receive a certificate.

**Admin API** — the REST API a running Caddy exposes
(default `localhost:2019`) for inspecting and replacing
its live config. `caddy reload` is a client of it.

**Adapt** — compiling a Caddyfile into the JSON config
Caddy actually runs (`caddy adapt`).

**Basic auth** — HTTP's built-in username/password
prompt. In Caddy: the `basic_auth` directive, passwords
stored as bcrypt hashes.

**Caddyfile** — Caddy's human config format: site blocks
of directives. Adapts to JSON at load time.

**Directive** — one instruction inside a site block
(`root`, `file_server`, `reverse_proxy`, `respond`...).

**Internal CA** — Caddy's built-in certificate
authority for names the public internet can't verify
(`localhost`, LAN names), enabled with `tls internal`.
Trust requires installing its root certificate.

**Let's Encrypt** — the free, automated public
certificate authority Caddy uses by default.

**Matcher** — a pattern narrowing a directive's reach:
a path (`/health`), a named set (`@lan`), `*` for all.

**Placeholder** — a runtime value in a config or log
format: `{uri}`, `{host}`, `{remote}`.

**Reverse proxy** — a server that receives requests and
forwards them to an app behind it, relaying responses —
Caddy's most common job.

**Site block** — one address line plus its braced
directives: the unit of Caddyfile organization.

**Upstream** — the app behind a proxy: the `dial`
address in `reverse_proxy 127.0.0.1:8080`.

**Via / X-Forwarded-** — headers a proxy adds so the
app knows the original visitor: their address
(`X-Forwarded-For`), the name and scheme they used.
