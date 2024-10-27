
# Patroni Ansible Deployment

this playbook is for deploy patroni using Ansible

## User Defined Variable

Variable that can be changed by user. defined by default in playbook.

```yaml
---
port_firewalld: #port_firewalld for open port using firewalld service, port must be in pair. e.q 8080/tcp or 443/tcp
firewalld_service: #service used for firewall e.q firewalld.service
firewalld_rich_rule:  #if you need to add rich rule to firewalld e.q
chrony_service:
etcd_config_file:
etcd_service:
package_install:

postgresql_repo:
pgdg_repo_extra:
python3_etcd_package:
patroni_pip_etcd:
patroni_directory:
patroni_data_directory:
patroni_pg_archive_directory:
patroni_config_file_yml:
patroni_service_file:
patroni_service:

haproxy_config_file:
haproxy_service:

keepalived_config_file:
keepalived_service:
```