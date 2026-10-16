# Authentication and Access

**Scope:** putting a password (or an IP rule) in front of
an app that doesn't have one — Caddy as the bouncer.
Basic auth verified for real.

## basic_auth

Plenty of self-hosted tools ship with weak or no login,
intended for "trusted networks" that turn out to be
less trusted than hoped. `basic_auth` adds HTTP Basic
authentication at the proxy: the browser shows its
standard login prompt, Caddy checks the credentials,
and only then does the request reach the app.

Passwords are stored **hashed** (bcrypt). Generate one:

```
caddy hash-password
```

```
$2a$14$BTT0Wy7SiQdO6...   (type the password at the prompt; the hash is the output)
```

[`examples/Caddyfile.auth`](examples/Caddyfile.auth)
(the demo password is `demo-password` — change it):

```
:8083 {
	basic_auth {
		paul $2a$14$...
	}
	respond "welcome to the secret area" 200
}
```

Verified outcomes:

```
no credentials    → 401
wrong password    → 401
paul + password   → welcome to the secret area
```

Swap the `respond` for a `reverse_proxy` and the same
gate stands in front of a real app.

Honest limits of Basic auth: credentials ride along on
every request (base64, not encryption) — it is only as
safe as the TLS in front of it, so pair it with HTTPS,
always. It's a lock for a garden gate, not a vault door:
right for "keep honest people and scanners out of my
dashboard," wrong for "protect valuable data from a
motivated attacker."

## Restricting by address (documented)

```
example.org {
	@lan remote_ip 192.168.0.0/16 10.0.0.0/8
	handle @lan {
		reverse_proxy 127.0.0.1:8080
	}
	respond "not from around here" 403
}
```

A named matcher (`@lan`) captures the source ranges;
`handle` routes matching requests to the app and
everyone else to a 403. This is the "admin panel only
from home" pattern. One trap, documented because it
bites constantly: behind *another* proxy or a tunnel,
`remote_ip` sees the nearest hop, not the visitor —
that's what `X-Forwarded-For` handling and
`trusted_proxies` exist for, and it's worth reading
Caddy's docs before relying on IP rules through layers
of forwarding.

## Which tool when

- App has real accounts → use them; Caddy stays a
  doorway.
- App has nothing, audience is you → basic_auth over
  HTTPS, or IP rules, or both.
- App should not exist publicly at all → the strongest
  config is an address it never gets: bind it to
  localhost, proxy only from a name reachable on your
  LAN or tailnet, and let Caddy stay out of the
  internet's sight entirely.
