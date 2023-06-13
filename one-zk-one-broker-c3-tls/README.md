# One Zookeeper node; one broker; Confluent Control Center (C3) and TLS (SSL)

## Introduction

Note that TLS/SSL can be configured by adding a single line to your playbook:

```yaml
  vars:
    ssl_enabled: true
```

Note that this is a catch-all and will enable TLS for all components. [https://docs.confluent.io/ansible/current/ansible-encrypt.html#configure-tls-for-individual-components](See the documentation for configuring TLS for selected components).

With SSL enabled as per the example above, note that the Ansible host system will generate a CA certificate for you:

```bash
changed: [ip-10-0-3-61.eu-west-1.compute.internal] => (item=snakeoil-ca-1.key)
changed: [ip-10-0-3-61.eu-west-1.compute.internal] => (item=snakeoil-ca-1.crt)
```

After this is done, you'll be able to access the files in the following directory:

```bash
cd /home/ubuntu/.ansible/collections/ansible_collections/confluent/platform/playbooks/generated_ssl_files/
ls -l
```

You should see:

```bash
-rw-rw-r-- 1 ubuntu ubuntu 1338 Jun 13 18:46 snakeoil-ca-1.crt
-rw-rw-r-- 1 ubuntu ubuntu 1854 Jun 13 18:46 snakeoil-ca-1.key
```

## Security Model Changes

Note that this configuration introduces a new TCP port which will need to be added to your Security Group:

![Zookeeper TLS Listener](/img/tls-zk-listener.png)

## Prerequisites

To get the playbook running, you'll need three EC2 instances:

- C3 (`t2.large`)
- Broker (`t2.large`)
- Zookeeper (`t2.medium`)
- Schema Registry (`t2.medium`)

## Modify the Playbook

Specify the PEM file that you used (to connect to the instance) when the instance was created in the playbook (`hosts.yaml`):

```yaml
    ansible_ssh_private_key_file: <yourPEMfilename>.pem
```

Configure the Zookeeper host (use the internal host DNS name for the first line and the Public DNS host name for the second line):

```yaml
zookeeper:
  hosts:
    ip-xxx-xxx-xxx-xxx.aws-region.compute.internal:
      ansible_host: ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com
```

Configure the Broker (use the internal host DNS name for the first line and the Public DNS host name for the second line):

```yaml
kafka_broker:
  hosts:
    ip-xxx-xxx-xxx-xxx.aws-region.compute.internal:
      ansible_host: ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com
```

Configure the Schema Registry host (use the internal host DNS name for the first line and the Public DNS host name for the second line):

```yaml
schema_registry:
  hosts:
    ip-xxx-xxx-xxx-xxx.aws-region.compute.internal:
      ansible_host: ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com
```

Configure the C3 host (use the internal host DNS name for the first line and the Public DNS host name for the second line):

```yaml
control_center:
  hosts:
    ip-xxx-xxx-xxx-xxx.aws-region.compute.internal:
      ansible_host: ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com
```

### Run Ansible

Run the playbook:

```bash
ansible-playbook -i hosts.yaml confluent.platform.all
```

When the playbook has finished running, you'll see something like this:

```bash
PLAY RECAP ***************************************************************************************************************************************************
ip-10-0-14-73.eu-west-1.compute.internal : ok=60   changed=1    unreachable=0    failed=0    skipped=51   rescued=0    ignored=0
ip-10-0-3-61.eu-west-1.compute.internal : ok=55   changed=22   unreachable=0    failed=0    skipped=41   rescued=0    ignored=0
ip-10-0-8-90.eu-west-1.compute.internal : ok=56   changed=10   unreachable=0    failed=0    skipped=42   rescued=0    ignored=0
```

### Testing the output

Let's start with Zookeeper:

```bash
zookeeper-shell localhost:2181
ls /
ls /brokers
ls /brokers/ids
```

This indicates that Zookeeper is running and a broker has been registered.

Let's check the broker:

```bash
kafka-topics --bootstrap-server localhost:9091 --list
kafka-topics --bootstrap-server localhost:9091 --under-replicated-partitions --describe
```

Create a topic:

```bash
kafka-topics --bootstrap-server localhost:9091 --create --topic test-topic --replication-factor 1 --partitions 1
```

Produce to the topic:

```bash
kafka-console-producer --bootstrap-server localhost:9091 --topic test-topic
```

Consume from the topic:

```bash
kafka-console-consumer --bootstrap-server localhost:9091 --topic test-topic --from-beginning
```

Now let's try to connect to C3:

<http://ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com:9021>
