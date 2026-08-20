# Practice session: Nginx virtual subdomain

This session configured a local Nginx virtual subdomain on the Debian server. The server was accessed over SSH from a local machine, and name resolution was provided manually on that local machine rather than by public DNS.

## Hostnames and local name resolution

In this exercise:

- `debian.local` is the server's local/base hostname.
- `one.debian.local` is the virtual subdomain served by Nginx.

Add the server IP and both names to `/etc/hosts` on the local machine used to access the server:

```text
<server-ip> debian.local one.debian.local
```

This mapping is local to that machine. It does not create a DNS record or make the hostname available to other clients.

## Subdomain web root

A separate directory was created under `/var/www` for the subdomain:

```text
/var/www/one.debian.local/
├── index.html
└── favicon.svg
```

The two files are:

- `index.html` - the page returned for the site root.
- `favicon.svg` - the SVG favicon file stored on disk.

The web root and its files live on the server and are not tracked in this repository. Create or update them on the Debian host, keeping ownership and permissions suitable for the Nginx worker to read them.

## Nginx site configuration

The server block for the subdomain listens on port 80 and points its `root` at the dedicated directory:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name one.debian.local;
    root /var/www/one.debian.local;
    index index.html;

    location = /favicon.ico {
        try_files /favicon.svg =404;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

The exact `/favicon.ico` location maps favicon requests to `favicon.svg`. If that SVG file is missing, Nginx returns `404` instead. All other requests are handled by the root location and return `404` when the requested path does not exist.

The repository follows Debian's usual site layout: keep the source file in `/etc/nginx/sites-available/one.debian.local` and enable it with a symlink in `/etc/nginx/sites-enabled/`:

```bash
sudo ln -s /etc/nginx/sites-available/one.debian.local \
    /etc/nginx/sites-enabled/one.debian.local
sudo nginx -t
sudo systemctl reload nginx
```

If the site was written directly in `sites-enabled` during the practice session, the server block is the same; the `sites-available` plus symlink layout is preferred for consistency with the rest of this repository's setup guide.

## Verification

From the local machine with the `/etc/hosts` entry, verify name resolution and the three relevant request paths:

```bash
getent hosts one.debian.local
curl -i http://one.debian.local/
curl -i http://one.debian.local/favicon.ico
curl -i http://one.debian.local/path-that-does-not-exist
```

The first request should return `index.html`, the favicon request should return the contents of `favicon.svg`, and the missing path should return `404`.
