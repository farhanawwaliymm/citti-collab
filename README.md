# Installation Postgresql DB Instanzen
Das Playbook für die Installation einer PostgreSQL DB berücksichtig, dass die Installation nach Instanzen erfolgt.
Wird kein Instanzname angegeben wird die DB als Default in der "pgsql" Instanz angelegt
```
default:
/data/database/pgsql/pgsql
/data/logs/pgsql/pgsql
/data/backup/pgsql/pgsql
```
```
Instance:
/data/database/pgsql/$INSTANCE$
/data/logs/pgsql/$INSTANCE$
/data/backup/pgsql/$INSTANCE$
```

# host_vars
Für die Installation einer Postgresql DB müssen die host_vars entsprechend vorbereitet werden um hier auch nach Instanze
installieren zu können. Erweiterung der postgresql.conf oder der pg_hba.conf wird in den Host Variablen des Servers hinterlegt
und zur Playbook laufzeit aktualisiert.

```
postgresql_instances:
  - instance_name: testdb
    postgresql_port: 5432
    postgresql_version: 16
    postgresql_listen_address: '*'
    postgresql_log_directory: /data/logs/pgsql/testdb
    postgresql_backup_directory: /data/backup/pgsql/testdb
    
    pg_hba_entries:
    - "# Application Server "
    - "host    all             all             192.168.10.7/32         scram-sha-256"

  - instance_name: testdb02
    postgresql_port: 5433
    postgresql_version: 16
    postgresql_listen_address: '*'
    postgresql_log_directory: /data/logs/pgsql/testdb02
    postgresql_backup_directory: /data/backup/pgsql/testdb02

    pg_hba_entries:
    - "# Application Server "
    - "host    all             all             192.168.10.7/32         scram-sha-256"
```
# Default settings
Im default, also wenn keine Instanz angegeben wird werden folgende Parameter gesetzt.
Die pg_hba_entries sind default gesetzt und können über die "host_vars" ergänzt werden.
```
# roles/deploy/defaults/main.yml
postgresql_version: 17
postgresql_action: install
postgresql_dir_owner: postgres
postgresql_dir_permissions: "0750"
postgresql_file_permissions: "0644"
postgresql_data_dir: /data/database/pgsql
postgresql_log_dir: /data/logs/pgsql
postgresql_backup_dir: /data/backup/pgsql

instance_defaults:
  postgresql_port: 5432
  postgresql_listen_address: '*'
  postgresql_log_directroy: /data/logs/pgsql/pgsql
  postgresql_backup_directory: /data/backup/pgsql/pgsql
  postgresql_max_connections: 100
  # - Security and Authentication -
  postgresql_ssl: 'off'
  postgresql_ssl_ca_file: ''
  postgresql_ssl_cert_file: ''
  postgresql_ssl_key_file: ''
  postgresql_password_encryption: 'scram-sha-256'

  # - Memory Settings -
  postgresql_shared_buffers: '1GB'
  postgresql_work_mem: '4MB'
  postgresql_maintenance_work_mem: '1024MB'
  postgresql_effective_cache_size: '3GB'
  postgresql_checkpoint_completion_target: '0.9'
  postgresql_wal_buffers: '16MB'
  postgresql_huge_pages: 'off'
  # - Kernel Resource Usage -
  postgresql_max_files_per_process: 1000
  postgresql_shared_preload_libraries: ''
  # - Cost-based Vacuum Delay -
  postgresql_autovacuum: 'on'
  postgresql_autovacuum_naptime: '1min'
  postgresql_autovacuum_vacuum_threshold: 50
  postgresql_autovacuum_analyze_threshold: 50
  postgresql_autovacuum_work_mem: '512MB'
  # - Write Ahead Log -
  postgresql_wal_level: 'replica'
  postgresql_archive_mode: 'off'
  postgresql_archive_command: ''
  postgresql_max_wal_size: '1GB'
  postgresql_min_wal_size: '80MB'
  # - Replication -
  postgresql_max_wal_senders: 10
  postgresql_wal_keep_size: '64MB'
  postgresql_hot_standby: 'on'
  # - Query Tuning -
  postgresql_default_statistics_target: 100
  postgresql_random_page_cost: 1.1
  postgresql_effective_io_concurrency: 200
  postgresql_max_worker_processes: 4
  postgresql_max_parallel_workers_per_gather: 2
  postgresql_max_parallel_workers: 4
  postgresql_max_parallel_maintenance_workers: 2
  # - Logging -
  postgresql_logging_collector: 'on'
  postgresql_log_directory: '/data/logs/pgsql'
  postgresql_log_filename: 'postgresql-%Y-%m-%d_%H%M%S.log'
  postgresql_log_rotation_age: '1d'
  postgresql_log_rotation_size: '0'
  postgresql_log_min_duration_statement: -1
  postgresql_log_line_prefix: '%m [%p] %d %u %a %h %e'
  postgresql_log_timezone: 'UTC'
  postgresql_log_destination: 'csvlog'
  # - Locale and Formatting -
  postgresql_datestyle: 'iso, mdy'
  postgresql_timezone: 'Europe/Berlin'
  postgresql_lc_messages: 'en_US.UTF-8'
  postgresql_lc_monetary: 'en_US.UTF-8'
  postgresql_lc_numeric: 'en_US.UTF-8'
  postgresql_lc_time: 'en_US.UTF-8'
  postgresql_default_text_search_config: 'pg_catalog.german'
  postgresql_custom_settings: ''

postgresql_instances:
  - instance_name: pgsql
    master: true


pg_hba_entries:
  - "# CommVault requires NOT to have peer-auth for user postgres active."
  - "# local   all             postgres                                peer"
  - "### CommVault setting"
  - local   all             all                                     scram-sha-256"
  - host    all             all             127.0.0.1/32            scram-sha-256"
  - host    all             all             ::1/128                 md5"
  - ""
  - "# Allow replication connections from localhost, by a user with the replication privilege."
  - "local   replication     all                                     peer"
  - "host    replication     all             127.0.0.1/32            scram-sha-256"
  - "host    replication     all             ::1/128                 scram-sha-256"
  - ""
  - "### JUMPHOST ACCESS"
  - "host    all             all             172.25.139.21/32         scram-sha-256"
  - "host    all             all             172.25.139.22/32         scram-sha-256"


postgresql_dirs:
  - "{{ postgresql_data_dir }}"
  - "{{ postgresql_log_dir }}"
  - "{{ postgresql_backup_dir }}"

```

