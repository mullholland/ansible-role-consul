# [consul](#consul)

**IMPORTANT**
This role will no longer be updated. you can use [robertdebock/ansible-role-consul](https://github.com/robertdebock/ansible-role-consul) as an alternative.

---

|GitHub|GitLab|
|------|------|
|[![github](https://github.com/mullholland/ansible-role-consul/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-consul/actions)|[![gitlab](https://gitlab.com/mullholland/ansible-role-consul/badges/main/pipeline.svg)](https://gitlab.com/mullholland/ansible-role-consul)|

description

## [Role Variables](#role-variables)

These variables are set in `defaults/main.yml`:
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
consul_bin_path: "/usr/bin"
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

consul_config_services:
  - name: web
    config:
```


## [Example Playbook](#example-playbook)

This example is taken from `molecule/default/converge.yml` and is tested on each push, pull request and release.
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

The machine needs to be prepared in CI this is done using `molecule/default/prepare.yml`:
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





## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/mullholland):

-   [debian10](https://hub.docker.com/r/mullholland/docker-molecule-debian10)
-   [debian11](https://hub.docker.com/r/mullholland/docker-molecule-debian11)
-   [ubuntu1804](https://hub.docker.com/r/mullholland/docker-molecule-ubuntu1804)
-   [ubuntu2004](https://hub.docker.com/r/mullholland/docker-molecule-ubuntu2004)
-   [ubuntu2204](https://hub.docker.com/r/mullholland/docker-molecule-ubuntu2204)
-   [centos7](https://hub.docker.com/r/mullholland/docker-molecule-centos7)
-   [centos-stream8](https://hub.docker.com/r/mullholland/docker-molecule-centos-stream8)
-   [amazonlinux](https://hub.docker.com/r/mullholland/docker-molecule-amazonlinux)
-   [rockylinux8](https://hub.docker.com/r/mullholland/docker-molecule-rockylinux8)
-   [almalinux8](https://hub.docker.com/r/mullholland/docker-molecule-almalinux8)

The minimum version of Ansible required is 2.10, tests have been done to:

-   The previous versions.
-   The current version.

This Role has the following additional molecule test scenarios:
-   cluster

Details can be found in ```molecule/```




If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-consul/issues)

## [License](#license)

MIT


## [Author Information](#author-information)

[Mullholland](https://github.com/mullholland)

## [Special Thanks](#special-thanks)

Template inspired by [Robert de Bock](https://github.com/robertdebock)
