coffeesprout.zfs
===================

**Expand zpools.**
This role can (if configured) expand zpools on FreeBSD.
The implementation is very naive and will only function with single vdev pools as this is something you often encounter on new virtual servers.

**Setup ZFS datasets.** 
This role can be used multiple times to construct several datastore hierarchies

**Setup snapshots.**
This role will automatically setup snapshots if requested using [zfstools](https://github.com/bdrewery/zfstools) 

Requirements
------------

FreeBSD 11 Base and higher

Role Variables
--------------

    zfs_dataset:
    
Contains all the information invoke the ansible [ZFS module](https://docs.ansible.com/ansible/latest/modules/zfs_module.html)

Example:

    zfs_dataset:
        name: storage2
        extra_zfs_properties:
            quota: 100G
            
This will assume a dataset on top of the default pool, which is:

    zfs_zpool: "zroot"
    
The default zpool to operate on; Each zfs_dataset can optionally override this with *zpool*

    expand_zpools: []
    
Optional list of zpools to expand; FreeBSD only and also requires setting:

    zpool_partition:

The partition containing the zpool. On a default FreeBSD install this is likely "3"    

    zpool_disk:

The disk containing the zpool; Only supports single disk vdevs as commonly found in virtual servers.
Example: vtbd0

Dependencies
------------


Example Playbook
----------------

Multiple datasets in one go:


    - hosts: storage
      roles:
      - role: coffeesprout.zfs
        zpool_disk: vtbd0
        zpool_partition: 3
        zpool: zroot
        zfs_datasets:
        - name: storage
          extra_zfs_properties:
            quota: 100G
            sharenfs: "-alldirs,-maproot=root,-network=10.0.0.0/24"
        - name: storage/logs
          extra_zfs_properties:
            compression: gzip
            quota: 10G
        - name: backups
          zpool: tank1
          extra_zfs_properties:
            quota: 2T



Single dataset:

    - hosts: storage
      roles:
      - role: coffeesprout.zfs
        zfs_dataset:
          name: storage2
          extra_zfs_properties:
            quota: 100G

You probably don't want your datasets to end up on the root, so you can either name them "blah/blah" or specify the parent for programmatic control:

    - hosts: jailserver
      vars:
        jail_dataset: "jail" #zpool defaults to zroot, so this is zroot/jail
        jail_name: somename
        jail_zfs_mounts:
        - name: logs
          extra_zfs_properties:
            mountpoint: "{{ jail_mountpoint }}/var/log"
            compression: gzip
            quota: 10G
      roles:
      coffeesprout.zfs:
        zfs_dataset:
          parents: "{{ jail_dataset }}"
          name: "{{ jail_name }}"
          extra_zfs_properties: "{{ extra_zfs_properties }}"
      coffeesprout.zfs:
         zfs_dataset:
          name: "{{ item.name }}"
          extra_zfs_properties: "{{ item.extra_zfs_properties }}"
          parents: "{{ jail_dataset }}/{{ jail_name }}"
      with_items: "{{ jail_zfs_mounts }}"


License
-------

BSD

Author Information
------------------

You can reach out via Github. Please note I generally build these roles for my use cases and those of our customers.
You're free to fork, but I usually don't accept PR's unless they keep all the defaults intact.
