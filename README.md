# Ansible: Hardened Nginx Web Server

An Ansible playbook that provisions an Ubuntu host into a reasonably
hardened Nginx web server: firewall locked down to only the ports it
needs, fail2ban watching for brute-force attempts, unattended security
updates, and an Nginx config with version info hidden and basic security
headers set.

## Structure

```
site.yml                          # entry point
inventory/hosts.ini                # target hosts
roles/security/tasks/main.yml      # ufw, fail2ban, unattended-upgrades
roles/nginx/tasks/main.yml         # nginx install + config
roles/nginx/templates/             # nginx.conf.j2, default_site.conf.j2
roles/nginx/handlers/main.yml      # reload nginx on config change
```

## What `security` role does

- Installs `ufw`, `fail2ban`, `unattended-upgrades`.
- Firewall: default-deny incoming, default-allow outgoing, then explicit
  allows for SSH, HTTP (80), and HTTPS (443).
- Enables `fail2ban` to block repeated failed SSH login attempts.
- Configures unattended upgrades to auto-install security patches.

## What `nginx` role does

- Installs Nginx.
- Deploys a hardened `nginx.conf`: `server_tokens off` (don't leak the
  Nginx version), and security headers (`X-Frame-Options`,
  `X-Content-Type-Options`, `Referrer-Policy`).
- Deploys a default site config that denies access to dotfiles.
- Reloads Nginx automatically (via a handler) whenever a config template
  changes — not a full restart, so existing connections aren't dropped.

## Usage

Update `inventory/hosts.ini` with your real hosts, then:

```bash
ansible-playbook -i inventory/hosts.ini site.yml --check   # dry run first
ansible-playbook -i inventory/hosts.ini site.yml
```

Requires SSH access to the target hosts and a user with sudo privileges
(`become: true` in `site.yml`).

## Why this project

Config management is often demoed as "install a package." This one is
built around the two things that actually matter for a host exposed to
the internet: the firewall rules that decide what can reach it at all,
and automatic patching so known vulnerabilities don't sit unpatched.

## License

MIT
