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

![walfisch_img01](https://github.com/tech-sketch/walfisch/wiki/images/walfisch_img01.png)

* SSH key authentication required
* select one virtual machine as a manager node

## Usage

### Prepare Servers

1) Edit ansible configuration file (**prepare_servers/ansible.cfg**)
* Change **private_key_file** to your private key filepath

```text:prepare_servers/ansible.cfg
[defaults]
#remote_user = vagrant
remote_user = root
#private_key_file = ~/.ssh/id_rsa
private_key_file = /home/vagrant/.ssh/softlayer_rsa # this line
```
      
2) Edit public IP addresses of ansible hosts file (**prepare_servers/servers**)
* [all] : enumerate public IP addresses of all virtual machines
* [manager] : public IP address of the manager node

```text:prepare_servers/servers
[all]
XXX.XXX.XXX.XX1
XXX.XXX.XXX.XX2
YYY.YYY.YYY.YY1

[manager]
XXX.XXX.XXX.XX1
```

3) run ansible playbook

1. `cd prepare_servers`
2. `ansible-playbook -i servers site.yml`

When you perform the above tasks, ansible prepares the servers like below:

![walfisch_img02](https://github.com/tech-sketch/walfisch/wiki/images/walfisch_img02.png)

### Create containers and overlay networks

1) Edit ansible configuration file (**create_containers/ansible.cfg**)
* Change **private_key_file** to your private key filepath

```text:create_containers/ansible.cfg
[defaults]
#remote_user = vagrant
remote_user = root
#private_key_file = ~/.ssh/id_rsa
private_key_file = /home/vagrant/.ssh/softlayer_rsa # this line
```

2) Edit public IP addresses of ansible hosts file (**create_containers/servers**)
  * [all] : enumerate public IP addresses of all virtual machines
  * [manager] : public IP address of the manager node
  * [vnet1],[vnet2],... : public IP address for each node

```text:create_containers/servers
[all]
XXX.XXX.XXX.XX1
XXX.XXX.XXX.XX2
YYY.YYY.YYY.YY1

[manager]
XXX.XXX.XXX.XX1

[vnet1]
XXX.XXX.XXX.XX1

[vnet2]
XXX.XXX.XXX.XX2

[vnet3]
YYY.YYY.YYY.YY1
```

3) Edit "Virtual Router" private IP address of variables file (**create_containers/vars/networks.yml**)
  * you have to define private IP address of Virtual Router for each physical network (in this case, you have two physical networks on DC-A and DC-B)

```yml:create_containers/vars/networks.yml
networks:
  physical:
    - name:    "physical01"
      vrouter:
        ip_addr: "10.PPP.PPP.PP0" # this line
        mac:     "10:00:00:00:01:01"
    - name:    "physical02"
      vrouter:
        ip_addr: "10.QQQ.QQQ.QQ0" # this line
        mac:     "10:00:00:00:01:02"
  virtual:
    - name:    "virtual"
      nw_addr: "192.168.99.0"
      nw_mask: 24
      vrouter:
        ip_addr: "192.168.99.1"
        mac:     "10:00:00:00:01:03"
...
```

4) run ansible playbook

1. `cd create_containers`
2. `ansible-playbook -i servers site_start.yml`

When you perform the above tasks, ansible prepares the servers like below:

![walfisch_img03](https://github.com/tech-sketch/walfisch/wiki/images/walfisch_img03.png)

Logically, network configuration is like below:

![walfisch_img04](https://github.com/tech-sketch/walfisch/wiki/images/walfisch_img04.png)

### Terminate containers and overlay networks

1) run ansible playbook

1. `cd create_containers`
2. `ansible-playbook -i servers site_stop.yml`

## Notice

For constraints of OpenVNet current implementation, a host can't communicate with other host's containers directly.  
(e.g. 10.PPP.PPP.PP1 can communicate with 192.168.99.11, but can't communicate with 192.168.99.21)

Of course, all containers can communicate with each other.

## License

Copyright 2015 Nobuyuki Matsui <nobuyuki.matsui@gmail.com>

Walfisch is released under the Apache License version2.0. The Apache License version2.0 official full text is published at this [link](http://www.apache.org/licenses/LICENSE-2.0.html).

