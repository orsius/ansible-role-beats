# \*beat

This role provides a generic means of installing Elastic supported beats using
Ansible. Currently this includes:

- Filebeat
- Heartbeat
- Packetbeat
- Metricbeat
- Auditbeat

The role installs a single beat. Multiple applications of the role are necessary
for multiple beats on the same host, see sample playbook below.

The logs are wherever the packages put them, which by default means
`/var/log/<beat name>/`.

## Requirements

* Ubuntu 16.04 or Centos 7 on the target host

## Role Variables

### `beats_beat`

- The name of the beat to install.
- Not defined by default. Example values: **filebeat**, **metricbeat**.

### `beats_use_repository`

- Use official Elastic repo for yum or apt if true. If false, you must define
  `beats_custom_package_url`.
- Default value: **true**

### `beats_custom_package_url`

- The URL of an RPM or DEB file to install, in case the Elastic repo isn't used.
  Ignored if `beats_use_repository` is true. Certificates and GPG signature
  not checked.
- Not defined by default.

### `beats_start_service`

- Start beat service after install.
- Default value: **true**

### `beats_restart_on_change`

- Restart service if there was a configuration or installed version change.
- Default value: **true**

### `beats_conf`

- The value of this variable will be the contents of the beat's
  configuration file
- Not defined by default. Must be defined. See playbook below.

### `beats_version`

- The version of the package you want to install. Ignored if
  `beats_use_repository` is false. If not defined, the latest package version
  will be used.
- Not defined by default.

### `beats_version_lock`

This uses the native package manager's capabilities to lock the beat package's
version. It will be unlocked and re-locked during subsequent runs of the role
if needed, to update the beat.

- Locks the version of the installed package.
- Default value: **true**

## Example Playbook
> :information_source: Please refer to our "Tests" folder to get the lastest examples.

Sample playbook (inventory much less relevant):
```
- hosts: all
  become: yes
  roles:
    - ansible-role-beats
  vars:
    beats_beat: filebeat
    beats_conf:
      filebeat:
        prospectors:
          - type: log
            paths:
              - /var/log/*.log
      output:
        elasticsearch:
          hosts:
            - localhost:9200
      logging:
        files:
          rotateeverybytes: 10485760  # 10 Mo
          keepfiles: 7  # 70 Mo total

- hosts: all
  become: yes
  roles:
    - ansible-role-beats
  vars:
    beats_beat: metricbeat
    beats_conf:
      metricbeat:
        modules:
          - module: system
            metricsets:
              - process
            processes:
              - ".*"
      output:
        elasticsearch:
          hosts:
            - localhost:9200
```

### Vagrant and tests

Cd into `tests/xenial/` and `vagrant up` for an instance with all beats active.
You can also choose to cd into `tests/centos-7` and do the same.

It's not very useful, as there is no ES to push the data to.

## License

Apache 2.0

## Maintainer

- [Gautier Franchini](mailto:gautier.franchini@data-essential.com)
- [Spyridon Gouliarmis](mailto:spyridon.gouliarmis@data-essential.com)

This role is a modification of https://github.com/elastic/ansible-beats, by
Dale McDiarmid.
