# Caddy for Mere Mortals

**Caddy** is a web server and reverse proxy whose claim
to fame is one sentence: **HTTPS by default, and it does
the certificate part for you.** Point it at a domain and
it obtains a real Let's Encrypt certificate, serves your
site or app over HTTPS, and renews the certificate
forever — no certbot cron, no expiry emails, no
fourteen-step tutorials. The config file fits on an
index card for most real uses.

This guide assumes you can use a terminal and know what
a web app on a port is. Everything Caddy-specific —
Caddyfiles, directives, the admin API — is explained
from zero.

## What this guide covers

- [**What Caddy is**](what-caddy-is.md) — web server,
  reverse proxy, and the certificate machinery, in plain
  terms
- [**The Caddyfile**](the-caddyfile.md) — site blocks,
  addresses, directives, and the JSON underneath
- [**Serving a static site**](serving-static-sites.md) —
  files on the web in three lines
- [**Reverse proxying**](reverse-proxying.md) — putting
  Caddy in front of your apps, headers and 502s included
- [**Automatic HTTPS**](automatic-https.md) — how the
  certificate magic works, its requirements, and internal
  TLS for the LAN
- [**Patterns**](patterns.md) — subdomains, subpaths,
  redirects, and the usual shapes
- [**Authentication and access**](authentication-and-access.md) —
  basic auth and IP rules in front of apps that lack them
- [**Running Caddy**](running-caddy.md) — as a service,
  reloading without dropping connections, and the admin
  API
- [**Logs**](logs.md) — what Caddy writes, where, and how
  to get access logs in the format you want
- [**Troubleshooting**](troubleshooting.md) — symptom →
  fix, with the real error messages
- [**Glossary**](glossary.md) — the terms, collected in
  one place

## What it doesn't cover

Building web applications (Caddy serves them; it isn't
one), Caddy's plugin ecosystem in depth, and enterprise
PKI. The mortal's Caddy is: static sites, proxied apps,
automatic certificates.

## How to use this guide

Everything was run for real with Caddy 2.11.7 — reverse
proxying (including the headers a backend actually
receives, checked against a control), static serving,
basic auth, a reload with zero downtime, and HTTPS via
Caddy's internal CA. The shipped files in
[`examples/`](examples/) all run as-is. The one piece
that needs *your* public domain and DNS — a real Let's
Encrypt certificate — is marked **(documented)**: the
mechanism is explained from Caddy's documentation and
the parts verifiable locally were verified.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [nginx for Mere Mortals](https://paulsullivan33-dev.github.io/nginx-for-mere-mortals/)
- [Traefik for Mere Mortals](https://paulsullivan33-dev.github.io/traefik-for-mere-mortals/)
- [Docker Compose for Mere Mortals](https://paulsullivan33-dev.github.io/docker-compose-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/caddy-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/caddy-for-mere-mortals/blob/main/LICENSE-CODE).
