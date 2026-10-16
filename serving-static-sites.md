# Serving a Static Site

**Scope:** the smallest useful Caddy — files in a folder,
on the web. Everything verified against the shipped
[`examples/Caddyfile.static`](examples/Caddyfile.static).

## The config

```
:8082 {
	root * /srv/www
	respond /health "all good" 200
	file_server
}
```

Run it:

```
caddy run --config Caddyfile.static
```

Three requests, three behaviors:

```
curl http://localhost:8082/          → the folder's index.html
curl http://localhost:8082/health    → all good
curl http://localhost:8082/nope.txt  → 404
```

- A directory request serves its `index.html`
  (file_server's default).
- The `respond` line intercepts exactly one path —
  a health check that answers even if the site folder is
  later moved or emptied. (Directive order is why
  `respond` wins over `file_server` for its path:
  Caddy's built-in order puts it first.)
- A missing file is an ordinary 404 from file_server.

## Making it a real site

Swap the `:8082` address for your domain and the picture
changes by itself (see the HTTPS chapter):

```
mysite.example.org {
	root * /srv/www
	file_server
	encode gzip
}
```

Same file_server; now Caddy also fetches a certificate
and redirects HTTP → HTTPS. The mortal's static-hosting
story is: folder of HTML, five lines, done. Add
`encode gzip` for compression — for a docs site like a
certain guide series, that and file_server are the
entire production config.

## Small print that matters

- **Trailing paths are literal.** `root` must be a
  directory that exists *where Caddy runs* — in a
  container or a service with a different working
  directory, relative paths surprise; use absolute ones.
- **Directory listings are off by default.** No
  index file = 404, not a file listing. (Add `browse`
  to file_server when you *want* a listing — a handy
  poor-man's file share on a LAN.)
- **Permissions are the silent 403/404.** Caddy reads
  files as the user it runs as. A site folder readable
  only by you is a folder of 403s when Caddy runs as a
  service account; the journal/status will say little,
  because from Caddy's chair the file simply isn't
  readable. `namei -l` the path when a static site
  misbehaves.
