---
title: Ansible Disk Format Playbook
author: K
date: 2022-09-04 14:39:00 +0800
categories: [Infra, Ansbile]
tags: [linux, ansbile, disk format, disk mount]
render_with_liquid: false
---

## Ansible Disk Format and Mount Playbook 
The following ansible playbook can format disk with xfs and automatically mount:
1. Format all disks which are defined in variable "devices" with xfs.
2. Mount and write the info into /etc/fstab. you can see the result using "df -h"
3. Because we use ansbile module "filesystem" and "mount", all opertions are idempotent. That means if the disk is already formated and ansible will ignore and we don't worry the playbook format the disk which has the data and it is safe.

Ansible modules docs:

[`Ansible filesystem`](https://docs.ansible.com/ansible/latest/collections/community/general/filesystem_module.html) 

[`Ansible mount`](https://docs.ansible.com/ansible/latest/collections/ansible/posix/mount_module.html#ansible-collections-ansible-posix-mount-module) 


```yaml
---
- hosts: ntp
  become: yes
  remote_user: supper-user
  vars:
    devices:
      data1:
        src: /dev/sdb
        dir: /data1
      data2:
        src: /dev/sdc
        dir: /var/log/test

  tasks:
    - name: Format disk with XFS
      filesystem:
        fstype: xfs
        dev: "{{ item.value.src }}"
      with_dict: "{{ devices }}"

# /etc/fstab will append the following lines:
# /dev/sdb /data1 xfs defaults 0 0
# /dev/sdc /var/log/test xfs defaults 0 0

    - name: Add the mount point to fstab and mount -a
      mount:
        path: "{{ item.value.dir }}"
        src: "{{ item.value.src }}"
        fstype: xfs
        opts: "defaults"
        state: mounted
      with_dict: "{{ devices }}"
```
