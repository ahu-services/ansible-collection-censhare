# censhare Server Ansible Role

> [!WARNING]
> This role may use default credentials for demonstration and testing purposes. **For production environments, you must override these defaults.** Relying on the default credentials poses a significant security risk. Please refer to the "Role Variables" section to identify and change all default passwords and secrets.

This role installs and configures the censhare Server stack (Server, Core Cloud Gateway, Static Resource Server) on EL8/EL9 hosts. It wires repositories, installs RPMs, opens firewall/Selinux ports, renders censhare XML configs, and starts the supporting services.

## What the role does

- Configures censhare and tools repositories with provided credentials, imports the signing key (EL8), and installs server, CGW, and SRS RPMs.
- Creates censhare sysconfig with `CSS_ID`, renders launcher/server/services XML from versioned templates, and ensures directories under `/opt/corpus/cscs/app/**`.
- Optionally mounts and links filesystems for assets/archive/interfaces; configures Keycloak and mail service XML when enabled.
- Sets SELinux port labels for CGW/SRS when SELinux is enforcing; opens firewall ports (9000, 8081, 8082, 30546) if firewalld is running.
- Deploys webpack artifacts for SRS (download + unarchive) and starts/enables `censhare.core-cloud-gateway` and `censhare.static-resource-server`.
- On Oracle systems, installs the Oracle Instant Client (`sqlplus`) from yum.oracle.com and a `censhare-sqlplus` helper, so the `corpus` user can inspect the database ad hoc.
- Supports remote-service setups (SSH key distribution, keystore generation, template sync) when `censhare_server_remote_server` is true.

## Requirements

- Ansible 2.16 or newer (matches role metadata).
- EL8/EL9 hosts with systemd, Python 3, `dnf`, and outbound access to censhare repositories (or mirrors).
- A privileged user to install RPMs, manage firewall/SELinux, and write to `/opt/corpus`.
- Mandatory: `censhare_server_repo_user` and `censhare_server_repo_pass` must be set; the role fails fast if they are empty.

## Key variables

Versions and repo access:
- `censhare_server_version` (default `2025.2.0`), `censhare_server_cgw_version` (`4.1.2-1`), `censhare_server_srs_version` (`4.0.1-1`) – pin these to your running stack to avoid implicit upgrades when the collection bumps its tested defaults. Versioned XML templates exist for 2023.1 … 2026.1; 2026.1 (Java 25) ships a launcher without the removed Java Security Manager properties and with `jdk.xml.elementAttributeLimit=10000`. Java itself is not installed by the role (2026.1 needs JDK 25, e.g. Amazon Corretto).
- `censhare_server_repo_user` / `censhare_server_repo_pass` (no defaults; required)
- `censhare_server_repo_host`, `censhare_server_repo_path`, `censhare_server_tools_repo_path` control repository URLs

Server/runtime:
- `censhare_server_css_id` (default `master`), `censhare_server_remote_server` (default `false`), `censhare_server_master_server_delegate` (host used for templating/sync)
- `censhare_server_jvm`/`_min`/`_max` (default `10g`) and `censhare_server_allowed_origins`
- Filesystem mounts via `censhare_server_filesystems` (see commented examples in `defaults/main.yml`)

Database:
- `censhare_server_db_type` (`postgres` default, or `oracle`); `censhare_server_db_host`/`_port`/`_name`/`_user`/`_pass`; the JDBC URL (`censhare_server_db_url`) derives from these (Oracle: `jdbc:oracle:thin:@//host:1521/service`, `_name` is the service name)
- PostgreSQL: master node installs `python3-psycopg2`, runs `CheckJDBC.sh` and creates the schema when the `party` table is missing
- Oracle: master node runs `CheckJDBC.sh` and fails when the connection does not work; the schema user must exist beforehand (`css/database/create-user.sql`, on RDS without DATAFILE paths). Tables are created by the server's database update on first start.

Keycloak / Web frontends:
- `censhare_server_keycloak_domain`, `censhare_server_keycloak_uri`, realm/user/client secrets, `censhare_server_host_port`
- `censhare_server_srs_host_port` (default `localhost:8081`) and `censhare_server_cgw_host_port` (default `localhost:8082`): internal host:port of the Static Resource Server and of the Cloud Gateway itself (style guide route). The CGW template carries the full route list (it replaces the CGW defaults), aligned with CGW 4.1.6: CI Hub (`/cihub/**`), uploads with sub paths, cookie filters on the SRS routes, plus one route set per entry of `censhare_server_cgw_brandings`.
- Webpack deploy toggled via `censhare_server_srs_deploy_webpack_artifact` (default `true`); artifact URL uses repo credentials

