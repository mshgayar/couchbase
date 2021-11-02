Couchbase Ansible Playbook
===========================

This project contains sample playbooks to manage Couchbase cluster.


Note: the current installation has been implemented based on 
* Centos 7 
* Couchbase version   couchbase-server-enterprise-4.6.4-centos7.x86_64.rpm

Installation Prerequesites
--------------------------
* Three Virtul Machines installed with Centos 7 
* From your Linux Control Machine(Your PC) , create ssh key and copy the public key to the three nodes
* confrim access to all nodes
* configure hostnames through your own DNS or /etc/hosts
 * 192.168.35.157   cb-cluster.lab.example.com cb-cluster
 * 192.168.35.142  cb-node01.lab.example.com cb-node01
 * 192.168.35.139  cb-node02.lab.example.com cb-node02   

Install Couchbase Cluster
-----------------------------------------------------------------------------------------------------------------------------------------------------------------
Part 1 ) Configure the list of Hosts in ansible "inventory" file as below :-
[couchbase-main]
cb-cluster.lab.example.com

[couchbase-nodes]
cb-node01.lab.example.com 
cb-node02.lab.example.com 

[cb:children]
couchbase-main
couchbase-nodes

[cb:vars]
ansible_ssh_user="ssh user to access all the nodes"
ansible_become=true
ansible_ssh_private_key_file=Path to the private ssh key to access all the nodes
ansible_ssh_common_args='-o StrictHostKeyChecking=no' 

This file contains 2 groups of server and ssh access variables:
[couchbase-main] 
select one of your server, this one will be used to initialize and configure the cluster.
                 cb-cluster.lab.example.com 
[couchbase-nodes] 
all other nodes that will be added to the cluster.
                 cb-node01.lab.example.com
                 cb-node02.lab.example.com
[cb:vars]
* ssh username to access all the nodes & install packages
* Path of ssh private key required to access nodes
* Option to avoid displaying ssh access confirmation


Part 2 ) Deployment
Enter the Administrator information and server configuration (RAM Quotas, Bucket name, Replicas) in the ./group_vars/all file or assign them directly from the role sections.

Run the Ansible command
ansible-playbook -i inventory  couchbase.yml -t "deploy,addnode"


This script will do the following:

1) ansible role "deploy"
* Install the required packages to prepare the couchbase server and nodes
* Configure perfomance tuning on all cluster nodes
  a) nofile limits /etc/security/limits.conf
  b) vm.swappiness
  c) disable Transparent Huge Pages
* Download the Couchbase package if not already present on all nodes
* Install Couchbase on all nodes
* Create a new cluster on the "main" node : cb-cluster.lab.example.com

2) ansible role "addnode"
* Add all other nodes to the cluster
* listing the current cluster nodes
* display couchbase cluster nodes
* configure cluster name
* displaying cluster nodes information
* Rebalance the cluster
* Create a new bucket


Uninstall Couchbase Cluster
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
3) Ansible role "remove" :

Uninstall the Couchbase server and delete all files (Warning you will lose your data)

Run the Ansible command :
ansible-playbook -i inventory couchbase.yml -t remove

