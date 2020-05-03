Role Name
=========

Ansible role to manage backups + archiving with encryption

Features :
	- local / remote backups 
	- local / remote archiving
	- crontab
	- rsync (through ssh if remote)

It does :
- Create a folder on backup user's home to put scripts and rsync's excludes file
- Create backup/archive folders on target servers if not present
- Create backup/archive folders locally if needed
- Create log folders and files
- Create modular backup/archive shell script + filter file, and copy on target
- Configure cron jobs
- Add remote host key if needed
- Backup and archive any source to any targets

Requirements
------------

Requirements :
- rsync
- ssh
- backup users and groups already created (role users) + privileges (role sudoers)
- borgbackup (for archiving) installed
- borgbackup passphrases defined

Role Variables
--------------

Variables are located in default/main.yml file, for what concerns global engine of this role.
Then, modular variables are directly in the playbooks.

```yaml
---
# defaults file for backup

backup: true

# Backup cron job options
backup_cron_job_state: present
backup_hour: []
backup_minute: []

# User under which backup jobs will run
backup_user: "backup-user"
backup_group: "backup-group"

# Path where scripts will be uploaded
backup_work_path: "/home/{{ backup_user }}/scripts"

# Directories to back up. {{ backup_user }} must have read access to these dirs.
backup_directories: []

# rsync default options
rsync_cmd_option: "-zArhlvptugo"
rsync_delete_option: "--delete "

# Items to exclude from backups. (Use rsync 'excludes' file syntax.)
backup_exclude_items:
  - "*.JPG"
  - "**/tmp_files"
  - "Thumbs.db"

# Options to control where the backup is delivered
target_server: "backup-server@backup.org"
backup_location: []
backup_identifier: "{{ backup_user }}"
backup_base_path: "/media/backup"
backup_remote_connection: "{{ backup_user }}@{{ target_server }}"
backup_full_path: "{{ backup_base_path }}{{ target_component }}/{{ inventory_hostname }}{{ target_dir }}"
backup_remote_connection_ssh_options: '-p 1234 -i /home/{{ backup_user }}/.ssh/id_rsa'
backup_remote_connection_rsync_options: "--rsync-path=\"sudo rsync\""

# Logs
backup_log_path: "/var/log/backup"

# Feature to add server's SSH keys in case not present
backup_remote_host_name: ''
backup_remote_host_key: ''

# Archives
# Directories to back up. {{ archive_user }} must have read access to these dirs.
# archive_directory: []

backup_archive: false
archive_work_path: "/home/{{ archive_user }}/scripts"
archive_base_path: "/media/archive"
archive_full_path: "{{ archive_base_path }}/{{ archive_name }}"
archive_remote_connection: "{{ archive_user }}@{{ target_server }}"
archive_name: []
archive_user: "archive-user"
archive_group: "backuop-group"
archive_cron_job_state: present
archive_hour: []
archive_minute: []
archive_passphrase: "{{ lookup('file', 'borg_passphrases/{{ archive_name }}') }}"
archive_log_path: "/var/log/archive"

# MySQL database backup options
db_dirname: "databases"
dbfiles_dirname: "databases_files"
databases_list: false
databases_location: []
databases_files: "/var/lib/mysql/"
backup_mysql: false
backup_mysql_user: []
backup_mysql_password: []
backup_mysql_credential_file: ''

```

Dependencies
------------

An Ansible Role to deploy users
An Ansible role to deploy sudo provileges

Example Playbook
----------------

Backup from a source host to backup host :

```yaml
- hosts: prod
  become: yes
  gather_facts: false
  roles:
    - backup
  vars:
    backup_name: "my_backup"
    backup_location: remote
    backup_user: "backup-user"
    backup_hour: "4"
    backup_minute: "30"
    target_component: "/my_backup"
    target_dir: "/web"
    backup_directories:
      - /var/www
```

Archive System backups folder locally, and then ship it to archive server :

```yaml
- hosts: backup-server
  become: yes
  gather_facts: false
  roles:
    - backup
  vars:
    backup: false
    archive: true
    archive_location: "local"
    target_server: "archive-server"
    archive_name: "system"
    archive_hour: "01"
    archive_minute: "04"
    archive_directory:
      - /media/backup/system

- hosts: backup-server
  become: yes
  gather_facts: false
  roles:
    - backup
  vars:
    backup_name: "archive_system"
    backup_location: remote
    backup_user: "backup-user"
    backup_hour: "06"
    backup_minute: "04"
    target_server: "archive-server"
    target_component: "/system"
    target_dir: "from_prod"
    backup_directories:
      - /media/archive/system
```

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