# Template postgresql.conf.j2
Die postgresql.conf Datei ist mit variablen konfiguriert die zur Laufzeit des Playbook erstellt, bzw. überschrieben werden können.
Jede einzelene Variable kann über die "host_vars" je nach Instanz individuell gesetzt werden

````
\# PostgreSQL configuration file

\# - Connection Settings -
listen_addresses = '{{ item.postgresql_listen_address | default('*') }}'
port = {{ item.postgresql_port | default(5432) }}
max_connections = {{ item.postgresql_max_connections | default(100) }}

\# - Security and Authentication -
ssl = {{ item.postgresql_ssl | default('off') }}
password_encryption = {{ item.postgresql_password_encryption | default('scram-sha-256') }}

\# - Memory Settings -
shared_buffers = '{{ item.postgresql_shared_buffers | default('1GB') }}'
work_mem = '{{ item.postgresql_work_mem | default('4MB') }}'
maintenance_work_mem = '{{ item.postgresql_maintenance_work_mem | default('256MB') }}'
effective_cache_size = '{{ item.postgresql_effective_cache_size | default('3GB') }}'
maintenance_work_mem = '{{ item.postgresql_maintenance_work_mem | default('256MB') }}'
checkpoint_completion_target = '{{ item.postgresql_checkpoint_completion_target | default('0.9') }}'
wal_buffers = '{{ item.postgresql_wal_buffers | default('16MB') }}'
huge_pages = '{{ item.postgresql_huge_pages | default('off') }}'

\# - Kernel Resource Usage -
max_files_per_process = {{ item.postgresql_max_files_per_process | default(1000) }}
shared_preload_libraries = '{{ item.postgresql_shared_preload_libraries | default('') }}'

