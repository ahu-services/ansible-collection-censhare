# censhare Keycloak Ansible Role

> [!WARNING]
> This role uses default credentials for demonstration and testing purposes. **For production environments, you must override these defaults.** Relying on the default credentials poses a significant security risk. Please refer to the "Role Variables" section to identify and change all default passwords and secrets.

This Ansible role installs and configures Keycloak as the authentication service for censhare Systems. It includes configuration for managing realms, clients, users, and SMTP settings.

## Requirements

- Ansible 2.16 or higher (matches the collection `requires_ansible` setting).
- Target hosts should be running a compatible Linux distribution with Podman available. The role relies on Podman Quadlets (`state: quadlet`) when Keycloak or its bundled Postgres DB runs in containers.

## Role Variables

### Keycloak General Configuration

- `censhare_keycloak_version`: The version of Keycloak to install. Default: `26.4.7` (pin this in your inventory if you do not want newer collection defaults on upgrades).
- `censhare_keycloak_self_hosted`: Indicates if Keycloak is self-hosted. Default: `true`.

### Database Configuration

- `censhare_keycloak_db_mode`: Specifies whether the database is in a container, hosted on the same machine, or delegated elsewhere. Options: `container`, `hosted`, `delegated`. Default: `container`.
- `censhare_keycloak_db_data_dir`: Directory for database data storage. Default: `/var/lib/pgsql/data`.
- `censhare_keycloak_db_host`: Hostname of the database server. Default: `postgres`.
- `censhare_keycloak_db_version`: Postgres major version for the bundled DB container. Default: `17`.
- `censhare_keycloak_db_user`: Database username for Keycloak. Default: `keycloak`.
- `censhare_keycloak_db_pass`: Password for the database user. Default: `keycloak`.
- `censhare_keycloak_db_name`: Name of the Keycloak database. Default: `keycloak`.
- `censhare_keycloak_db_uid`: Numeric UID applied to the host PGDATA directory when running Postgres in a container. Default: `999`.
- `censhare_keycloak_db_gid`: Numeric GID applied to the host PGDATA directory when running Postgres in a container. Default: `999`.
- `censhare_keycloak_pgdata_path`: Path inside the container that Postgres uses as `PGDATA`. Default: `/var/lib/postgresql/data`.

### Keycloak Realm Configuration

- `censhare_keycloak_realm`: Name of the default Keycloak realm. Default: `censhare`.
- `censhare_keycloak_realms`: List of realm configuration dictionaries. The first (default) entry mirrors the legacy single variables, so existing inventories continue to work. Each entry supports `name`, `display_name`, `svc_user`, `svc_pass`, `client_secret`, `desktop_secret`, `smtp_host`, `smtp_user`, `smtp_pass`, `smtp_from`, `smtp_from_display_name`, `smtp_reply_to`, `smtp_port`, `smtp_starttls`, `smtp_ssl`, and `smtp_auth`.
- `censhare_keycloak_realms_defaults`: Helper variable that contains the role's default list. Reference it if you only want to append or extend the shipped configuration, e.g. `censhare_keycloak_realms: "{{ censhare_keycloak_realms_defaults + my_extra_realms }}"`.

### Keycloak User Configuration

- `censhare_keycloak_admin_user`: Admin username for Keycloak. Default: `admin`.
- `censhare_keycloak_admin_pass`: Admin password for Keycloak. Default: `secret`.
- `censhare_keycloak_svc_user`: Service user for Keycloak. Default: `keycloak-service`.
- `censhare_keycloak_svc_pass`: Password for the service user. Default: `service_password`.
- `censhare_keycloak_realm_roles`: List of roles to be to be assigned for the censhare Service user.

### Keycloak Client Configuration

- `censhare_keycloak_client_redirect_uris`: Final list of redirect URIs configured for the censhare client. Defaults to the `*_default` list (https/http wildcard entries) plus `censhare_keycloak_client_redirect_uris_extra` so you can append custom callbacks.
- `censhare_keycloak_client_secret`: Secret for the Keycloak client. Default: `client_secret`.
- `censhare_keycloak_desktop_secret`: Secret for the desktop client. Default: `desktop_secret`.

### HTTPS / TLS

- `censhare_keycloak_tls_enabled`: When `true`, the role wires Keycloak to serve HTTPS directly from the container. A PKCS#12 keystore is mounted into the container and the proper `KC_HTTPS_*` options are set. Default: `false`.
- `censhare_keycloak_http_enabled`: Controls whether the HTTP listener stays enabled alongside HTTPS (defaults to `true` for backwards compatibility). If you disable it, the role waits for the HTTPS port instead.
- `censhare_keycloak_tls_certificate_src` / `censhare_keycloak_tls_private_key_src`: Optional controller-side files that should be copied to the host. If you omit both certificate and key, the role generates a self-signed pair on the fly (see `censhare_keycloak_tls_self_signed_subject`, `censhare_keycloak_tls_self_signed_sans`, `censhare_keycloak_tls_self_signed_valid_days`).
- `censhare_keycloak_tls_certificate_content` / `censhare_keycloak_tls_private_key_content`: Inline PEM alternatives to the `_src` options.
- `censhare_keycloak_https_host_port` / `censhare_keycloak_https_container_port`: Host and container port for HTTPS (default `8443:8443`). HTTP ports are managed via `censhare_keycloak_http_host_port` and `censhare_keycloak_http_container_port`.

### Keycloak SMTP Configuration

- `censhare_keycloak_smtp_host`: mail.example.com
- `censhare_keycloak_smtp_user`: mailuser
- `censhare_keycloak_smtp_pass`: mailpassword
- `censhare_keycloak_smtp_from`: mailuser@example.com
- `censhare_keycloak_smtp_from_display_name`: "censhare Keycloak"
- `censhare_keycloak_smtp_reply_to`: "no-reply@example.com"

## Dependencies

There are no external role dependencies. However, this role requires standard system packages (Podman, runc, etc.) which the role installs/configures automatically when `censhare_keycloak_self_hosted` is `true`.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters):

```yaml
---
- name: "Setup Keycloak"
  hosts: keycloak
  become: true
  roles:
    - role: ahu_services.censhare.censhare_keycloak
      vars:
        censhare_keycloak_admin_user: "admin"
        censhare_keycloak_admin_pass: "admin_password"
        censhare_keycloak_db_user: "keycloak_user"
        censhare_keycloak_db_pass: "keycloak_password"
        censhare_keycloak_domain: "auth.example.com"
        censhare_keycloak_client_secret: aqu9Iexeng6ahqu6IeTogheiT7mah2tu
        censhare_keycloak_desktop_secret: ohyoh6Sungah7Aemeashaitai7caethe
        censhare_keycloak_additional_realms:
          - name: censhare-dev
            svc_user: keycloak-dev-service
            svc_pass: dev_service_password
            client_secret: new_client_secret
            desktop_secret: new_desktop_secret
            smtp_host: mail.dev.example.com
            smtp_user: ""
            smtp_pass: ""
            smtp_from: dev@example.com
            smtp_from_display_name: censhare Dev Keycloak
            smtp_reply_to: no-reply-dev@example.com
            smtp_port: "587"
            smtp_starttls: "true"
            smtp_ssl: "true"
            smtp_auth: "true"
        censhare_keycloak_realms: "{{ censhare_keycloak_realms_defaults + censhare_keycloak_additional_realms }}"
```

License
-------

MIT

Author Information
------------------

This role was created in 2024 by Andreas Hubert. For more information, contact andreas.hubert@ahu.services
