# One Zookeeper node; one broker; Confluent Control Center (C3)

To get the playbook running, you'll need three EC2 instances:

- Broker (`t2.large`)
- Zookeeper (`t2.medium`)
- C3 (`t2.large`)

### Modify the Playbook

Specify the PEM file that you used when the instance was created in the playbook (`hosts.yaml`): 

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
PLAY RECAP *********************************************************************
ip-10-0-15-151.eu-west-1.compute.internal : ok=63   changed=27   unreachable=0    failed=0    skipped=51   rescued=0    ignored=0
ip-10-0-7-82.eu-west-1.compute.internal : ok=55   changed=22   unreachable=0    failed=0    skipped=41   rescued=0    ignored=0
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