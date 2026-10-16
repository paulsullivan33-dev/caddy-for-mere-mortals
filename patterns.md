# Patterns

**Scope:** the Caddyfile shapes that recur in real
configs. Site blocks, matchers, and basic auth were run
for real in earlier chapters; the redirect/rewrite
entries are (documented) standard patterns.

## Named app, TLS, proxy — the flagship

```
app.example.org {
	reverse_proxy 127.0.0.1:8080
}
```

Four-fifths of real-world Caddy is this with different
names filled in. Everything else in this chapter is a
variation for a specific itch.

## A second matcher: long-lived health endpoint

```
example.org {
	respond /health "ok" 200
	reverse_proxy 127.0.0.1:8080
}
```

`respond` takes its path and only its path; everything
else falls to the proxy. Handy when the app itself has
no cheap health page — the doorway answers for it.

## Old address → new address (documented)

```
old.example.org {
	redir https://new.example.org{uri} permanent
}
```

`{uri}` is a placeholder (Caddy's word for a runtime
variable); `permanent` makes it a 301. Whole-site
redirects are one block, no rewrite-rule archaeology.

## Static + app on one name (documented)

```
example.org {
	root * /srv/www
	file_server /static/*
	reverse_proxy 127.0.0.1:8080
}
```

The matcher confines file_server to `/static/*`; the
rest proxies. The cousin of the subpath pattern from
the proxying chapter, with the same warning: apps that
generate their own URLs need to know where they live.

## Separate logs per site (documented)

```
example.org {
	log {
		output file /var/log/caddy/example.log
	}
	reverse_proxy 127.0.0.1:8080
}
```

A per-site access log block — the Logs chapter covers
formats; the pattern worth remembering is that `log`
lives *inside* the site it describes.

## The catch-all that says no

```
:443 {
	respond "nothing here" 404
}
```

Requests arriving for names you didn't configure get a
polite nothing instead of your first site by accident.
(Which site answers an unknown name is otherwise
"the first matching block" — a classic source of
"why is it serving *that*?" — see Troubleshooting.)
