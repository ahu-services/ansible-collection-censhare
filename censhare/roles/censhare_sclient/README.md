# censhare Service Client Ansible Role

> [!WARNING]
> This role uses default credentials for demonstration and testing purposes. **For production environments, you must override these defaults.** Relying on the default credentials poses a significant security risk. Please refer to the "Role Variables" section to identify and change all default passwords and secrets.

This role provisions the censhare Service Client stack on Red Hat compatible hosts. It installs Podman, pulls the censhare Service Client and Collabora CODE images, and wires both into Podman Quadlets so systemd manages them natively.

## What the role does

- Installs Podman helper packages (`podman-docker`, `runc`) and enforces `runc` as the default runtime via `/etc/containers/containers.conf`.
- Creates the Quadlet directory (`/etc/containers/systemd` by default) and a Podman network for the Collabora sidecar.
- Pulls the `cs-image-tools` (service-client) and `collabora/code` images with configurable tags.
- Creates and enables two Quadlet units: `collabora` publishes `127.0.0.1:9980` and the service-client runs on the host network with an `After=` dependency on Collabora.
- Sets SELinux context for the ICC profiles directory when enforcement is enabled and keeps both Quadlets restarted by systemd.

## Requirements

- Ansible 2.16 or newer (matches the collection metadata).
- EL8/EL9 compatible hosts with systemd.
- A user with privileges to install packages, manage systemd units, and create Podman networks/containers.
- Credentials for censhare registries if you pull protected images.

## Role variables

Container images and versions:
- `censhare_sclient_tools_image` (default `docker.io/ahuservices/cs-image-tools`)
- `censhare_sclient_tools_version` (default `1.8.1`) – pin this tag in your inventory to avoid pulling a newer collection-tested default on upgrades
- `censhare_sclient_collabora_image` (default `docker.io/collabora/code`)
- `censhare_sclient_collabora_version` (default `25.04.7.3.1`) – also pin if you do not want Collabora to track the collection defaults
- `censhare_sclient_collabora_extra_params` (default `--o:ssl.enable=false`) forwarded to Collabora as `extra_params`

Service Client configuration:
- `censhare_sclient_repo_user` / `censhare_sclient_repo_pass` (default `rpm_repo_user` / `rpm_repo_pass`)
- `censhare_sclient_censhare_version` (default `2025.2.0`) – update/pin to the censhare Server build that matches your environment
- `censhare_sclient_svc_user` / `censhare_sclient_svc_pass` (default `service-client` / `secret`)
- `censhare_sclient_svc_host` (default `censhare-hostname`)
- `censhare_sclient_rmi_port` (default `30550`) fixed RMI callback port exposed to the censhare server
- `censhare_sclient_callback_host` (default to `ansible_default_ipv4.address` fallback to the first IPv4 address or `inventory_hostname`) advertised callback host/IP when running in bridge/NAT mode
- `censhare_sclient_office_url` (default `http://{{ censhare_sclient_collabora_name }}:9980/cool/convert-to/pdf`)
- `censhare_sclient_instances` (default `4`)
- `censhare_sclient_svc_appendix` (default `""`) optional suffix appended to all service-client resources when you run multiple stacks on one host
- `censhare_sclient_collabora_appendix` (default `{{ censhare_sclient_svc_appendix }}`) optional suffix for Collabora resources; inherits the service-client suffix by default
- `censhare_sclient_svc_name` / `censhare_sclient_collabora_name` (Quadlet names; defaults `service-client{{ censhare_sclient_svc_appendix }}` / `collabora{{ censhare_sclient_collabora_appendix }}`)

Host integration:
- `censhare_sclient_quadlet_dir` (default `/etc/containers/systemd`)
- `censhare_sclient_quadlet_wanted_by` (default `multi-user.target`)
- `censhare_sclient_service_unit` / `censhare_sclient_collabora_service_unit` (systemd unit names; default `{{ name }}.service`)
- `censhare_sclient_container_network` (default `censhare{{ censhare_sclient_svc_appendix }}`) used for Collabora and service-client
- `censhare_sclient_firewalld_service` (default `censhare-service-client{{ censhare_sclient_svc_appendix }}`) firewalld service name for the callback port; opened automatically when firewalld is running
- `censhare_sclient_iccprofiles_dir` (default `/opt/iccprofiles`) mounted read-only into the service-client container (with `:Z` when SELinux enforcing)
- `censhare_sclient_volumes` (default `[]`) optional host bind mounts added to the service-client container; `:Z` is appended automatically when SELinux is enforcing and no label is present
- `censhare_sclient_volumes_info` (default `{}`) forwarded as the container env `VOLUMES_INFO` to rewrite `hosts.xml` volume metadata when you mount storage

Networking: the service-client now runs in bridge mode on `{{ censhare_sclient_container_network }}`, publishes `{{ censhare_sclient_rmi_port }}` to the host, and rewrites the RMI stub using `censhare_sclient_callback_host`. Firewalld is adjusted when running to keep only the current port open.

The Collabora Quadlet is ordered before the service-client Quadlet (`After=`) so Podman starts it first. If Collabora fails to start, the service-client still comes up but its conversion endpoint will be unavailable until Collabora is healthy.

See `roles/censhare_sclient/defaults/main.yml` for the full set of tunables.

## Example playbook

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
    censhare_sclient_iccprofiles_dir: "/srv/iccprofiles"
  roles:
    - ahu_services.censhare.censhare_sclient
```

Quick check locally:

```bash
ansible-playbook -i roles/censhare_sclient/tests/inventory roles/censhare_sclient/tests/test.yml --check
```

## License

MIT

## Author Information

Created by Andreas Hubert (andreas.hubert@ahu.services).
