# Automatic HTTPS

**Scope:** how Caddy gets certificates without being
asked twice — the public-internet path (documented, it
needs your domain), and the internal path (verified here
for real).

## The public path (documented)

Give Caddy a public domain name as a site address and
this sequence happens on first run, unattended:

1. Caddy contacts **Let's Encrypt** (the free,
   nonprofit certificate authority) via the **ACME**
   protocol.
2. Let's Encrypt checks that you control the domain —
   usually by asking Caddy to answer a challenge on
   port 80 (HTTP-01) or via DNS (DNS-01).
3. A certificate is issued (90-day lifetime), installed,
   and used immediately. HTTP visitors are redirected
   to HTTPS.
4. Renewal happens in the background, well before
   expiry, forever. There is no step 5 for you.

The requirements are the whole fine print: the domain's
DNS must point at the machine, and ports 80/443 must be
reachable from the internet (port 80 for the common
challenge and the redirect). Fail those and issuance
fails — the Troubleshooting chapter has the shape of
that failure. Nothing else is required: no account, no
certbot, no renewal cron. (Documented: this guide's lab
had no public domain to issue against.)

## The internal path (verified)

For names the public internet can't vouch for —
`localhost`, LAN names, IPs — Caddy is its own CA:

[`examples/Caddyfile.tls`](examples/Caddyfile.tls):

```
localhost:8443 {
	tls internal
	respond "hello over TLS" 200
}
```

```
curl https://localhost:8443/
```

```
hello over TLS
```

Real TLS, real certificate verification — the cert
chains to **Caddy's local root**, stored at
`~/.local/share/caddy/pki/authorities/local/root.crt`.
In the lab, Caddy (running as root) also installed that
root into the machine's system trust store, which is why
curl verified it with no extra flags. Other devices
don't trust it until you install `root.crt` on them —
until then they show the usual "certificate not trusted"
warning, which for once is telling the truth: they don't
know this CA.

**When internal TLS is the answer:** services that only
your LAN should reach, but where plain HTTP is still a
bad idea (logins crossing the network, browsers refusing
features to insecure pages). When it's not: anything
the public touches — public visitors can't and shouldn't
trust your private CA; that's what the Let's Encrypt
path is for.

## The decisions Caddy makes for you

- **HTTP → HTTPS redirects** are automatic for sites
  with certs.
- **Renewals** start long before expiry and retry with
  backoff; a healthy Caddy's certificate situation is a
  log line, not an incident.
- **Storage**: certificates live in Caddy's data
  directory (`~/.local/share/caddy` for a user run,
  `/var/lib/caddy` under the packaged service).
  Back that directory up if the machine matters — losing
  it means re-issuing everything, and Let's Encrypt
  rate-limits the impatient.
