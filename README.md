Ansible Role: Deploy vms with cloud init image on KVM
=========

This role does deploy vms inside Kvm for lab proposes with cloud-init. 


*Details*

- Download cloud Image.
- Generate cloud init user/meta data and booting iso.
- Increase root storage size.
- Deploy vms with volumes. 
- Deploy a DNS Server with Bind. 
- This was tested with :
    + Ubuntu: 16 and 18
    + RHEL OS based on 6 and 7 version (CentOS and Fedora).  



Requirements:
=========

This playbook was tested with Ansible 2.7 and 2.9.



Variables:
=========

+ cloudimg_url: Url to download qcow2 image.
+ osversion: OS label for deployment.
+ domain: fqdn configuration. 
+ vm_public_key: SSH Public Key
+ virtual_machines: Definitions for create many virtual machines like name,osvariant,cpu,memory,disk size,network and storage definitions. 
+ enableroot: It does enable root authentication.


How to : 
---------


In my example I will create a deployment of 3 vms with CentOS7 using bridge0, some volumes and insert my ssh public key. 

If you wasn't set the bridge configuration it will config the default bridge from kvm.


After clone this repo update the inventory file : 

```bash 
vi inventory/hosts
...


[kvm]
kvm.example.com

[nameserver]
192.168.15.199

...
```


Copy the kvm-sample.yml for kvm hostname: 

```bash 

cp inventory/host_vars/kvm-sample.yml inventory/host_vars/kvm.example.com.yml


```

kvm.example.com.yml variables:

So it's important put the correct osvariant because this variable is important for virt-install command. 

```bash 
---
cloudimg_url: https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-1809.qcow2
osvariant: centos7.0 
domain: local.lab
vm_public_key: "{{lookup('file','~/.ssh/id_rsa.pub')}}"
enableroot: yes
root_pwd: redhat
virtual_machines:
  - name: domain
    cpu: 1
    mem: 1024
    disk: 15G
    bridge: bridge0
    net:
      ip: 192.168.15.199
      mask: 255.255.255.0
      gateway: 192.168.15.1
      dns: 192.168.15.199
  - name: lab1
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
    volumes:
      - device: vdb
        size: 5G
        type: raw
      - device: vdc
        size: 5G
        type: raw
      - device: vdd
        size: 5G
        type: raw
  - name: lab2
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
    volumes:
      - device: vdb
        size: 2G
        type: raw
```

Execute the site.yml playbook : 


```bash 

ansible-playbook -i inventory/hosts site.yml 

``` 


Setup the dns.yml like this configuration bellow: 

```bash
- hosts: nameserver
  vars_files:
    - inventory/host_vars/kvm.example.com.yml
  roles:
    - role: ansible-bind
      when: dnsconfig is defined and dnsconfig == 'ok'
```

Remember the DNS server setup it's only for laboratory proposes.

```bash 

ansible-playbook -i inventory/hosts dns.yml

``` 


Remove vms :
---------

If you need to destroy the vms run this playbook bellow, so it will undefine, destroy and remove vms disks. 


Run the destroy.yml playbook and confirm the operation. 

```bash

ansible-playbook -i inventory/hosts destroy.yml

```



License
-------
BSD

Author Information
------------------

This role was created in 2020 by [Fabio Abreu Reis](http://github.com/fabioabreureis).
