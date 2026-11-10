# HoLiC — Secure deployment notes

## Required environment variables

Set these in your VPS/container hosting dashboard (do not put them in source code or commit them):

- `SECRET_KEY`: a unique random value with at least 32 random bytes. Generate one with `python -c "import secrets; print(secrets.token_hex(32))"`.
- `ADMIN_USERNAME`: administrator login name (default: `admin`).
- `ADMIN_PASSWORD`: a unique password of at least 14 characters. The first login creates the administrator account using a scrypt password hash.
- `COOKIE_SECURE=1`: enable when the site is served over HTTPS. Keep it `0` for plain local HTTP testing only.

The old shared-password login and old SHA-256 password records have been disabled. The bundled `data.json` starts with no users; the first valid login using the configured administrator credentials bootstraps the admin account. Existing server records have been assigned to `admin` so they remain manageable. Back up your original `data.json` before replacing the deployment.

## Important operational security

- Run behind HTTPS and a reverse proxy; do not expose the Flask development server directly to the internet.
- Run the panel as a dedicated, unprivileged Linux user. Do not run it as root.
- Hosting users can run programs and shell commands inside their own project directories. This is inherent to a code-hosting panel; do not give accounts to untrusted people and do not treat this as strong OS-level isolation. For public multi-user hosting, use separate containers/VMs with resource and network restrictions per user.
- Keep the VPS firewall restrictive. Expose only the reverse-proxy ports and the ports your hosted projects explicitly need.
- Back up `data.json` and `servers/` before upgrades.
- Never upload unknown ZIP files or run unknown projects. This patch checks ZIP paths and rejects archive symlinks, but it cannot make arbitrary uploaded programs safe.

## Changes in this hardened build

- Replaced shared-password / SHA-256 login with per-account scrypt password hashes and a one-time environment-configured administrator bootstrap.
- Removed the old shared password from bundled data.
- Enforced ownership checks on log viewing and log clearing.
- Added server-name/runtime validation, safer uploaded filenames, ZIP traversal checks, and ZIP symlink rejection.
- Added main-file path and port validation, HTTP-only/SameSite session cookies, upload size limit, and production secret-key startup check.

This is a targeted hardening pass, not a formal penetration test or a guarantee that arbitrary hosted code is safe.
