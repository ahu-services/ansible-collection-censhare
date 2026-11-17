# Changelog

## 1.5.0 - unreleased

### censhare_server

- CGW template aligned with censhare-core-cloud-gateway 4.1.6. Because the template defines `spring.cloud.gateway.routes` it replaces the routes shipped in the CGW jar, so the new defaults were missing: added the CI Hub route `/cihub/**`, uploads now match `/upload/**` and keep the sub path (`/webserver/upload/<segment>`, the bare `/upload` still works), the SRS routes get the `RemoveRequestHeader=Cookie` / `CENSHARE_USER_LOGIN` filters, and the style guide route uses `${CGW_HOST_PORT}` from the new `censhare_server_cgw_host_port` (default `localhost:8082`).
- New: on `censhare_server_db_type: oracle` the role installs the Oracle Instant Client (`basic` + `sqlplus`, optionally `tools` for `expdp`/`impdp`/`sqlldr`) from yum.oracle.com, so the `corpus` user can inspect the database with `sqlplus`. Repository and GPG key are derived from the EL major version; Oracle's own release RPM is not used because its baseurl depends on the Oracle-Linux-only `$ociregion`/`$ocidomain` dnf variables.
- New `censhare-sqlplus` helper (installed to `/usr/local/bin` and linked into `~corpus/localbin`, which is in the corpus PATH). It reads the credentials from the censhare database service config and feeds the `CONNECT` to SQL*Plus through a named pipe, so the password never reaches the process list, the shell history or a file. Handles both JDBC URL forms and keeps stdin on the terminal, so interactive sessions behave natively.
- `/etc/profile.d/oracle-instantclient.sh` sets `NLS_LANG=AMERICAN_AMERICA.AL32UTF8` (the client default US7ASCII mangles UTF-8 content of a censhare schema), a sortable `NLS_DATE_FORMAT`, and `ORACLE_PATH`/`SQLPATH` for the shipped `login.sql`. `ORACLE_HOME` and `LD_LIBRARY_PATH` are deliberately not set.

## 1.4.1 - 2026-09-13

### censhare_server

- Fix: `#jinja2:` template headers set `lstrip_blocks`/`trim_blocks` as strings (`"True"`); ansible-core 2.19 rejects that with `TemplateOverrides.lstrip_blocks must be <class 'bool'>`, which broke the mail and filesystem service templates. Now booleans, works on 2.16 as well.

## 1.4.0 - 2026-09-13

### censhare_server

- New versioned templates `2026.1/`, aligned with the shipped defaults of censhare-Server 2026.1.0-60 (launcher: Feature.OS/Marmind/Syndic8 properties; server: video-ai and ai-sovereign flags; services unchanged in structure), for censhare 2026.1 on Java 25: the launcher no longer sets `java.security.manager` / `java.security.policy` / `java.security.properties` (the Security Manager was removed from the server) and sets `jdk.xml.elementAttributeLimit=10000`.
- New `censhare_server_db_type` (`postgres` default, `oracle`). `censhare_server_db_port` and `censhare_server_db_url` derive from it (Oracle: `jdbc:oracle:thin:@//host:1521/service`). With `oracle` the role skips psycopg2 and the PostgreSQL schema script, runs `CheckJDBC.sh` and fails early when the connection does not work.

### censhare_keycloak

- New `censhare_keycloak_api_timeout` (default 60 s) passed as `connection_timeout` to all Keycloak admin API tasks; the module default of 10 s is too short for realm creation on small instances.
- Fix: the Keycloak version guard used `regex_search` in a `when:` (string result); ansible-core 2.19 requires boolean conditionals and failed the role. Now uses the `match` test.
- New `censhare_keycloak_login_theme` (and per-realm `login_theme`) to select the realm login theme, e.g. the censhare theme.
- Optional LDAP / Active Directory user federation (`censhare_keycloak_ldap_*`), applied to every configured realm, including with `censhare_keycloak_self_hosted: false` against an externally hosted Keycloak. Default mappers for username, email, first and last name.

## 1.3.0 - 2026-09-07

### censhare_keycloak

- Default Keycloak version raised to 26.7.3 (fixes CVE-2026-18963 and the 26.6.2 batch of CVEs).
- Container environment now uses `KC_BOOTSTRAP_ADMIN_USERNAME` / `KC_BOOTSTRAP_ADMIN_PASSWORD`; the pre-26 `KEYCLOAK_ADMIN*` names were deprecated in 26.0. Role variables are unchanged.
- The role now asserts `censhare_keycloak_version >= 26.0` when a numeric tag is pinned and fails with a clear message otherwise. Pin the collection to 1.2.x for older Keycloak releases.
- New `censhare_keycloak_no_log` (default `true`) drives every `no_log` in the role, so module errors can be surfaced with `-e censhare_keycloak_no_log=false`.
- The systemd unit status and container inspect tasks are now `no_log` as well; they previously echoed the admin and database passwords into the play output.
- Realm SMTP settings now send `replyTo` (the key Keycloak expects) instead of `reply_to`, which made every run report a change and dropped the configured reply-to address.
- README default version corrected to match `defaults/main.yml`.

## 1.0.0 - 2025-10-31

- Initial public release of the `ahu_services.censhare` collection with roles for censhare_proxy, censhare_server, censhare_keycloak, and censhare_sclient.
