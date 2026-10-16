# What Caddy Is

**Scope:** the two jobs Caddy does and the one feature
that makes people switch to it.

## Job 1: web server

A web server answers HTTP requests: it takes "GET
/index.html" and returns a file. Caddy does this the way
you'd hope — point it at a folder and it's serving:

```
example.org {
	root * /var/www/example
	file_server
}
```

Three ideas in there: a **site block** (the address it
answers for), a **root** (where the files are), and
**file_server** (the directive that serves them).

## Job 2: reverse proxy

Most mortals meet Caddy here. Your apps — Open WebUI, a
wiki, a dashboard — each listen on some port like
:8080, speak plain HTTP, and have varying opinions about
logins and security. A **reverse proxy** stands in front
of them:

```
app.example.org {
	reverse_proxy 127.0.0.1:8080
}
```

Visitors talk to Caddy (ports 80/443, HTTPS, one public
name); Caddy talks to the app (localhost, plain HTTP).
The app never faces the internet directly. One line of
config per app, and the whole menagerie gets one front
door with one consistent set of rules.

## The famous feature: automatic HTTPS

Every other web server treats certificates as your
problem: obtain one, install it, renew it before it
expires, notice when renewal silently broke. Caddy
treats the certificate as *its* problem:

- The site address is a public domain → Caddy asks
  Let's Encrypt for a certificate, proves it controls
  the domain (the ACME protocol), installs the cert, and
  renews it in the background for the life of the
  process.
- The address is `localhost` or an internal name →
  Caddy uses its own built-in certificate authority
  instead (verified in the HTTPS chapter).
- HTTP requests to a site with a certificate get
  redirected to HTTPS automatically.

No directive enables this. It's what Caddy *does* unless
the address pattern says otherwise (a bare `:8080` port
or an `http://` prefix opts out, for internal-only
services).

## What it costs and what it isn't

Caddy is free and open source (Apache 2.0), a single Go
binary — the install is "put the file somewhere." It is
not a load balancer fleet manager, not a CDN, and not
an application server: your app still runs itself; Caddy
is the trustworthy doorway in front of it. For one
machine (or one home) full of services, that doorway is
exactly the missing piece.
