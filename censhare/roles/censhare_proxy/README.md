# censhare Proxy Ansible Role

This Ansible role installs and configures HAProxy for use in a censhare environment. It includes configuration for handling SSL, various backend services, and specific proxy settings for different censhare components.

## Requirements

- Ansible 2.1 or higher.
- Target hosts should be running a compatible Linux distribution with HAProxy available.

## Role Variables

- `censhare_proxy_backends`: Define a list of default backend systems running the services Keycloak, censhare-Server Web, censhare-Server RMI, and censhare Cloud Gateway.
- `censhare_proxy_backends_keycloak`: Define a list of backend servers that provide Keycloak only. This will override the usage for Keycloak from `censhare_proxy_backends`.
- `censhare_proxy_backends_css`: Define a list of backend servers that provide censhare Web only. This will override the usage for censhare Web from `censhare_proxy_backends`.
- `censhare_proxy_backends_rmi`: Define a list of backend servers that provide censhare RMI only. This will override the usage for censhare RMI from `censhare_proxy_backends`.
- `censhare_proxy_backends_cgw`: Define a list of backend servers that provide censhare Cloud Gateway only. This will override the usage for censhare Cloud Gateway from `censhare_proxy_backends`.
- `censhare_proxy_keycloak_port`: Define the default port for Keycloak backend services. Default: `8080`.
- `censhare_proxy_cgw_port`: Define the default port for censhare Cloud Gateway backend services. Default: `8082`.
- `censhare_proxy_css_port`: Define the default port for censhare Web backend services. Default: `9000`.
- `censhare_proxy_rmi_port`: Define the default port for censhare RMI backend services. Default: `30546`.
- `censhare_proxy_timeout_client` / `censhare_proxy_timeout_server`: Global HAProxy client/server timeouts. Default: `1m` each.
- `censhare_proxy_default_redirect`: Path to redirect to from `/` root path. Default: `censhare5/client`.
- `censhare_proxy_ssl`: Which SSL service to use in the backend. Valid options: `self-signed`, `lets-encrypt`, or `commercial`. Default: `self-signed`.
- `censhare_proxy_ssl_domain`: Domain used for SSL certificates.
- `censhare_proxy_domain_pem`: PEM content for the commercial certificate bundle used when `censhare_proxy_ssl` is set to `commercial`.

### Role Variables for use with lets encrypt

- `censhare_proxy_ssl_acme_fingerprint`: The ACME account thumbprint (required when `censhare_proxy_ssl` is `lets-encrypt`). acme.sh does not store it on disk; capture it when registering/updating the account.
- The ACME flow installs acme.sh if missing, ensures `/etc/haproxy/certs` exists, issues a cert for `censhare_proxy_ssl_domain` if none exists, and writes a combined PEM to `/etc/haproxy/certs/{{ domain }}.pem` (HAProxy bind path). HAProxy is reloaded after install.

To obtain the thumbprint (once, manually):
1. Install acme.sh (e.g. `curl https://get.acme.sh | sh`).
2. Register your account: `~/.acme.sh/acme.sh --register-account -m you@example.com --server letsencrypt`.
3. Capture the printed `ACCOUNT_THUMBPRINT='...'` from that command (or re-run `--update-account --server letsencrypt --debug` after registration to print it again).
4. Set `censhare_proxy_ssl_acme_fingerprint` to that value in your inventory/vars.

### Stats listener

- `censhare_proxy_stats_enabled`: Toggle the stats listener. Default: `false`.
- `censhare_proxy_stats_port`: Port for the stats listener. Default: `8404`.
- `censhare_proxy_stats_uri`: URI path for stats. Default: `/stats`.
- `censhare_proxy_stats_refresh`: Refresh interval. Default: `10s`.
- `censhare_proxy_stats_user` / `censhare_proxy_stats_pass`: Basic auth credentials. Defaults: `admin` / `admin`.

## Dependencies

There are no external role dependencies. However, this role requires several base packages to be present on the system, which are typically covered under the installation tasks within the role.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters):

```yaml
---
- name: "Setup HAProxy"
  hosts: loadbalancer
  roles:
    - role: ahu_services.censhare.censhare_proxy
      vars:
        censhare_proxy_backends:
          - name: censhare-server.internal
            address: censhare-server.internal
        censhare_proxy_backends_keycloak:
          - name: auth.internal
            address: auth.internal
        censhare_proxy_ssl: commercial
        censhare_proxy_ssl_domain: censhare.example.com
```

License
-------

BSD

Author Information
------------------

This role was created in 2024 by Andreas Hubert. For more information, contact andreas.hubert@ahu.services
