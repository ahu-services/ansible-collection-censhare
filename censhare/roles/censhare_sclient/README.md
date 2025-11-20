# censhare Service Client Ansible Role

This role provisions the censhare Service Client stack on Red Hat compatible hosts.
It installs Podman, pulls the required container images, and wires the Collabora CODE sidecar plus the service-client container into Podman Quadlets so systemd manages them natively.

## Requirements

- Ansible 2.14 or newer.
- Target hosts running an EL8/EL9 compatible distribution with systemd.
- Podman packages available from the platform repositories (the role installs them automatically).
- Credentials for the censhare repositories if you intend to pull protected images.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `censhare_sclient_repo_user` / `censhare_sclient_repo_pass` | `rpm_repo_user` / `rpm_repo_pass` | Credentials that are injected into the service-client container as environment variables. |
| `censhare_sclient_censhare_version` | `2023.1.2` | Version string passed to the service-client container. |
| `censhare_sclient_tools_version` | `latest` | Tag to use when pulling `ahuservices/cs-image-tools`. |
| `censhare_sclient_collabora_version` | `latest` | Tag for the Collabora CODE image (default `collabora/code`). |
| `censhare_sclient_collabora_image` | `collabora/code` | Image repository to use for Collabora. |
| `censhare_sclient_collabora_extra_params` | `--o:ssl.enable=false` | Additional arguments passed through the `extra_params` environment variable to Collabora. |
| `censhare_sclient_container_network` | `censhare` | Podman network created for both containers (allows service-client to reach Collabora via container name). |
| `censhare_sclient_svc_user` / `censhare_sclient_svc_pass` | `service-client` / `secret` | Service credentials provided to the container. |
| `censhare_sclient_svc_host` | `censhare-hostname` | Default backend host name the container connects to. |
| `censhare_sclient_office_url` | `http://localhost:9980/cool/convert-to/pdf` | URL the service-client uses for the Collabora conversion endpoint (exposed from the Collabora quadlet onto localhost). |
| `censhare_sclient_instances` | `4` | Number of service-client instances to launch inside the container. |
| `censhare_sclient_volumes` | `['/assets:/assets/']` | List of volume mappings in `HOST:CONTAINER` format. |
| `censhare_sclient_volumes_info` | see defaults | Metadata injected into the service configuration. |

The Collabora quadlet is ordered before the service-client quadlet (via `After=`) so podman starts it first, but there is no hard dependency – if Collabora fails, service-client will still come up.

See `defaults/main.yml` for the full set of tunable parameters.

## Example Playbook

```yaml
- name: Deploy censhare service-client
  hosts: service_clients
  become: true
  vars:
    censhare_sclient_repo_user: "{{ lookup('env', 'CENSHARE_REPO_USER') }}"
    censhare_sclient_repo_pass: "{{ lookup('env', 'CENSHARE_REPO_PASS') }}"
    censhare_sclient_svc_host: "censhare-app.internal"
    censhare_sclient_instances: 8
    censhare_sclient_collabora_version: "23.05"
  roles:
    - ahu_services.censhare.censhare_sclient
```

## License

MIT

## Author Information

Created by Andreas Hubert (andreas.hubert@ahu.services).
