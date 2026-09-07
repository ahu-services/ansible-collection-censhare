# Changelog

## 1.3.0 - 2026-09-07

### censhare_keycloak

- Default Keycloak version raised to 26.7.3 (fixes CVE-2026-18963 and the 26.6.2 batch of CVEs).
- Container environment now uses `KC_BOOTSTRAP_ADMIN_USERNAME` / `KC_BOOTSTRAP_ADMIN_PASSWORD`; the pre-26 `KEYCLOAK_ADMIN*` names were deprecated in 26.0. Role variables are unchanged.
- The role now asserts `censhare_keycloak_version >= 26.0` when a numeric tag is pinned and fails with a clear message otherwise. Pin the collection to 1.2.x for older Keycloak releases.
- New `censhare_keycloak_no_log` (default `true`) drives every `no_log` in the role, so module errors can be surfaced with `-e censhare_keycloak_no_log=false`.
- The systemd unit status and container inspect tasks are now `no_log` as well; they previously echoed the admin and database passwords into the play output.
- README default version corrected to match `defaults/main.yml`.

## 1.0.0 - 2025-10-31

- Initial public release of the `ahu_services.censhare` collection with roles for censhare_proxy, censhare_server, censhare_keycloak, and censhare_sclient.
