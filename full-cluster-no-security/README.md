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

![Open these in your CIDR range](/img/security-group-settings.png)

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

## Basic Tests

List topics

```bash
ssh -i <yourPEMFile>.pem ubuntu@ec2-xxx-yyy
kafka-topics --bootstrap-server localhost:9091 --list
```

## Debugging

```bash
ansible-playbook -i hosts.yaml confluent.platform.all -vvvv
```

Is `confluent-server` installed?  What is it's status?

```bash
apt list --installed confluent-server
sudo systemctl status confluent-server
```

## Restarting the components

To restart the broker:

```bash
sudo systemctl restart confluent-server
sudo systemctl status confluent-server
```

To restart Confluent Control Center (C3):

```bash
sudo systemctl restart confluent-control-center
sudo systemctl status confluent-control-center
```

To restart Zookeeper

```bash
sudo systemctl restart confluent-zookeeper
sudo systemctl status confluent-zookeeper
```

## Where are the logs?

For the broker:

```bash
sudo su
cd /var/log/kafka/
tail -n200 server.log
```

For Control Center (C3):

```bash
sudo su
cd /var/log/confluent/control-center
tail -f control-center.log
```

For Zookeeper:

```bash
sudo su
cd /var/log/kafka
tail -n200 zookeeper-server.log
```

## Where are the configuration files?

For the broker:

```bash
cd /etc/kafka
less server.properties
```

For Zookeeper:

```bash
cd /etc/kafka
less zookeeper.properties
```

## Where is all the data stored for the brokers?

By default, you'll find it in `/var/lib/kafka/data`:

```bash
cd /var/lib/kafka/data
```

## Looking for specific files

Install `mlocate` to find a specific file installed by cp-ansible:

```bash
sudo apt install mlocate
sudo su
locate server.properties
```

You should see:

```bash
/etc/kafka/server.properties
/etc/kafka/kraft/server.properties
```

If you have 4-letter words enabled for Zookeeper, you can access those stats by running:

```bash
echo mntr | nc localhost 2181
```

How you do this with a TLS connection?

```bash
echo mntr | openssl s_client -connect localhost:2182
```

(this returns data - but it's not very readable...)

This works:

```bash
echo mntr | openssl s_client -connect localhost:2182 -ign_eof
```

### Try with `ncat`

This also works:

```bash
sudo apt-get install ncat
echo mntr | ncat --ssl localhost 2182
```

### What about `zookeeper-shell` over TLS?

`zookeeper-shell` doesn't appear to work if Zookeeper is configured with TLS; if you run this, it will fail:

```bash
zookeeper-shell localhost:2182
```

To make this connection using TLS, you need to pass in an extra switch: `-zk-tls-config-file`.  You need to specify a properties file for this.

Here's an example of a file called `zkshell.properties` to illustrate what that would look like (note that this uses the default truststore information that `cp-ansible` uses):

```properties
zookeeper.ssl.client.enable=true
zookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty
zookeeper.ssl.truststore.location=/var/ssl/private/zookeeper.truststore.jks
zookeeper.ssl.truststore.password=confluenttruststorepass
```

With this file (`zkshell.properties`) in place, you can now run `zookeeper-shell` to connect using TLS by doing the following:

```bash
zookeeper-shell ip-10-0-10-36.eu-west-1.compute.internal:2182 -zk-tls-config-file /etc/kafka/zkshell.properties
```

Below is an example to get broker topic information for a given topic (`_confluent-command`) and to pipe it out to a text file (`zookeeper-shell.out`):

```bash
zookeeper-shell ip-10-0-10-36.eu-west-1.compute.internal:2182 -zk-tls-config-file /etc/kafka/zkshell.properties get /brokers/topics/_confluent-command 2>&- | grep "^{" >> zookeeper-shell.out
```

See: (Confluent Documentation: Connecting to TLS-enabled ZooKeeper using CLI tools)[https://docs.confluent.io/platform/current/security/zk-security.html#connecting-to-tls-enabled-zk-using-cli-tools]
