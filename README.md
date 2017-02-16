# AutoFS
The autofs role is used to set up the Linux automounter for various network attached directories.

For best results in attaching ISU provided network storage, the system should be domain joined


## Variables

### Global Variables
No global variables are used in this role.

### Role Variables
These roles, for the most part, can be overridden by:

- **autofs_pkgs**: Contains the yum packages necessary for installation
- **autofs_master**: Contains the location of the auto.master file
- **autofs_scripts_source**: Contains the location of any files that should be copied from the role to another location on the client system.
- **autofs_indirect_maps**: Contains definitions for the autofs indirect map files.

#### Indirect Map definitions
Many definitions can be placed into the **autofs_indirect_maps** variable, but only definitions that are also noted in the **enabled_autofs_indirect_maps** variable will be installed on the system.


    autofs_indirect_maps:
      - name: auto.myfiles
        path: "/LAS"
        autofsoptions: "--timeout 1800"
        mounts:
          - name: 'LAS'
            fstype: 'nfs4'
            mountoptions: 'sec=krb5,rw'
            url: 'my.files.iastate.edu:/ifs/isu/LAS'
      - name: auto.home
        path: "/home"
        autofsoptions: "--timeout 1800"
        mounts:
          - name: '*'
            fstype: 'nfs4'
            mountoptions: 'sec=krb5,rw,async'
            url: 'some.server:/path/to/home/&'

    enabled_autofs_indirect_maps:
      - auto.myfiles

### User Variables
- **enabled_autofs_indirect_maps**: List of maps to enable. These should be listed by the ***autofs_indirect_maps.name*** variable.

## Tasks

### Description
This task installs the autofs yum packages and allows you to set up indirect autofs maps for your systems.

### Changed Files
- /etc/auto.master
- /etc/auto.*name* where *name* is a variable in *autofs_indirect_maps.name*

### Installed Programs
- autofs


## Example
The following example includes two indirect map definitions. The **enabled_autofs_indirect_maps** variable controls what maps are included on your machines.

- The auto.myfiles mount is enabled by default for ALL systems in your hosts files
    - This mount point is **excluded** from the system *homes.example.com* because the **enabled_autofs_indirect_maps** variable is overridden to only include the auto.home map.
- The auto.home mount is enabled only on the *homes.example.com* system.

### playbooks/autofs.yml

    ---
    - name: AutoFS Ansible Playbook
      hosts: all
      user: root
      connection: smart

      roles:
        - krb5
        - kinit
        - ldap
        - samba
        - adjoin
        - sssd
        - pam_sss
        - autofs

### inventory/group_vars/all

    autofs_indirect_maps:
      - name: auto.myfiles
        path: "/LAS"
        autofsoptions: "--timeout 1800"
        mounts:
           - name: 'LAS'
             fstype: 'nfs4'
             mountoptions: 'sec=krb5,rw'
             url: 'my.files.iastate.edu:/ifs/isu/LAS'
      - name: auto.home
        path: "/home"
        autofsoptions: "--timeout 1800"
        mounts:
          - name: '*'
            fstype: 'nfs4'
            mountoptions: 'sec=krb5,rw,async'
            url: 'some.server:/path/to/home/&'

    enabled_autofs_indirect_maps:
      - auto.myfiles

### inventory/host_vars/homes.example.com

    enabled_autofs_indirect_maps:
      - auto.home
