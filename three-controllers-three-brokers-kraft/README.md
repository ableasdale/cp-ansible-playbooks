# Three KRaft nodes; three brokers; Confluent Control Center (C3)

## Prerequisites

To get the playbook running, you'll need seven EC2 instances:

- C3 (`t2.large`)
- Broker (`t2.large`)
- KRaft Controller (`t2.medium`)
  
## Modify the Playbook

Specify the PEM file that you used (to connect to the instance) when the instance was created in the playbook (`hosts.yaml`):

```yaml
    ansible_ssh_private_key_file: <yourPEMfilename>.pem
```

Configure the three KRaft Quorum Controllers (`kafka_controller`) - use the internal host DNS name for the first line and the Public DNS host name for the second line:

```yaml
kafka_controller:
  hosts:
    ip-xxx-xxx-xxx-xxx.aws-region.compute.internal:
      ansible_host: ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com
    ip-xxx-xxx-xxx-xxx.aws-region.compute.internal:
      ansible_host: ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com
    ip-xxx-xxx-xxx-xxx.aws-region.compute.internal:
      ansible_host: ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com
```

Configure the three Brokers (use the internal host DNS name for the first line and the Public DNS host name for the second line):

```yaml
kafka_broker:
  hosts:
    ip-xxx-xxx-xxx-xxx.aws-region.compute.internal:
      ansible_host: ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com
    ip-xxx-xxx-xxx-xxx.aws-region.compute.internal:
      ansible_host: ec2-xxx-xxx-xxx-xxx.aws-region.compute.amazonaws.com
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
PLAY RECAP ************************************************************************************************************************************
ip-10-0-0-149.eu-west-1.compute.internal : ok=66   changed=17   unreachable=0    failed=0    skipped=48   rescued=0    ignored=0
ip-10-0-12-71.eu-west-1.compute.internal : ok=66   changed=2    unreachable=0    failed=0    skipped=40   rescued=0    ignored=0
ip-10-0-13-195.eu-west-1.compute.internal : ok=66   changed=17   unreachable=0    failed=0    skipped=48   rescued=0    ignored=0
ip-10-0-14-129.eu-west-1.compute.internal : ok=66   changed=17   unreachable=0    failed=0    skipped=52   rescued=0    ignored=0
ip-10-0-2-251.eu-west-1.compute.internal : ok=65   changed=1    unreachable=0    failed=0    skipped=39   rescued=0    ignored=0
ip-10-0-3-222.eu-west-1.compute.internal : ok=56   changed=10   unreachable=0    failed=0    skipped=43   rescued=0    ignored=0
ip-10-0-4-219.eu-west-1.compute.internal : ok=65   changed=1    unreachable=0    failed=0    skipped=39   rescued=0    ignored=0
```

## Confirm that the cluster is using KRaft mode

If we connect to one of the Kafka brokers, we can grep for messages about quorum by running:

```bash
grep Raft server.log
```

You should see messages logged by `RaftManager` and `KafkaRaftServer`:

```log
[2023-06-15 09:43:38,532] INFO [RaftManager nodeId=1] Completed transition to Unattached(epoch=0, voters=[9991, 9992, 9993], electionTimeoutMs=1415) (org.apache.kafka.raft.QuorumState)
[2023-06-15 09:43:38,535] INFO [kafka-raft-outbound-request-thread]: Starting (kafka.raft.RaftSendThread)
[2023-06-15 09:43:38,544] INFO [kafka-raft-io-thread]: Starting (kafka.raft.KafkaRaftManager$RaftIoThread)
[2023-06-15 09:43:38,734] INFO [RaftManager nodeId=1] Completed transition to FollowerState(fetchTimeoutMs=2000, epoch=86, leaderId=9992, voters=[9991, 9992, 9993], highWatermark=Optional.empty, fetchingSnapshot=Optional.empty) (org.apache.kafka.raft.QuorumState)
[2023-06-15 09:43:38,831] INFO [RaftManager nodeId=1] High watermark set to Optional[LogOffsetMetadata(offset=551, metadata=Optional.empty)] for the first time for epoch 86 (org.apache.kafka.raft.FollowerState)
[2023-06-15 09:43:40,595] INFO [RaftManager nodeId=1] Registered the listener kafka.server.metadata.BrokerMetadataListener@1031726463 (org.apache.kafka.raft.KafkaRaftClient)
[2023-06-15 09:43:46,471] INFO [KafkaRaftServer nodeId=1] Kafka Server started (kafka.server.KafkaRaftServer)
```

