# censhare Ansible Collection

This collection, `ahu_services.censhare`, provides a set of Ansible roles for installing and configuring a complete censhare environment. It is designed to be modular, allowing you to deploy different components of the censhare stack across your infrastructure.

## Requirements

- Ansible 2.15 or higher.
- A Red Hat-compatible Linux distribution (e.g., RHEL, CentOS, Rocky Linux) for the target hosts.

## Installation

To use this collection, install it from Ansible Galaxy:

```bash
ansible-galaxy collection install ahu_services.censhare
```

## Roles

This collection includes the following roles:

- **`censhare_server`**: Installs and configures the core censhare Server stack, including the Core Cloud Gateway (CGW) and Static Resource Server (SRS). It manages repositories, RPMs, firewall rules, SELinux policies, and service configurations.

- **`censhare_keycloak`**: Deploys and configures Keycloak as the authentication service for the censhare environment. It can manage Keycloak and its database in containers and handles realms, clients, and users.

- **`censhare_proxy`**: Sets up HAProxy as a reverse proxy for the censhare services. It includes configurations for SSL termination (self-signed, Let's Encrypt, or commercial), backend service routing, and a statistics listener.

- **`censhare_sclient`**: Provisions the censhare Service Client and a Collabora Online Development Edition (CODE) sidecar using Podman Quadlets. It manages the container images, networking, and systemd integration.

## Example Playbook

Here is a basic example of how to use the roles in this collection to set up a censhare environment. This example assumes you have three groups in your inventory: `censhare_servers`, `keycloak_servers`, and `proxy_servers`.

```yaml
---
- name: "Deploy censhare Server"
  hosts: censhare_servers
  become: true
  roles:
    - role: ahu_services.censhare.censhare_server
      vars:
        censhare_server_repo_user: "{{ lookup('env', 'CENSHARE_REPO_USER') }}"
        censhare_server_repo_pass: "{{ lookup('env', 'CENSHARE_REPO_PASS') }}"
        censhare_server_db_host: "db.internal"
        censhare_server_db_user: "corpus"
        censhare_server_db_pass: "supersecret"
        censhare_server_keycloak_domain: "auth.example.com"

- name: "Deploy Keycloak"
  hosts: keycloak_servers
  become: true
  roles:
    - role: ahu_services.censhare.censhare_keycloak
      vars:
        censhare_keycloak_admin_pass: "a_very_secure_password"
        censhare_keycloak_domain: "auth.example.com"

- name: "Configure HAProxy"
  hosts: proxy_servers
  become: true
  roles:
    - role: ahu_services.censhare.censhare_proxy
      vars:
        censhare_proxy_backends_css:
          - name: censhare-app-1
            address: 10.0.1.10
        censhare_proxy_backends_keycloak:
          - name: keycloak-1
            address: 10.0.2.10
        censhare_proxy_ssl: "lets-encrypt"
        censhare_proxy_ssl_domain: "censhare.example.com"
        censhare_proxy_ssl_acme_fingerprint: "YOUR_ACME_THUMBPRINT"

- name: "Deploy censhare Service Client"
  hosts: service_client_nodes
  become: true
  roles:
    - role: ahu_services.censhare.censhare_sclient
      vars:
        censhare_sclient_svc_host: "censhare.example.com"
```

## Testing

Molecule suites were removed to reduce maintenance overhead; no automated integration tests are currently shipped with the collection.

## Development

Run `pre-commit run --all-files` (or install the hook with `pre-commit install`) to execute `ansible-lint` and `ansible-test sanity --requirements` against the collection using ansible-core 2.15. The hook wires the repository into a local collection layout automatically.

## License

This collection is licensed under the MIT License. See the `LICENSE` file for more details.

## Author Information

This collection was created in 2024 by Andreas Hubert. For more information, please contact [andreas.hubert@ahu.services](mailto:andreas.hubert@ahu.services).
