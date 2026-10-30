# ERPNext

[ERPNext](https://erpnext.com) using the official Frappe Docker image and service
commands. This package includes no custom Python scripts, entrypoint wrappers,
migration state machine or automatic recovery framework.

## First installation

Use fresh dedicated storage. Choose the site name and database root password in
the install form. A 404 at the portal is expected until a site is created.

After installation, open the **backend** container shell in TrueNAS with `bash`.
Run this upstream command, replacing `erpnext.localhost` if the form used another name:

```bash
bench new-site erpnext.localhost --db-host db --db-root-username root --mariadb-user-host-login-scope='%' --install-app erpnext
```

Enter the database root password from the install form when prompted, then choose
an Administrator password. Do not repeat this for an existing site. If creation
fails, inspect the output and follow upstream troubleshooting; do not delete data.
Enable scheduling:

```bash
bench --site erpnext.localhost enable-scheduler
```

Sign in as `Administrator`. The configured site-name routing header supports
access by IP. Changing the form does not rename a site or rotate an existing DB password.

## Backup, upgrade and recovery

These are operator-managed upstream operations. This package does not automatically
back up, migrate, roll back or import sites, and does not block runtime during an
incomplete migration. From the backend shell, a manual backup is:

```bash
bench --site erpnext.localhost backup --with-files
```

Backups are in `sites/<site>/private/backups`. Preserve the database, public/private
files, configuration and encryption key outside app storage; test restoration.
Restrict backup access because configuration contains credentials. An image
downgrade or app snapshot is not a verified database rollback.

Before changing images, follow the upstream maintenance/backup/upgrade procedure
and rehearse on a separate restored copy. Stop users and background writes first.
The target image's upstream migration command is `bench --site <site> migrate`;
the full maintenance procedure must also control workers and scheduler. This
package provides no automatic upgrade workflow.

This is a **fresh-storage architecture replacement**, not an in-place update of
the earlier guarded prototype. Do not apply it to that app or reuse its storage
as a shortcut to remove migration guards. Existing production/test apps are untouched.

Health checks cover MariaDB, Redis and the frontend TCP listener. Other ERPNext
services use upstream processes without probes into Frappe/RQ internals. Listener
health does not establish site readiness. Resource limits apply per container.

## Upstream references

- [Service definitions](https://github.com/frappe/frappe_docker/blob/61d35da827ebc04d01b64c9e78385597ccebc9d8/compose.yaml)
- [Site operations](https://github.com/frappe/frappe_docker/blob/61d35da827ebc04d01b64c9e78385597ccebc9d8/docs/04-operations/01-site-operations.md)
- [Example site creation](https://github.com/frappe/frappe_docker/blob/61d35da827ebc04d01b64c9e78385597ccebc9d8/pwd.yml)

The example's default passwords and floating versions are not used. Images remain
pinned to ERPNext 16.52.1 (bundled Frappe 16.53.0), MariaDB 11.8 and Redis 8.6.
This is a draft contribution; accepted CDN artwork and full upstream CI are pending.