\# - Cost-based Vacuum Delay -
autovacuum = {{ item.postgresql_autovacuum | default('on') }}
autovacuum_naptime = '{{ item.postgresql_autovacuum_naptime | default('1min') }}'
autovacuum_vacuum_threshold = {{ item.postgresql_autovacuum_vacuum_threshold | default(50) }}
autovacuum_analyze_threshold = {{ item.postgresql_autovacuum_analyze_threshold | default(50) }}

\# - Write Ahead Log -
wal_level = {{ item.postgresql_wal_level | default('replica') }}
archive_mode = {{ item.postgresql_archive_mode | default('off') }}
archive_command = '{{ item.postgresql_archive_command | default('') }}'
max_wal_size = '{{ item.postgresql_max_wal_size | default('1GB') }}'
min_wal_size = '{{ item.postgresql_min_wal_size | default('80MB') }}'

\# - Replication -
max_wal_senders = {{ item.postgresql_max_wal_senders | default(10) }}
wal_keep_size = '{{ item.postgresql_wal_keep_size | default('64MB') }}'
hot_standby = {{ item.postgresql_hot_standby | default('on') }}

\# - Query Tuning -
default_statistics_target = {{ item.postgresql_default_statistics_target | default(100) }}
random_page_cost = {{ item.postgresql_random_page_cost | default(1.1) }}
effective_io_concurrency = {{ item.postgresql_effective_io_concurrency | default(200) }}
max_worker_processes = {{ item.postgresql_max_worker_processes | default(4) }}
max_parallel_workers_per_gather = {{ item.postgresql_max_parallel_workers_per_gather | default(2) }}
max_parallel_workers = {{ item.postgresql_max_parallel_workers | default(4) }}
max_parallel_maintenance_workers = {{ item.postgresql_max_parallel_maintenance_workers | default(2) }}

\# - Logging -
logging_collector = {{ item.postgresql_logging_collector | default('on') }}
log_directory = '{{ item.postgresql_log_directory | default('pg_log') }}'
log_filename = '{{ item.postgresql_log_filename | default('postgresql-%Y-%m-%d_%H%M%S.log') }}'
log_rotation_age = '{{ item.postgresql_log_rotation_age | default('1d') }}'
log_rotation_size = '{{ item.postgresql_log_rotation_size | default('0') }}'
log_min_duration_statement = {{ item.postgresql_log_min_duration_statement | default(-1) }}
log_line_prefix = '{{ item.postgresql_log_line_prefix | default('%m [%p] %d %u %a %h %e') }}'
log_timezone = '{{ item.postgresql_log_timezone | default('UTC') }}'

\# - Locale and Formatting -
datestyle = '{{ item.postgresql_datestyle | default('iso, mdy') }}'
timezone = '{{ item.postgresql_timezone | default('Europe/Berlin') }}'
lc_messages = '{{ item.postgresql_lc_messages | default('en_US.UTF-8') }}'
lc_monetary = '{{ item.postgresql_lc_monetary | default('en_US.UTF-8') }}'
lc_numeric = '{{ item.postgresql_lc_numeric | default('en_US.UTF-8') }}'
lc_time = '{{ item.postgresql_lc_time | default('en_US.UTF-8') }}'

\# - Miscellaneous -
default_text_search_config = '{{ item.postgresql_default_text_search_config | default('pg_catalog.german') }}'

\# Add any custom settings below this line

{{ item.postgresql_custom_settings | default('') }}
```


# Playbook command inside database/

```
ansible-playbook -i inventory_linux_postgresql.yml -l $hostname$  collections/ansible_collections/ccgroups_compute/database/postgresql.yml --extra-vars "postgresql_action=install" --ask-vault-pass
ansible-playbook -i inventory_linux_postgresql.yml -l $hostname$ collections/ansible_collections/ccgroups_compute/database/postgresql.yml --extra-vars "postgresql_action=uninstall" --ask-vault-pass
```

# Connect to enterprice_nightly with enterpriceuser

```
psql -U enterpriceuser -d enterprice_nightly -h localhost
```
# Switch to PostgreSQL shell as postgres user
```
sudo psql -U postgres
```