Walfisch -- Docker networking tool using OpenVNet
====

This [Ansible](http://www.ansible.com) playbooks create the [Docker](https://www.docker.com/) Overlay Network using [OpenVNet](http://openvnet.org/) on [SoftLayer](http://www.softlayer.com/).

## Description

* **prepare_servers** playbook prepares virtual machines on SoftLayer as follows:
  * configure OS(sshd, iptables, etc)
  * install iproute2 from OpenStack RDO
  * install Docker
  * install Open vSwitch
  * install OpenVNet from axsh
  * configure NIC as OVS bridge
  * configure OpenVNet

* **create_containers** playbook creates Docker containers and overlay networks on these virtual machines
  * create Docker containers on each virtual machine
  * create overlay network using OpenVNet
  * set static route to containers and virtual machines

## Requirement

### Andible Client

* Ansible 1.8.2 or later

### Virtual Machines

You have to create virtual machines(**CentOS 6.6**) on SoftLayer like below:

* SSH key authentication required
* select one virtual machine as a manager node

## Usage

### Prepare Servers

1. Edit ansible configuration of `prepare_servers/ansible.cfg`
  * Change `private_key_file` to your private key filepath
1. Edit Public IP Addresses of `prepare_servers/servers`
  * [all] : enumerate Public IP Addresses of all virtual machines
  * [manager] : Public IP Address of the manager node
2. run ansible playbook
  * `cd prepare_servers`
  * `ansible-playbook -i servers site.yml`

### Create containers and overlay networks




