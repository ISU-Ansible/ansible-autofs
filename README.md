

# Autofs [![Build Status](https://travis-ci.org/ISU-Ansible/ansible-autofs.svg?branch=master)](https://travis-ci.org/ISU-Ansible/ansible-autofs)


Documentation generated using [mddoc](https://github.com/raznikk/mddoc).


This ansible role aids in the creation of automount files using either
the autofs service or the systemd init system.


This role works under Enterprise Linux (Red Hat, CentOS, Scientific Linux)
and the Fedora environments.


To use this role, simply fill out the **systemd_mounts** or **autofs_mounts**
variables in your variable list. The mounts are a set of key/value pairs
defined as in the example below.


There are 5 options used in the **systemd_mounts** file:


| Option        | Description                                              |
| ------------- | -------------------------------------------------------- |
| **share**     | The location of the remote share using normal notation   |
| **mount**     | The local mount-point of the remote filesystem           |
| **type**      | Mount type                                               |
| **options**   | Mount options (lists will be joined together by a comma) |
| **automount** | Whether or not this will be an automount filesystem      |


The autofs dictionary is the same as the systemd file without the
**automount** mount option.


To enable the mounts on your system, just populate the
**systemd_mounts_enabled** list with the appropriate dictionary key of the
mount and run the mount.


This role also installs the following:


* nfs-utils
* nfs4-acl-tools
* cifs-utils


**Note**: Any mountpoint enabled for both autofs and systemd **should **be
handled by the autofs daemon, but it is not recommended to enable both
at the same time as this is not supported.




## Default Variables


### Usage Variables


* **use_systemd_mounts**: by default, set to false. Set this to true to enable the systemd task list.
* **use_autofs_mounts**: by default, set to false. Set this to true to enable the autofs task list.
* **systemd_mounts**: This variable is a dictionary of systemd mounts for the system.
* **autofs_mounts**: This variable is a dictionary of autofs mounts for the system.
* **systemd_mounts_enabled**: This variable should be a list of mounts to enable via systemd
* **autofs_mounts_enabled**: This variable should be a list of mounts to enable via autofs


### Other Variables


The following variables are used in testing.


* **systemd_mounts_manage_service**
* **systemd_mounts_allow_reload**
* **autofs_mounts_manage_service**
* **autofs_mounts_allow_reload**


### Examples


    use_systemd_mounts: true
    systemd_mounts:
      Mount1:
        share: //server/service$
        mount: /somemountpoint
        type: cifs
        options:
          - domain=local
          - username=user
          - password=pass
          - uid=1000
          - gid=1000
        automount: true
    systemd_mounts_enabled:
      - Mount1


    use_autofs_mounts: true


    autofs_mounts:
     Mount2:
       share: someserver:/export
       mount: /someothermountpoint
       type: nfs
       options:
         - uid=1000


    autofs_mounts_enabled:
      - Mount2




## Variables


By default, we set the **systemd_automount_os_supported** and
**autofs_automount_os_supported** variables to be false. This is overridden
in the *RedHat.yml* and *Fedora.yml* files. When other operating systems are
later included, they will have files added to the *vars/* folder


### Fedora Variables


There is currently no difference between the *Fedora.yml* and *RedHat.yml*
files. The only reason we include a separate file is in the event that
Fedora upstream includes some separate file/variable/prerequisite that
causes the role not to run on Fedora.


### Red Hat 7


We currently only support the Red Hat Enterprise Linux 7 OS variant. Red Hat
Enterprise Linux 6 support could be added, but unless this is a highly
requested feature, we intend to only support RHEL 7 and above.




## Tasks


The tasks module includes the following:


* *systemd_mounts.yml*
* *autofs_mounts.yml*


Each include is associated with a **use** variable that must be enabled in
order to include the file -- **use_systemd_mounts** and **use_autofs_mounts**.


To facilitate mounts, the files include the first found of the following
variable files from the *vars/* folder. This is meant to check for the most
specific variable file to the least specific.


* "{{ ansible_distribution }}_{{ ansible_distribution_version | replace('.','_') }}.yml"
* "{{ ansible_distribution }}_{{ ansible_distribution_major_version}}.yml"
* "{{ ansible_distribution }}.yml"
* "{{ ansible_os_family }}_{{ ansible_distribution_version | replace('.','_') }}.yml"
* "{{ ansible_os_family }}_{{ ansible_distribution_major_version }}.yml"
* "{{ ansible_os_family }}.yml"


The inclusion of these variable files  is to facilitate later inclusion of
other operating systems, should the need arise.


After this, we set the following variables based on the
**ansible_virtualization_type** fact:


* **[autofs|systemd]_mounts_manage_service**
* **[autofs|systemd]_mounts_allow_reload**


These variables are set to aid in testing. These variables will be set to true
only if the executing operating system is not running in a docker container.
When these are set to 'true', no handlers will be run. Since testing is through
travis-ci in a docker container, restarting the service would cause the role
to fail the travis checks.


The final check is to assert that the current operating system is supported.
This variable is set within the *vars/* folder, using one of the variable
files included above.


Otherwise, the task list simply installs necessary packages for NFS and CIFS,
then creates the mount/automount files in the appropraite location.




## Handlers


*Reload systemd* handles reloading the systemd daemon (with daemon_reload)
after mounts are established on the system. This should run after any mount
is added to, or removed from, the system.


The *Enable systemd mount* and *Start systemd mount* handlers are run on any
mount that does not have the 'automount' directive set to false, or not set.
Alternatively, the *Start systemd automount* and *Enable systemd automount*
handlers are run on mounts that have the 'automount' directive set to true.
