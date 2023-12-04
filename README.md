# [Ansible role consul](#consul)

Install and configure hashicorp consul

|GitHub|Downloads|Version|
|------|---------|-------|
|[![github](https://github.com/mullholland/ansible-role-consul/actions/workflows/molecule.yml/badge.svg)](https://github.com/mullholland/ansible-role-consul/actions/workflows/molecule.yml)|[![downloads](https://img.shields.io/ansible/role/d/mullholland/consul)](https://galaxy.ansible.com/mullholland/consul)|[![Version](https://img.shields.io/github/release/mullholland/ansible-role-consul.svg)](https://github.com/mullholland/ansible-role-consul/releases/)|
## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/mullholland/ansible-role-consul/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  vars:
    consul_config_sections:
      - name: base
        config: |
          datacenter     = "dc1"
          node_name      = "{{ inventory_hostname_short }}"
          data_dir       = "{{ consul_data_path }}"
          client_addr    = "{{ consul_bind_addr }}"
          bind_addr      = "{{ consul_bind_addr }}"
          advertise_addr = "{{ consul_bind_addr }}"
      - name: UI
        config: |
          ui_config{
            enabled = true
          }
      - name: server
        config: |
          server = {{ consul_server | lower }}
          retry_join = ["{{ consul_bind_addr }}"]
          bootstrap_expect = 1
      - name: logging
        config: |
          log_level            = "INFO"
          log_json             = false
          log_file             = "{{ consul_log_path }}/consul.log"
          # rotate after 24h
          log_rotate_duration  = "24h"
          # keep 1 week worth of logs
          log_rotate_max_files = 7
      - name: telementry
        config: |
          telemetry {
            disable_hostname           = true
            prometheus_retention_time  = "60s"
          }

  roles:
    - role: "mullholland.consul"
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/mullholland/ansible-role-consul/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: true

  tasks:
    - name: Debian/Ubuntu | Install support packages for testing
      package:
        name:
          - "iproute2"  # for ansible_default_ipv4.address
          - "cron"  # for backup cron
          - "findutils"  # for backup cron
        state: latest
      when: ansible_os_family == "Debian"

    - name: RedHat/CentOS | Install support packages for testing
      package:
        name:
          - "iproute"  # for ansible_default_ipv4.address
          - "cronie"  # for backup cron
          - "findutils"  # for backup cron
        state: latest
      when: ansible_os_family == "RedHat" or ansible_os_family == "Rocky"

  roles:
    - role: mullholland.repository_hashicorp
```



## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/mullholland/ansible-role-consul/blob/master/defaults/main.yml):

```yaml
---
# Is this server a consul server?
# agents need this as `false`
consul_server: true

# System user and group (from package)
consul_user: "consul"
consul_group: "consul"

# Packages
consul_packages:
  - consul
  # - consul-enterprise

# set consul license variable
# consul_license: "12345679987654321"

# Bind addr
consul_bind_addr: "{{ ansible_default_ipv4.address | default(ansible_all_ipv4_addresses[0]) }}"

# Paths
consul_config_path: "/etc/consul.d"
consul_data_path: "/opt/consul/data"
consul_log_path: "/var/log/consul"
consul_run_path: "/var/run/consul"

# Create consul configuration
# Sections are only for your personal structure
# they have no effect on the configuration
consul_config_sections:
  - name: base
    config: |
      datacenter     = "dc1"
      node_name      = "{{ inventory_hostname_short }}"
      data_dir       = "{{ consul_data_path }}"
      client_addr    = "{{ consul_bind_addr }}"
      bind_addr      = "{{ consul_bind_addr }}"
      advertise_addr = "{{ consul_bind_addr }}"
  - name: UI
    config: |
      ui_config{
        enabled = true
      }
  - name: server
    config: |
      server = {{ consul_server | lower }}
      retry_join = ["{{ consul_bind_addr }}"]
      bootstrap_expect = 1
  - name: logging
    config: |
      log_level            = "INFO"
      log_json             = false
      log_file             = "{{ consul_log_path }}/consul.log"
      # rotate after 24h
      log_rotate_duration  = "24h"
      # keep 1 week worth of logs
      log_rotate_max_files = 7
  - name: telementry
    config: |
      telemetry {
        disable_hostname           = true
        prometheus_retention_time  = "60s"
      }
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/mullholland/ansible-role-consul/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[mullholland.repository_hashicorp](https://galaxy.ansible.com/mullholland/repository_hashicorp)|[![Build Status GitHub](https://github.com/mullholland/ansible-role-repository_hashicorp/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-repository_hashicorp/actions)|[![Build Status GitLab](https://gitlab.com/opensourceunicorn/ansible-role-repository_hashicorp/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-repository_hashicorp)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://mullholland.net) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/mullholland/ansible-role-consul/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/mullholland):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/r/mullholland/enterpriselinux)|all|
|[Amazon](https://hub.docker.com/r/mullholland/amazonlinux)|Candidate|
|[Fedora](https://hub.docker.com/r/mullholland/fedora/)|38, 39|
|[Ubuntu](https://hub.docker.com/r/mullholland/ubuntu)|all|
|[Debian](https://hub.docker.com/r/mullholland/debian)|all|

The minimum version of Ansible required is 2.10, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-consul/issues).

## [License](#license)

[MIT](https://github.com/mullholland/ansible-role-consul/blob/master/LICENSE).

## [Author Information](#author-information)

[Mullholland](https://mullholland.net)
