# censhare Server Ansible Role

This Ansible role installs and configures the censhare server, setting up necessary components like the Core Cloud Gateway, Static Resource Server, and the relevant databases and web applications. It's designed for environments requiring automated, consistent deployments of censhare applications.

## Requirements

- Ansible 2.1 or higher.
- Access to a repository containing the censhare packages.
- The target servers must have access to the Internet to fetch dependencies unless all required packages are hosted internally.
- Python3 and `dnf` module installed on the target machines for managing packages.

## Role Variables

This role uses several variables to facilitate its configuration and ensure adaptability to different environments and use cases. Below are the categories of variables you'll need to understand and potentially modify:

### Mandatory Variables

These variables must be set for the role to function correctly. They are crucial for accessing the repositories containing the necessary software packages:

- `censhare_repo_user`: The repository username for accessing the censhare software packages.
- `censhare_repo_pass`: The repository password.

### Commonly Modified Variables

In a typical production environment, you may need to adjust the following variables to suit your specific setup:

- `censhare_db_host`: Specifies the hostname of the database server.
- `censhare_db_user`: Database username for accessing your PostgreSQL database.
- `censhare_db_pass`: Password for the database user.
- `censhare_keycloak_domain`: The domain where Keycloak for censhare is hosted.
- `censhare_allowed_origins`: Specifies the origins allowed for cross-origin resource sharing (CORS).
- `censhare_jvm`: Defines the Java Virtual Machine settings such as memory allocation.

### Version Control Variables

These variables allow the control of specific software versions to be installed. It's important to keep these up-to-date or align them with the versions you require:

- `censhare_version`: Main version of the censhare server software, e.g., "2023.1.2".
- `censhare_cgw_version`: Version of the censhare Core Cloud Gateway, e.g., "3.1.9-1".
- `censhare_srs_version`: Version of the censhare Static Resource Server, e.g., "3.0.5-1".

### Multi-Server Configuration

For setups involving multiple servers, each server must be uniquely identifiable:

- `censhare_css_id`: A unique identifier for each server instance in a multi-server environment.

These variables can be set directly in the playbook, included via external variables files, or managed through group_vars/host_vars depending on your organizational practices.

## Dependencies

There are no external role dependencies. However, this role requires several base packages to be present on the system, which are typically covered under the installation tasks within the role.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters):

```yaml
- hosts: censhare_servers
  roles:
     - { role: ahu_services.censhare.server, censhare_db_host: 'db.example.com' }
```

License
-------

BSD

Author Information
------------------

This role was created in 2024 by Andreas Hubert. For more information, contact andreas.hubert@ahu.services
