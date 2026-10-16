# Reverse Proxying

**Scope:** `reverse_proxy` — the directive most Caddy
configs exist for — what it adds to requests, and what
its failures look like.

## One line

[`examples/Caddyfile.proxy`](examples/Caddyfile.proxy):

```
:8081 {
	reverse_proxy 127.0.0.1:8080
}
```

Whatever arrives on :8081 is forwarded to the app on
:8080 and the response relayed back. From the lab, the
visitor's view of the app through Caddy:

```
HTTP/1.1 200 OK
Server: SimpleHTTP/0.6 Python/3.12.3
Via: 1.0 Caddy

<h1>static works</h1>
```

The app's own headers survive; Caddy adds `Via` and —
more importantly — tells the app about the *original*
visitor.

## What the backend learns

Measured, not assumed: an echo backend printed the
headers it received through Caddy, versus direct
(a control run showed **zero** X-Forwarded headers
direct):

```
Via: 1.1 Caddy
X-Forwarded-For: ::1
X-Forwarded-Host: localhost:8096
X-Forwarded-Proto: http
```

- **`X-Forwarded-For`** — the visitor's IP. Without it,
  every request looks like it came from Caddy itself
  (127.0.0.1/::1), and your app's logs, rate limits, and
  IP-based rules all see one very busy localhost.
- **`X-Forwarded-Host` / `X-Forwarded-Proto`** — the name
  and scheme the visitor used, so the app can build
  correct links ("you arrived at https://app.example.org"
  even though the Caddy→app hop is http on localhost).

Any sane app framework reads these. If yours builds
wrong URLs or logs useless IPs behind the proxy, it's
not trusting forwarded headers — that's the app's
setting to find ("trust proxy" in most frameworks).

## When the app is down: 502

Stop the backend and the visitor gets:

```
HTTP/1.1 502 Bad Gateway
Server: Caddy
```

**502 is Caddy saying "I'm fine; the app isn't
answering."** It's the most useful error in proxying
because it assigns blame correctly: Caddy's log line for
it names the upstream and the dial error (connection
refused, in the dead-app case). 502 ≠ your Caddy config
is wrong; it means the config is *right* and the thing
behind it needs attention. (A 404 or a garbled page, by
contrast, is the app answering — Caddy is innocent.)

## The two mortal patterns

**One subdomain per app** (the clean way, needs DNS for
each name or a wildcard):

```
wiki.example.org { reverse_proxy 127.0.0.1:8080 }
chat.example.org { reverse_proxy 127.0.0.1:3000 }
```

**One domain, path per app** (when DNS is scarce):

```
example.org {
	handle_path /wiki* {
		reverse_proxy 127.0.0.1:8080
	}
}
```

`handle_path` strips the prefix before forwarding —
many apps break under a subpath unless they know their
base, which is why subdomains are the recommendation
and paths are the fallback. More shapes in
[Patterns](patterns.md).
