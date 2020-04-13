Ansible Role: Deploy vms with cloud init image on KVM
=========

This role does deploy vms inside Kvm for lab proposes with cloud-init. 


*Details*
- Download cloud Image.
- Generate cloud init user/meta data and booting iso.
- Increase root storage size.
- Deploy vms.  


Variables:
=========

+ cloudimg_url: Url to download qcow2 image.
+ osversion: OS label for deployment.
+ domain: fqdn configuration. 
+ vm_public_key: SSH Public Key
+ virtual_machines: Definitions for create many virtual machines like name,osvariant,cpu,memory,disk size and network definitions. 
+ enableroot: It enables root authentication, so if does it set in variables, then you need set the root_pwd.
+ root_pwd: root password.



Examples 
---------


In my example I will create a deployment of 2 vms with CentOS7 using bridge0 and insert my ssh public key. 


If you wasn't set the bridge configuration it will config the default bridge from kvm.


Variables: 

```bash 
---
cloudimg_url: https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-1809.qcow2
osversion: centos7
domain: local.lab
vm_public_key: "{{lookup('file','~/.ssh/id_rsa.pub')}}"
enableroot: yes
root_pwd: redhat
virtual_machines:
  - name: lab1
    osvariant: rhel7
    cpu: 1
    mem: 512
    disk: 15G
    bridge: bridge0
    net:
      ip: 192.168.15.201
      mask: 255.255.255.0
      gateway: 192.168.15.1
      domain: local.lab
      dns: 8.8.8.8
  - name: lab2
    osvariant: rhel7
    cpu: 1
    mem: 512
    disk: 15G
    bridge: bridge0
    net:
      ip: 192.168.15.202
      mask: 255.255.255.0
      gateway: 192.168.15.1
      domain: local.lab
      dns: 8.8.8.8
```

Playbook example : 


```bash 

- name: Create KVM Vms
  hosts: kvm
  become: true
  roles:
    - ansible-kvm-cloudinit-prosivion

```


License
-------
BSD

Author Information
------------------

This role was created in 2020 by [Fabio Abreu Reis](http://github.com/fabioabreureis).
