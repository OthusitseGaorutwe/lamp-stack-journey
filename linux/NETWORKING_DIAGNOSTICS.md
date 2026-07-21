# Linux Networking Diagnostics — Debugging a "Works on Refresh" HTTP Issue

Notes from diagnosing an intermittent 404 on a custom port (6503) behind Nginx.
Useful as a general troubleshooting reference for any "first request fails,
second request works" style problem.

## Diagnostic order used

### 1. Inspect the full Nginx config as parsed (not just source files)

```bash
sudo nginx -T
```

`nginx -T` dumps every config file nginx actually loaded and merged — including
files pulled in via `include` directives that might live outside the usual
`sites-enabled`/`conf.d` folders. This is more reliable than grepping raw config
directories, since:

```bash
sudo grep -rl "6503" /etc/nginx/sites-enabled/ /etc/nginx/conf.d/
```

can return nothing even when a matching `listen` directive is active, if it's
defined in a config file outside those two paths.

### 2. Isolate the relevant server block

```bash
sudo nginx -T | grep -A 20 "listen.*6503"
```

Check for:
- `server_name` matches the domain being requested
- `root` points to the correct directory
- `try_files` (or equivalent) fallback logic is correct
- No duplicate/competing `server {}` blocks silently catching the same port
  (a block with `server_name _;` or no `server_name` can shadow the intended one)

### 3. Understand what does and doesn't show up in each log

```bash
sudo tail -50 /var/log/nginx/error.log
sudo tail -100 /var/log/nginx/access.log
```

Key distinction:
- **`error.log`** only captures server-level errors (permission issues, upstream
  failures, config problems) — a clean HTTP 404 that a config successfully
  routes and serves is *not* an error from nginx's perspective, so it won't
  appear here.
- **`access.log`** is where you'll find the actual request line and status code
  for every request nginx handled, successful or not.

If a specific request doesn't appear in `access.log` either, that's a strong
signal the request never reached this nginx instance at all.

### 4. Rule out a CDN/proxy sitting in front of the origin

```bash
dig yourdomain.example.com +short
```

Compare the returned IP against your server's actual public IP. If it belongs
to a CDN/proxy provider's IP range rather than your own infrastructure, the
proxy (not your origin) may be caching or mishandling the request — check the
proxy's caching rules before looking any further at the origin.

### 5. Test the origin directly, but watch out for NAT hairpinning

```bash
curl -iv "https://yourdomain.example.com:6503/some/path"
```

If you run this **from a machine on the same LAN as the server**, and it hangs
or times out, this is often **not** a real fault — many home/office routers
don't properly handle "hairpin NAT" (a LAN device reaching the router's public
IP, expecting to be routed back inside to the same LAN). The request looks like
it timed out, but an external client (like a third-party webhook or payment
provider) would never experience this.

To confirm hairpin NAT is the cause, test using the server's **private LAN IP**
directly instead of the public domain:

```bash
curl -ik "https://192.x.x.x:6503/some/path"
```

If this succeeds where the public-domain test failed, you've confirmed the
earlier timeout was a local network artifact, not a real server-side problem —
and you should stop testing from inside the LAN.

### 6. Identify what's actually listening on a port, if config location is unclear

```bash
sudo ss -tlnp | grep 6503
# or, if ss isn't available:
sudo netstat -tlnp | grep 6503
```

This tells you which process owns the port — useful when it's unclear whether
Nginx, a container's internal proxy, an application server started directly, or
a tunnel/forwarding process is actually the one handling connections.

## Key lessons

1. **`nginx -T` is more trustworthy than grepping config directories** — it
   reflects nginx's actual merged, active configuration, regardless of where
   the source files physically live.
2. **A quiet `error.log` doesn't mean nothing went wrong** — it only logs
   server-level faults. A normally-routed 404/200/etc. lives in `access.log`,
   not `error.log`.
3. **An empty `access.log` for a specific request usually means the request
   never reached that nginx instance** — check DNS, firewall rules, and whether
   a CDN/proxy is intercepting traffic upstream before assuming the app itself
   is broken.
4. **Testing a public domain from inside the same network as the server can
   trigger NAT hairpinning issues** that have nothing to do with the actual
   service. Always test both the public domain (external-facing) and the
   private LAN IP (internal-facing) to isolate network-layer artifacts from
   real server-side faults.
5. **`ss -tlnp` / `netstat -tlnp` is the fastest way to confirm which process
   owns a port** when config file locations are unclear or non-standard.