Oracle client (`censhare_server_db_type: oracle` only):
- `censhare_server_oracle_client_enabled` (default: true when the db type is `oracle`) installs `oracle-instantclient-basic` + `-sqlplus` from `yum.oracle.com` and writes `/etc/profile.d/oracle-instantclient.sh`
- `censhare_server_oracle_client_major` (`23`), `censhare_server_oracle_client_repo_baseurl` / `_repo_gpgkey` (derived from the EL major version; the Oracle release RPM cannot be used on AlmaLinux/Rocky because its baseurl needs the Oracle-Linux-only `$ociregion` variable)
- `censhare_server_oracle_client_tools` (default `false`) adds `oracle-instantclient-tools` with `expdp`/`impdp`/`sqlldr` for migration hosts
- `censhare_server_oracle_client_nls_lang` (`AMERICAN_AMERICA.AL32UTF8`), `_date_format`, `_linesize`, `_pagesize` shape the environment and the `login.sql` in `censhare_server_oracle_client_sqlpath`
- `censhare_server_oracle_client_helper` (`/usr/local/bin/censhare-sqlplus`, linked into `~corpus/localbin`), `censhare_server_oracle_client_verify_connection`

`ORACLE_HOME` and `LD_LIBRARY_PATH` are deliberately not set: the RPM registers its library directory in `/etc/ld.so.conf.d`, and Instant Client runs without an `ORACLE_HOME`.

### censhare-sqlplus

```bash
censhare-sqlplus                              # interactive session
censhare-sqlplus "SELECT count(*) FROM asset;"  # one statement, then exit
censhare-sqlplus -S report.sql                # run a script silently
echo 'SELECT user FROM dual;' | censhare-sqlplus -S
censhare-sqlplus -n                           # show config, user and connect string
```

It reads `connection-url`, `user-name` and `user-password` of the `<connection>` element from
`services/database/config.<CSS_ID>.xml` and feeds the `CONNECT` to SQL*Plus through a named pipe:
the password never appears in the process list, in the shell history or in a file. Both JDBC URL
forms are supported (`@//host:port/service` and the older `@host:port:SID`).

Mail and TLS automation:
- `censhare_server_mail_*` flags/credentials control optional mail service XML
- `censhare_server_certbot_enabled` plus ACME directory/FQDN settings for certbot integration

See `roles/censhare_server/defaults/main.yml` for the full list.

## Example playbook

```yaml
- name: Install censhare Server
  hosts: censhare_servers
  become: true
  vars:
    censhare_server_repo_user: "{{ lookup('env', 'CENSHARE_REPO_USER') }}"
    censhare_server_repo_pass: "{{ lookup('env', 'CENSHARE_REPO_PASS') }}"
    censhare_server_db_host: "db.internal"
    censhare_server_db_user: "corpus"
    censhare_server_db_pass: "supersecret"
    censhare_server_keycloak_domain: "auth.example.com"
    censhare_server_allowed_origins: "censhare.example.com:9000"
    censhare_server_filesystems:
      - name: assets
        usage: assets
        mount: nfs
        src: "10.0.0.10:/srv/assets"
        dest: "/asset"
    censhare_server_remote_server: false
  roles:
    - ahu_services.censhare.censhare_server
```

## Tags

- `css_install` – repo wiring and package installation steps. Use for a fresh install or when updating RPMs.
- `css_server_config` – renders launcher/server XML and related server-level config.
- `css_database_config` – database XML templating and DB checks.
- `css_services_config` – service XML generation (filesystem, Keycloak, mail) and supporting filesystem layout.
- `css_webpack_config` – CGW/SRS templating and webpack artifact deployment.
- `css_update` – idempotent steps that should run on update cycles (RPM updates and config template refresh). Assumes an initial install already created required directories and prerequisites.
- `keycloak_setup` (other role) – applies Keycloak realm/user/client configuration.

## License

MIT

## Author Information

This role was created in 2024 by Andreas Hubert. For more information, contact andreas.hubert@ahu.services
