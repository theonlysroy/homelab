# Repository guidance for coding agents

## Purpose and scope

This repository documents and partially captures the setup of a small self-hosted Debian server running on an old laptop. It is an infrastructure/documentation repository, not an application: there is no application source, package manifest, build system, test suite, linter, or CI configuration.

The documented target is Debian 13 (Trixie). The setup covers SSH hardening, optional Wi-Fi via NetworkManager, Nginx, UFW, Fail2Ban, and a Docker runtime intended for future containerized services.

## Repository map

- `README.md` - short entry point linking to the documentation.
- `docs/index.md` - server overview, implemented capabilities, planned work, and links to the other guides.
- `docs/server-setup.md` - manual, phased installation and hardening procedure, from Debian installation through SSH, Nginx, UFW, and Fail2Ban.
- `docs/decisions.md` - rationale for key-only SSH authentication and a non-default SSH port.
- `docs/troubleshooting.md` - troubleshooting log; currently empty apart from its heading.
- `docs/nginx-subdomain-practice.md` - practice-session record for the local `one.debian.local` virtual subdomain, dedicated web root, and favicon mapping.
- `docs/assets/` - server screenshots embedded by the documentation.
- `nginx/sites-available/one.debianserver.local` - the checked-in Nginx server block for the local hostname.
- `.gitignore` - ignores `.env` and `.env.*`; do not commit credentials or machine-local configuration.

## Architecture and operational data flow

The checked-in Nginx configuration is a static-site server block, despite the overview describing Nginx as a reverse proxy. Requests for `one.debianserver.local` arrive on HTTP port 80 over IPv4 or IPv6, are served from `/var/www/one.debianserver.local/html`, and use `try_files` to fall back to `/index.html`. It has no `proxy_pass`, upstream, HTTPS listener, certificate configuration, or container integration.

The setup guide expects local DNS or an `/etc/hosts` entry to resolve the hostname to the server IP. The server's live state is outside this repository: SSH settings are under `/etc/ssh`, Nginx sites under `/etc/nginx`, the web content under `/var/www`, and firewall/Fail2Ban state is managed by their respective Debian services. The repository currently contains no Dockerfile or Compose file.

## Working with changes

There are no project dependencies to install and no repository-defined build, test, lint, or release commands. Changes are normally Markdown edits or Nginx configuration edits. Preserve the existing structure: lowercase, hyphenated documentation filenames; ATX Markdown headings; fenced `bash`, `html`, and `nginx` examples; backticks for commands and configuration keys; and angle-bracket placeholders for hostnames, IPs, users, and paths.

Before submitting a change:

1. Review `git diff` and run `git diff --check`.
2. For documentation changes, verify links and image paths relative to the file being edited and check that commands match the surrounding setup phase.
3. For Nginx changes, apply the site file to the target host, run `sudo nginx -t`, and only then reload with `sudo systemctl reload nginx`. The setup guide shows the analogous `sites-available`/`sites-enabled` symlink workflow.
4. For server changes, follow the order and safety checks in `docs/server-setup.md`; this repository does not automate or test those operations locally.

## Configuration and security requirements

Use placeholders in examples and keep passwords, private keys, tokens, real credentials, and machine-specific secrets out of Git. The guide uses privileged Debian commands and assumes access to the target machine as root initially or as a configured sudo user. It recommends confirming non-root SSH access with sudo and key authentication before disabling root login and password authentication.

The documented intended controls are key-only SSH authentication, no remote root login, UFW rules for required services, and Fail2Ban protection. The guide also describes allowing HTTP, HTTPS, and optional custom application ports, but the committed Nginx site currently configures HTTP only. Any change to the SSH port must be kept consistent across `sshd_config`, the UFW rule, Fail2Ban's `[sshd]` jail, and client commands.

## Important gotchas

- The files under `docs/` are a manual runbook, not an idempotent provisioning system. Do not assume a documented command has been run merely because it appears in the repository.
- `nginx/sites-available/one.debianserver.local` must be deployed on the server and enabled; it also assumes that `/var/www/one.debianserver.local/html/index.html` exists.
- `one.debianserver.local` is a local hostname. It will not resolve for other machines without suitable local DNS or hosts-file configuration.
- The overview says Docker is ready and lists a containerized service behind Nginx as next work, but no Docker or Compose configuration is currently checked in.
- The overview mentions a reverse proxy and HTTPS firewall access, but the current site is plain HTTP static serving. Treat those as planned or externally configured unless the repository is updated.
- The initial root-password SSH option is explicitly temporary and high risk. Keep an active recovery session while hardening SSH, and do not disable the working access path before testing the replacement.
- There are no automated tests or CI checks to catch broken Markdown links, unsafe shell instructions, or invalid Nginx syntax. Validate those manually, especially after editing operational commands.
