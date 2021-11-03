
# Couchbase Ansible Playbook
This project contains sample playbooks to manage Couchbase cluster.

## 🔗 Links
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com//in/mosalah90/)

The current installation has been implemented based on
  * Centos 7
  * Couchbase version   couchbase-server-enterprise-4.6.4-centos7.x86_64.rpm

Installation Pre-Requisties
-
- Three Virtual Machines installed with Centos 7 
- From your Linux Control Machine(Your PC)
- Create ssh key and copy the public key to the three nodes
- Confirm access to all nodes
- Configure hostnames through your own DNS or /etc/hosts :
```
  * 192.168.35.157  cb-cluster.lab.example.com cb-cluster
  * 192.168.35.142  cb-node01.lab.example.com cb-node01
  * 192.168.35.139  cb-node02.lab.example.com cb-node02   
```





## Couchbase Cluster Deployment

   *   Step 1 : Configure Ansible inventory file "inventory"

```bash
#This file contains 2 groups of couchbase server & couchbase nodes and ssh access variables
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
*    Step 3 : Access the cluster
        - from your browser : https://MainNodeHostname/IP:8091
        - login with cluster Username and Password


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

## Run Locally

Clone the project

```bash
  git clone https://github.com/mshgayar/couchbase.git
```

Go to the project directory

```bash
  cd couchbase
```

Update Invenotry file with the cluster hostnames
```
# vim inventory
 [couchbase-main]
 cb-cluster.lab.example.com

 [couchbase-nodes]
 cb-node01.lab.example.com
 cb-node02.lab.example.com

 [cb:children]
 couchbase-main
 couchbase-nodes

 [cb:vars]
 ansible_ssh_user=centos
 ansible_become=true
 ansible_ssh_private_key_file=~/.ssh/cloud-key
 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

Update the global vars :
```
# vim group_vars/all
```

install the cluster with ansible playbook

```bash
  ansible-playbook -i inventory couchbase.yml -t "deploy,addnode"
```