And in the configuration reported in `server.log`:

```log
    controller.quorum.voters = [9991@ip-10-0-12-71.eu-west-1.compute.internal:9093, 9992@ip-10-0-4-219.eu-west-1.compute.internal:9093, 9993@ip-10-0-2-251.eu-west-1.compute.internal:9093]
```

If we connect to one of the controllers, look in `/var/log/controller` for `controller.log` for evidence that the three brokers have been registered with the Quorum Controllers:

```log
[2023-06-15 09:43:41,092] INFO [Controller 9991] Registered new broker: RegisterBrokerRecord(brokerId=1, isMigratingZkBroker=false, incarnation
Id=PU2-jSusTtiCnPMbuNkGsw, brokerEpoch=555, endPoints=[BrokerEndpoint(name='INTERNAL', host='ip-10-0-14-129.eu-west-1.compute.internal', port=9
092, securityProtocol=0), BrokerEndpoint(name='BROKER', host='ip-10-0-14-129.eu-west-1.compute.internal', port=9091, securityProtocol=0)], feat
ures=[BrokerFeature(name='confluent.metadata.version', minSupportedVersion=1, maxSupportedVersion=108), BrokerFeature(name='metadata.version',
minSupportedVersion=1, maxSupportedVersion=8)], rack=null, fenced=true, inControlledShutdown=false) (org.apache.kafka.controller.ClusterControl
Manager)
[2023-06-15 09:43:41,092] INFO [Controller 9991] Registered new broker: RegisterBrokerRecord(brokerId=3, isMigratingZkBroker=false, incarnation
Id=TBTJydJWRXKRCNXtGH-eew, brokerEpoch=556, endPoints=[BrokerEndpoint(name='INTERNAL', host='ip-10-0-13-195.eu-west-1.compute.internal', port=9
092, securityProtocol=0), BrokerEndpoint(name='BROKER', host='ip-10-0-13-195.eu-west-1.compute.internal', port=9091, securityProtocol=0)], feat
ures=[BrokerFeature(name='confluent.metadata.version', minSupportedVersion=1, maxSupportedVersion=108), BrokerFeature(name='metadata.version',
minSupportedVersion=1, maxSupportedVersion=8)], rack=null, fenced=true, inControlledShutdown=false) (org.apache.kafka.controller.ClusterControl
Manager)
[2023-06-15 09:43:41,591] INFO [Controller 9991] Registered new broker: RegisterBrokerRecord(brokerId=2, isMigratingZkBroker=false, incarnation
Id=wZeHi16sSEagVvZpvd9JUA, brokerEpoch=560, endPoints=[BrokerEndpoint(name='INTERNAL', host='ip-10-0-0-149.eu-west-1.compute.internal', port=90
92, securityProtocol=0), BrokerEndpoint(name='BROKER', host='ip-10-0-0-149.eu-west-1.compute.internal', port=9091, securityProtocol=0)], featur
es=[BrokerFeature(name='confluent.metadata.version', minSupportedVersion=1, maxSupportedVersion=108), BrokerFeature(name='metadata.version', mi
nSupportedVersion=1, maxSupportedVersion=8)], rack=null, fenced=true, inControlledShutdown=false) (org.apache.kafka.controller.ClusterControlMa
nager)
```

We can visit Confluent Control Center on port `9021`:

<http://ec2-3-249-62-113.eu-west-1.compute.amazonaws.com:9021/>

And we should be able to see evidence that the cluster is running in KRaft mode:

![KRaft Mode Example 1](../img/kraft-c3.png)
![KRaft Mode Example 2](../img/kraft-c3-2.png)
