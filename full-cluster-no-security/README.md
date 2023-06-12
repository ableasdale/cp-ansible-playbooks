# Full Cluster - no security

Basic setup to build on later.

## Prerequisites

For the brokers we will use 3 `t2.large` Ubuntu 20.04 (focal) instances
For the other components we will use `t2.medium`.
The instances will all reside within a subnet in your VPC.

## Setup

TODO - configure VPC, Route Table, Subnet, Internet Gateway

## Configure VPC for Public DNS hostnames (if required)

In your VPC:

![Edit VPC Settings](/img/edit-vpc-settings.png)

![Public Hostnames](/img/public-hostnames.png)

## Routes

![Routes](/img/route-table.png)

## Security Group Rules

![Open these in your CIDR range](/img/security-group-inbound.png)

## Configure Ansible on your Ubuntu "ansible" machine

```bash
ssh -i <yourPEMfile.pem> ubuntu@ec2-xxx-yyy-....
```

## Installing cp-ansible and dependencies

```bash
sudo apt update
sudo apt install python3-pip
sudo -H pip3 install ansible
ansible-galaxy collection install community.general
ansible-galaxy collection install git+https://github.com/confluentinc/cp-ansible.git
```

## Modify the instances

## Ping test

Ensure all hosts can be reached by running:

```bash
ansible -i hosts.yaml all -m ping
```

Install the playbook:

```bash
ansible-playbook -i hosts.yaml confluent.platform.all
```

## Basic Tests

List topics

```bash
ssh -i <yourPEMFile>.pem ubuntu@ec2-xxx-yyy
kafka-topics --bootstrap-server localhost:9091 --list
```

## Debugging

```bash
ansible-playbook -i hosts.yaml confluent.platform.all -vvvv
```

Is `confluent-server` installed?  What is it's status?

```bash
apt list --installed confluent-server
sudo systemctl status confluent-server
```
