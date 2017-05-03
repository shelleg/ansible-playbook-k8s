Kubernetes playbook
===================

This playbook was designed to `get started` with K8s which will give more than the minikube all in one solution.

QuickStart
----------

1. **Vagrant** path

    `vagrant up --no-provision` - will create 4 machines (edit the servers.yml and remove 2 for the minimal installation path)
    `ansible-galaxy install -r reuirements.yml` - get all the needed roles
    `ansible-playbook -i ./inventories/vagrant_hosts ./playbooks/default.yml` - runs ansible with the required vagrant inventory

1. **Custom** path

    1. prepare your custom servers - make sure they are Centos7
    1. Update the ./invntories/hosts file and add the required info
    2. `ansible-galaxy install -r reuirements.yml` - get all the needed roles
    3. `ansible-playbook -i ./inventories/hosts ./playbooks/default.yml` - runs ansible with the required vagrant inventory

Upon completion of Ansible execution you can login (vagrant ssh || ssh k8s-master01) and run `kubectl get nodes` to find the list of nodes managed by the k8s master.

Caviates
--------

* The playbook heavy relies on the `inveontory file` and `fact caching`
* `inveontory file` and coresponding nodes require 2 groups
    * k8s_masters - the list of master machines [ only tested with 1 ATM ...]
    * k8s_minions - the group of minions / worker nodes
* `fact caching` - in order to be able to update the /etc/hosts file on each machine I needed to store the Ansible and Host facts locally, this is achived by a small configuration on the `ansible.cfg` file.

Requirements
------------
* 2-n Virtual / cloud instances
* Ansible 2.2.2.0
* If using Vagrant path: 
    * Vagrant (1.9.4) 
    * Virtaulbox (5.1.22 r115126)
* If using custom infra path you will probebly need an account and instances and add them to the invntory


Ansible Invntory options
------------------------

**_Custom inventory_**

```yamlex
[k8s_masters]
k8s-master01 ansible_host=hagzag1.mydomain.com

[k8s_minions]
k8s-minion01 ansible_host=hagzag2.mydomain.com
k8s-minion02 ansible_host=hagzag3.mydomain.com

[localhost]
localhost ansible_connection=local

[all:vars]
ansible_user="user"
ansible_ssh_private_key_file="~/.ssh/id_rsa"
```    
**_Vagrant inventory_**

```yamlex
[k8s_masters]
k8s-master01 ansible_host=172.16.1.2

[k8s_minions]
k8s-minion01 ansible_host=172.16.1.3
k8s-minion02 ansible_host=172.16.1.4
k8s-minion03 ansible_host=172.16.1.5

[localhost]
localhost ansible_connection=local

[all:vars]
ansible_user="vagrant"
ansible_ssh_private_key_file=".vagrant/machines/{{ inventory_hostname }}/virtualbox/private_key"
```

See the ./inventories firecroty in this repo for these exmaples + the Vagrantfile and servers.yml which will enable you to create these machines for the inventory which is defined in ./invnetories/vagrant_hots 


Roles in use
------------

* external see requirements.yml for details:
    * `shellg.ntp` nodes time syncronization
    * `andrewrothstein.kubectl` installes kubectl on your controller | administration node
* internal see ./site-roles directory
    * `ansible-role-k8s-common` - common stuff such as set hostnames and add entroes to /etc/hosts on each node in the cluster
    * `ansible-role-k8s-base` - consists of 2 set of tasks 1 for masters 1 for minions 
    * `ansible-role-kubectl-cfg` - configure kubectl - this is used to setup a control machine to manage the k8s cluster
    * `ansible-role-inventory-init` - this was desigend to generate an Ansible inveontory I will re-visit this at a later time once we have support for more that these 2 implementations 
    
    
Example playbook
----------------

See the ./playbooks/default.yml
```yamlex
---
- hosts: all:!localhost
  become: true
  tasks:

- hosts: all:!localhost
  become: true
  roles:
    - role: shelleg.ntp
    - role: ansible-role-k8s-common

- hosts: k8s_masters
  become: true
  roles:
    - role: ansible-role-k8s-base
    - role: ansible-role-kubectl-cfg

- hosts: k8s_minions
  become: true
  roles:
    - role: ansible-role-k8s-base

```
