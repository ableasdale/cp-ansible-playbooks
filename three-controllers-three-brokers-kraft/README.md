

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

```
[2023-06-15 09:43:38,532] INFO [RaftManager nodeId=1] Completed transition to Unattached(epoch=0, voters=[9991, 9992, 9993], electionTimeoutMs=1415) (org.apache.kafka.raft.QuorumState)
[2023-06-15 09:43:38,734] INFO [RaftManager nodeId=1] Completed transition to FollowerState(fetchTimeoutMs=2000, epoch=86, leaderId=9992, voters=[9991, 9992, 9993], highWatermark=Optional.empty, fetchingSnapshot=Optional.empty) (org.apache.kafka.raft.QuorumState)
```

And in the configuration reported in `server.log`:

```
        controller.quorum.voters = [9991@ip-10-0-12-71.eu-west-1.compute.internal:9093, 9992@ip-10-0-4-219.eu-west-1.compute.internal:9093, 9993@ip-10-0-2-251.eu-west-1.compute.internal:9093]
```


We can visit Confluent Control Center on port `9021`:

<http://ec2-3-249-62-113.eu-west-1.compute.amazonaws.com:9021/>

And we should be able to see evidence that the cluster is running in KRaft mode:

![KRaft Mode Example 1](../img/kraft-c3.png)
![KRaft Mode Example 2](../img/kraft-c3-2.png)
