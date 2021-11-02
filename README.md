
# Couchbase Ansible Playbook
This project contains sample playbooks to manage Couchbase cluster.


The current installation has been implemented based on 
  * Centos 7 
  * Couchbase version   couchbase-server-enterprise-4.6.4-centos7.x86_64.rpm

Installation Prerequesites
-
- Three Virtul Machines installed with Centos 7 
- From your Linux Control Machine(Your PC) , create ssh key and copy the public key to the three nodes
- confrim access to all nodes
- configure hostnames through your own DNS or /etc/hosts :
```
  * 192.168.35.157  cb-cluster.lab.example.com cb-cluster
  * 192.168.35.142  cb-node01.lab.example.com cb-node01
  * 192.168.35.139  cb-node02.lab.example.com cb-node02   
```


 


## Couchbase Cluster Deployment

   *   Step 1 : Configure Ansible inventory file "inventory"

```bash
# vim inventory

#This file contains 2 groups of couchbase server & couchbase nodes
and ssh access variables

#select one of your server, this one will be used to initialize and configure the cluster
  [couchbase-main]
    cb-cluster.lab.example.com

#all other nodes that will be added to the cluster.
  [couchbase-nodes]
    cb-node01.lab.example.com
    cb-node02.lab.example.com

  [cb:children] 
    couchbase-main
    couchbase-nodes

# Description of vars section
# - ssh username to access all the nodes & install packages
# - Path of ssh private key required to access nodes
# - Option to avoid displaying ssh access confirmation

  [cb:vars]
    ansible_ssh_user="Your Own SSH User"
    ansible_become=true
    ansible_ssh_private_key_file="Path to Your Own SSH Private Key"
    ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

*    Step 2 : Installing Cluster with Ansible Playbook roles
      
          - Enter the Administrator information and server configuration (RAM Quotas, Bucket name, Replicas) 
          in the ./group_vars/all file or assign them directly from the role sections.
          - ansible-playbook -i inventory  couchbase.yml -t "deploy,addnode"



Ansible Role "deploy" Tasks :-
  -
     * Install the required packages to prepare the couchbase server and nodes
     * Configure perfomance tuning on all cluster nodes :-
         1) nofile limits /etc/security/limits.conf
         2) vm.swappiness with sysctl
         3) disable Transparent Huge Pages :
           echo never > /sys/kernel/mm/transparent_hugepage/enabled
           echo never > /sys/kernel/mm/transparent_hugepage/defrag
      * Download the Couchbase package if not already present on all nodes
      * Install Couchbase on all nodes
      * Create a new cluster on the "main" node :
          -   cb-cluster.lab.example.com

Ansible Role "addnode" Tasks :_ 
   -
       * Adding all other nodes to the cluster
       * listing the current cluster nodes
       * displaying couchbase cluster nodes
       * configuring couchbase cluster name
       * displaying cluster nodes information
       * Rebalancing the cluster after the added nodes registration
       * Creating a new bucket 


Couchbase Cluster Uninstall
-
     -  Uninstall the Couchbase server and delete all files (Warning you will lose your data)
     -  ansible-playbook -i inventory couchbase.yml -t remove
