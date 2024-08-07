# censhare Keycloak Ansible Role

This Ansible role installs and configures Keycloak as the authentication service for censhare Systems. It includes configuration for managing realms, clients, users, and SMTP settings.

## Requirements

- Ansible 2.1 or higher.
- Target hosts should be running a compatible Linux distribution with Podman and the required images available.

## Role Variables

### Keycloak General Configuration

- `censhare_sclient_censhare_version`: The version of Keycloak to install. Default: `24.0.4`.
- `censhare_sclient_self_hosted`: Indicates if Keycloak is self-hosted. Default: `true`.

### Database Configuration

- `censhare_sclient_db_mode`: Specifies whether the database is in a container or hosted externally. Options: `container`, `hosted`. Default: `container`.
- `censhare_sclient_db_data_dir`: Directory for database data storage. Default: `/var/lib/psql/data`.
- `censhare_sclient_db_host`: Hostname of the database server. Default: `postgres`.
- `censhare_sclient_db_user`: Database username for Keycloak. Default: `keycloak`.
- `censhare_sclient_db_pass`: Password for the database user. Default: `keycloak`.
- `censhare_sclient_db_name`: Name of the Keycloak database. Default: `keycloak`.

### Keycloak Realm Configuration

- `censhare_sclient_realm`: Name of the Keycloak realm to be created. Default: `censhare`.

### Keycloak User Configuration

- `censhare_sclient_admin_user`: Admin username for Keycloak. Default: `admin`.
- `censhare_sclient_admin_pass`: Admin password for Keycloak. Default: `secret`.
- `censhare_sclient_svc_user`: Service user for Keycloak. Default: `keycloak-service`.
- `censhare_sclient_svc_pass`: Password for the service user. Default: `service_password`.
- `censhare_sclient_realm_roles`: List of roles to be to be assigned for the censhare Service user. 

### Keycloak Client Configuration

- `censhare_sclient_client_secret`: Secret for the Keycloak client. Default: `client_secret`.
- `censhare_sclient_desktop_secret`: Secret for the desktop client. Default: `desktop_secret`.

### Keycloak SMTP Configuration

- `censhare_sclient_smtp_host`: mail.example.com
- `censhare_sclient_smtp_user`: mailuser
- `censhare_sclient_smtp_pass`: mailpassword
- `censhare_sclient_smtp_from`: mailuser@example.com
- `censhare_sclient_smtp_from_display_name`: "censhare Keycloak"
- `censhare_sclient_smtp_reply_to`: "no-reply@example.com"

## Dependencies

There are no external role dependencies. However, this role requires several base packages to be present on the system, which are typically covered under the installation tasks within the role.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters):

```yaml
---
- name: "Setup Keycloak"
  hosts: keycloak
  become: true
  roles:
    - role: ahu_services.censhare.sclient
      vars:
        censhare_sclient_admin_user: "admin"
        censhare_sclient_admin_pass: "admin_password"
        censhare_sclient_db_user: "keycloak_user"
        censhare_sclient_db_pass: "keycloak_password"
        censhare_sclient_domain: "auth.example.com"
        censhare_sclient_client_secret: aqu9Iexeng6ahqu6IeTogheiT7mah2tu
        censhare_sclient_desktop_secret: ohyoh6Sungah7Aemeashaitai7caethe
```

License
-------

BSD

Author Information
------------------

This role was created in 2024 by Andreas Hubert. For more information, contact andreas.hubert@ahu.services
