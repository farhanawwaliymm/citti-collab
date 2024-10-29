
# Patroni Ansible Deployment

this playbook is for deploy patroni using Ansible

## User Defined Variable

Variable that can be changed by user. defined by default in playbook.

```yaml
---
port_firewalld:
#port_firewalld for open port using firewalld service, port must be in pair. e.g 8080/tcp or 443/tcp
firewalld_service:
#this variable is for service used for firewall. default value is firewalld.service
firewalld_rich_rule:
#this variable is for if you need to add rich rule to firewalld
chrony_service:
#this is for restart chrony service. default value is chronyd.service
etcd_config_file:
#this variable is for etcd.conf file location. default value is
etcd_service:
#this variable is for etcd service, default value is etcd.service
package_install:
#this variable is for this is list of package that needed to be install to run the service.

postgresql_repo:
#this variable is for postgresql repo that need to be defined
pgdg_repo_extra:
#this variable is for enabling pgdg repo e.g pgdg-rhel8-extras
python3_etcd_package:
#this variable is for installing etcd python package
patroni_pip_etcd:
#this variable is for installing etcd package needed from pip
patroni_directory:
#this variable is for patroni configuration path. default value is /etc/patron
patroni_data_directory:
#this variable is for patroni data directory. default value is /psql/citti/data
patroni_pg_archive_directory:
#this variable is for patroni pg archive directory. default value is /psqlarchive/citti/archive
patroni_config_file_yml:
#this variable is for patroni yml configuration file. default value is /etc/patroni/patroni-citti.yml
patroni_service_file:
#this variable is for patroni service file. default vile is /usr/lib/systemd/system/patroni-citti.service
patroni_service:
#this variable is for activating patroni service. default value is patroni-citti.service

haproxy_config_file:
#this variable is for haproxy configuration file path. default value is /etc/haproxy/haproxy.cfg
haproxy_service:
#this variable is for enable haproxy service. default value is haproxy.service

keepalived_config_file:
#this variable is for keepalive configuration file path. default value is /etc/keepalived/keepalived.conf
keepalived_service:
#this variable is for enable keepalive service. default value is keepalived.service
```