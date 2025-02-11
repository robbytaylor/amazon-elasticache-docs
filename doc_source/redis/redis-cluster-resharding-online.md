# Online resharding and shard rebalancing for Redis \(cluster mode enabled\)<a name="redis-cluster-resharding-online"></a>

By using online resharding and shard rebalancing with Amazon ElastiCache for Redis version 3\.2\.10 or newer, you can scale your ElastiCache for Redis \(cluster mode enabled\) dynamically with no downtime\. This approach means that your cluster can continue to serve requests even while scaling or rebalancing is in process\.

You can do the following:
+ **Scale out** – Increase read and write capacity by adding shards \(node groups\) to your Redis \(cluster mode enabled\) cluster \(replication group\)\.

  If you add one or more shards to your replication group, the number of nodes in each new shard is the same as the number of nodes in the smallest of the existing shards\.
+ **Scale in** – Reduce read and write capacity, and thereby costs, by removing shards from your Redis \(cluster mode enabled\) cluster\.
+ **Rebalance** – Move the keyspaces among the shards in your ElastiCache for Redis \(cluster mode enabled\) cluster so they are as equally distributed among the shards as possible\.

You can't do the following:
+ **Configure shards independently:**

  You can't specify the keyspace for shards independently\. To do this, you must use the offline process\.

Currently, the following limitations apply to ElastiCache for Redis online resharding and rebalancing:
+ These processes require Redis engine version 3\.2\.10 or newer\. For information on upgrading your engine version, see [Upgrading engine versions](VersionManagement.md)\.
+ There are limitations with slots or keyspaces and large items:

  If any of the keys in a shard contain a large item, that key isn't migrated to a new shard when scaling out or rebalancing\. This functionality can result in unbalanced shards\.

  If any of the keys in a shard contain a large item \(items greater than 256 MB after serialization\), that shard isn't deleted when scaling in\. This functionality can result in some shards not being deleted\.
+ When scaling out, the number of nodes in any new shards equals the number of nodes in the smallest existing shard\.
+ When scaling out, any tags that are common to all existing shards are copied to the new shards\.

For more information, see [Best practices: Online cluster resizing](best-practices-online-resharding.md)\.

You can horizontally scale or rebalance your ElastiCache for Redis \(cluster mode enabled\) clusters using the AWS Management Console, the AWS CLI, and the ElastiCache API\.

## Adding shards with online resharding<a name="redis-cluster-resharding-online-add"></a>

You can add shards to your Redis \(cluster mode enabled\) cluster using the AWS Management Console, AWS CLI, or ElastiCache API\. When you add shards to a Redis \(cluster mode enabled\) cluster, any tags on the existing shards are copied over to the new shards\.

**Topics**

### Adding shards \(Console\)<a name="redis-cluster-resharding-online-add-console"></a>

You can use the AWS Management Console to add one or more shards to your Redis \(cluster mode enabled\) cluster\. The following procedure describes the process\.

**To add shards to your Redis \(cluster mode enabled\) cluster**

1. Open the ElastiCache console at [https://console\.aws\.amazon\.com/elasticache/](https://console.aws.amazon.com/elasticache/)\.

1. From the navigation pane, choose **Redis**\.

1. Locate and choose the name, not the box to the left of the cluster's name, of the Redis \(cluster mode enabled\) cluster that you want to add shards to\.
**Tip**  
Redis \(cluster mode enabled\) show **Clustered Redis** in the **Mode** column

1. Choose **Add shard**\.

   1. For **Number of shards to be added**, choose the number of shards you want added to this cluster\.

   1. For **Availability zone\(s\)**, choose either **No preference** or **Specify availability zones**\.

   1. If you chose **Specify availability zones**, for each node in each shard, select the node's Availability Zone from the list of Availability Zones\.

   1. Choose **Add**\.

### Adding shards \(AWS CLI\)<a name="redis-cluster-resharding-online-add-cli"></a>

The following process describes how to reconfigure the shards in your Redis \(cluster mode enabled\) cluster by adding shards using the AWS CLI\.

Use the following parameters with `modify-replication-group-shard-configuration`\.

**Parameters**
+ `--apply-immediately` – Required\. Specifies the shard reconfiguration operation is to be started immediately\.
+ `--replication-group-id` – Required\. Specifies which replication group \(cluster\) the shard reconfiguration operation is to be performed on\.
+ `--node-group-count` – Required\. Specifies the number of shards \(node groups\) to exist when the operation is completed\. When adding shards, the value of `--node-group-count` must be greater than the current number of shards\.

  Optionally, you can specify the Availability Zone for each node in the replication group using `--resharding-configuration`\.
+ `--resharding-configuration` – Optional\. A list of preferred Availability Zones for each node in each shard in the replication group\. Use this parameter only if the value of `--node-group-count` is greater than the current number of shards\. If this parameter is omitted when adding shards, Amazon ElastiCache selects the Availability Zones for the new nodes\.

The following example reconfigures the keyspaces over four shards in the Redis \(cluster mode enabled\) cluster `my-cluster`\. The example also specifies the Availability Zone for each node in each shard\. The operation begins immediately\.

**Example \- Adding Shards**  
For Linux, macOS, or Unix:  

```
aws elasticache modify-replication-group-shard-configuration \
    --replication-group-id my-cluster \
    --node-group-count 4 \
    --resharding-configuration \
        "PreferredAvailabilityZones=us-east-2a,us-east-2c" \
        "PreferredAvailabilityZones=us-east-2b,us-east-2a" \
        "PreferredAvailabilityZones=us-east-2c,us-east-2d" \
        "PreferredAvailabilityZones=us-east-2d,us-east-2c" \
    --apply-immediately
```
For Windows:  

```
aws elasticache modify-replication-group-shard-configuration ^
    --replication-group-id my-cluster ^
    --node-group-count 4 ^
    --resharding-configuration ^
        "PreferredAvailabilityZones=us-east-2a,us-east-2c" ^
        "PreferredAvailabilityZones=us-east-2b,us-east-2a" ^
        "PreferredAvailabilityZones=us-east-2c,us-east-2d" ^
        "PreferredAvailabilityZones=us-east-2d,us-east-2c" ^
    --apply-immediately
```

For more information, see [modify\-replication\-group\-shard\-configuration](https://docs.aws.amazon.com/cli/latest/reference/elasticache/modify-replication-group-shard-configuration.html) in the AWS CLI documentation\.

### Adding shards \(ElastiCache API\)<a name="redis-cluster-resharding-online-add-api"></a>

You can use the ElastiCache API to reconfigure the shards in your Redis \(cluster mode enabled\) cluster online by using the `ModifyReplicationGroupShardConfiguration` operation\.

Use the following parameters with `ModifyReplicationGroupShardConfiguration`\.

**Parameters**
+ `ApplyImmediately=true` – Required\. Specifies the shard reconfiguration operation is to be started immediately\.
+ `ReplicationGroupId` – Required\. Specifies which replication group \(cluster\) the shard reconfiguration operation is to be performed on\.
+ `NodeGroupCount` – Required\. Specifies the number of shards \(node groups\) to exist when the operation is completed\. When adding shards, the value of `NodeGroupCount` must be greater than the current number of shards\.

  Optionally, you can specify the Availability Zone for each node in the replication group using `ReshardingConfiguration`\.
+ `ReshardingConfiguration` – Optional\. A list of preferred Availability Zones for each node in each shard in the replication group\. Use this parameter only if the value of `NodeGroupCount` is greater than the current number of shards\. If this parameter is omitted when adding shards, Amazon ElastiCache selects the Availability Zones for the new nodes\.

The following process describes how to reconfigure the shards in your Redis \(cluster mode enabled\) cluster by adding shards using the ElastiCache API\.

**Example \- Adding Shards**  
The following example adds node groups to the Redis \(cluster mode enabled\) cluster `my-cluster`, so there are a total of four node groups when the operation completes\. The example also specifies the Availability Zone for each node in each shard\. The operation begins immediately\.  

```
https://elasticache.us-east-2.amazonaws.com/
    ?Action=ModifyReplicationGroupShardConfiguration
    &ApplyImmediately=true
    &NodeGroupCount=4
    &ReplicationGroupId=my-cluster
    &ReshardingConfiguration.ReshardingConfiguration.1.PreferredAvailabilityZones.AvailabilityZone.1=us-east-2a 
    &ReshardingConfiguration.ReshardingConfiguration.1.PreferredAvailabilityZones.AvailabilityZone.2=us-east-2c 
    &ReshardingConfiguration.ReshardingConfiguration.2.PreferredAvailabilityZones.AvailabilityZone.1=us-east-2b 
    &ReshardingConfiguration.ReshardingConfiguration.2.PreferredAvailabilityZones.AvailabilityZone.2=us-east-2a 
    &ReshardingConfiguration.ReshardingConfiguration.3.PreferredAvailabilityZones.AvailabilityZone.1=us-east-2c 
    &ReshardingConfiguration.ReshardingConfiguration.3.PreferredAvailabilityZones.AvailabilityZone.2=us-east-2d 
    &ReshardingConfiguration.ReshardingConfiguration.4.PreferredAvailabilityZones.AvailabilityZone.1=us-east-2d 
    &ReshardingConfiguration.ReshardingConfiguration.4.PreferredAvailabilityZones.AvailabilityZone.2=us-east-2c 
    &Version=2015-02-02
    &SignatureVersion=4
    &SignatureMethod=HmacSHA256
    &Timestamp=20171002T192317Z
    &X-Amz-Credential=<credential>
```

For more information, see [ModifyReplicationGroupShardConfiguration](https://docs.aws.amazon.com/AmazonElastiCache/latest/APIReference/API_ModifyReplicationGroupShardConfiguration.html) in the ElastiCache API Reference\.

## Removing shards with online resharding<a name="redis-cluster-resharding-online-remove"></a>

You can remove shards from your Redis \(cluster mode enabled\) cluster using the AWS Management Console, AWS CLI, or ElastiCache API\.

**Topics**
+ [Removing shards \(Console\)](#redis-cluster-resharding-online-remove-console)
+ [Removing shards \(AWS CLI\)](#redis-cluster-resharding-online-remove-cli)
+ [Removing shards \(ElastiCache API\)](#redis-cluster-resharding-online-remove-api)

### Removing shards \(Console\)<a name="redis-cluster-resharding-online-remove-console"></a>

The following process describes how to reconfigure the shards in your Redis \(cluster mode enabled\) cluster by removing shards using the AWS Management Console\.

Before removing node groups \(shards\) from your replication group, ElastiCache makes sure that all your data will fit in the remaining shards\. If the data will fit, the specified shards are deleted from the replication group as requested\. If the data won't fit in the remaining node groups, the process is terminated and the replication group is left with the same node group configuration as before the request was made\.

You can use the AWS Management Console to remove one or more shards from your Redis \(cluster mode enabled\) cluster\. You cannot remove all the shards in a replication group\. Instead, you must delete the replication group\. For more information, see [Deleting a replication group](Replication.DeletingRepGroup.md)\. The following procedure describes the process for deleting one or more shards\.

**To remove shards from your Redis \(cluster mode enabled\) cluster**

1. Open the ElastiCache console at [https://console\.aws\.amazon\.com/elasticache/](https://console.aws.amazon.com/elasticache/)\.

1. From the navigation pane, choose **Redis**\.

1. Locate and choose the name, not the box to the left of the cluster's name, of the Redis \(cluster mode enabled\) cluster you want to remove shards from\.
**Tip**  
Redis \(cluster mode enabled\) clusters have a value of 1 or greater in the **Shards** column\.

1. From the list of shards, choose the box to the left of the name of each shard that you want to delete\.

1. Choose **Delete shard**\.

### Removing shards \(AWS CLI\)<a name="redis-cluster-resharding-online-remove-cli"></a>

The following process describes how to reconfigure the shards in your Redis \(cluster mode enabled\) cluster by removing shards using the AWS CLI\.

**Important**  
Before removing node groups \(shards\) from your replication group, ElastiCache makes sure that all your data will fit in the remaining shards\. If the data will fit, the specified shards \(`--node-groups-to-remove`\) are deleted from the replication group as requested and their keyspaces mapped into the remaining shards\. If the data will not fit in the remaining node groups, the process is terminated and the replication group is left with the same node group configuration as before the request was made\.

You can use the AWS CLI to remove one or more shards from your Redis \(cluster mode enabled\) cluster\. You cannot remove all the shards in a replication group\. Instead, you must delete the replication group\. For more information, see [Deleting a replication group](Replication.DeletingRepGroup.md)\.

Use the following parameters with `modify-replication-group-shard-configuration`\.

**Parameters**
+ `--apply-immediately` – Required\. Specifies the shard reconfiguration operation is to be started immediately\.
+ `--replication-group-id` – Required\. Specifies which replication group \(cluster\) the shard reconfiguration operation is to be performed on\.
+ `--node-group-count` – Required\. Specifies the number of shards \(node groups\) to exist when the operation is completed\. When removing shards, the value of `--node-group-count` must be less than the current number of shards\.

  
+ `--node-groups-to-remove` – Required when `--node-group-count` is less than the current number of node groups \(shards\)\. A list of shard \(node group\) IDs to remove from the replication group\.

The following procedure describes the process for deleting one or more shards\.

**Example \- Removing Shards**  
The following example removes two node groups from the Redis \(cluster mode enabled\) cluster `my-cluster`, so there are a total of two node groups when the operation completes\. The keyspaces from the removed shards are distributed evenly over the remaining shards\.  
For Linux, macOS, or Unix:  

```
aws elasticache modify-replication-group-shard-configuration \
    --replication-group-id my-cluster \
    --node-group-count 2 \
    --node-groups-to-remove "0002" "0003" \
    --apply-immediately
```
For Windows:  

```
aws elasticache modify-replication-group-shard-configuration ^
    --replication-group-id my-cluster ^
    --node-group-count 2 ^
    --node-groups-to-remove "0002" "0003" ^
    --apply-immediately
```

### Removing shards \(ElastiCache API\)<a name="redis-cluster-resharding-online-remove-api"></a>

You can use the ElastiCache API to reconfigure the shards in your Redis \(cluster mode enabled\) cluster online by using the `ModifyReplicationGroupShardConfiguration` operation\.

The following process describes how to reconfigure the shards in your Redis \(cluster mode enabled\) cluster by removing shards using the ElastiCache API\.

**Important**  
Before removing node groups \(shards\) from your replication group, ElastiCache makes sure that all your data will fit in the remaining shards\. If the data will fit, the specified shards \(`NodeGroupsToRemove`\) are deleted from the replication group as requested and their keyspaces mapped into the remaining shards\. If the data will not fit in the remaining node groups, the process is terminated and the replication group is left with the same node group configuration as before the request was made\.

You can use the ElastiCache API to remove one or more shards from your Redis \(cluster mode enabled\) cluster\. You cannot remove all the shards in a replication group\. Instead, you must delete the replication group\. For more information, see [Deleting a replication group](Replication.DeletingRepGroup.md)\.

Use the following parameters with `ModifyReplicationGroupShardConfiguration`\.

**Parameters**
+ `ApplyImmediately=true` – Required\. Specifies the shard reconfiguration operation is to be started immediately\.
+ `ReplicationGroupId` – Required\. Specifies which replication group \(cluster\) the shard reconfiguration operation is to be performed on\.
+ `NodeGroupCount` – Required\. Specifies the number of shards \(node groups\) to exist when the operation is completed\. When removing shards, the value of `NodeGroupCount` must be less than the current number of shards\.
+ `NodeGroupsToRemove` – Required when `--node-group-count` is less than the current number of node groups \(shards\)\. A list of shard \(node group\) IDs to remove from the replication group\.

The following procedure describes the process for deleting one or more shards\.

**Example \- Removing Shards**  
The following example removes two node groups from the Redis \(cluster mode enabled\) cluster `my-cluster`, so there are a total of two node groups when the operation completes\. The keyspaces from the removed shards are distributed evenly over the remaining shards\.  

```
https://elasticache.us-east-2.amazonaws.com/
    ?Action=ModifyReplicationGroupShardConfiguration
    &ApplyImmediately=true
    &NodeGroupCount=2
    &ReplicationGroupId=my-cluster
    &NodeGroupsToRemove.member.1=0002
    &NodeGroupsToRemove.member.2=0003
    &Version=2015-02-02
    &SignatureVersion=4
    &SignatureMethod=HmacSHA256
    &Timestamp=20171002T192317Z
    &X-Amz-Credential=<credential>
```

## Online shard rebalancing<a name="redis-cluster-resharding-online-rebalance"></a>

You can rebalance shards in your Redis \(cluster mode enabled\) cluster using the AWS Management Console, AWS CLI, or ElastiCache API\.

**Topics**
+ [Online Shard Rebalancing \(Console\)](#redis-cluster-resharding-online-rebalance-console)
+ [Online shard rebalancing \(AWS CLI\)](#redis-cluster-resharding-online-rebalance-cli)
+ [Online shard rebalancing \(ElastiCache API\)](#redis-cluster-resharding-online-rebalance-api)

### Online Shard Rebalancing \(Console\)<a name="redis-cluster-resharding-online-rebalance-console"></a>

The following process describes how to reconfigure the shards in your Redis \(cluster mode enabled\) cluster by rebalancing shards using the AWS Management Console\.

**To rebalance the keyspaces among the shards on your Redis \(cluster mode enabled\) cluster**

1. Open the ElastiCache console at [https://console\.aws\.amazon\.com/elasticache/](https://console.aws.amazon.com/elasticache/)\.

1. From the navigation pane, choose **Redis**\.

1. Choose the name, not the box to the left of the name, of the Redis \(cluster mode enabled\) cluster that you want to rebalance\.
**Tip**  
Redis \(cluster mode enabled\) clusters have a value of 1 or greater in the **Shards** column\.

1. Choose **Rebalance**\.

1. When prompted, choose **Rebalance**\. You might see a message similar to this one: *Slots in the replication group are uniformly distributed\. Nothing to do\. \(Service: AmazonElastiCache; Status Code: 400; Error Code: InvalidReplicationGroupState; Request ID: 2246cebd\-9721\-11e7\-8d5b\-e1b0f086c8cf\)*\. If you do, choose **Cancel**\.

### Online shard rebalancing \(AWS CLI\)<a name="redis-cluster-resharding-online-rebalance-cli"></a>

Use the following parameters with `modify-replication-group-shard-configuration`\.

**Parameters**
+ `-apply-immediately` – Required\. Specifies the shard reconfiguration operation is to be started immediately\.
+ `--replication-group-id` – Required\. Specifies which replication group \(cluster\) the shard reconfiguration operation is to be performed on\.
+ `--node-group-count` – Required\. To rebalance the keyspaces across all shards in the cluster, this value must be the same as the current number of shards\.

The following process describes how to reconfigure the shards in your Redis \(cluster mode enabled\) cluster by rebalancing shards using the AWS CLI\.

**Example \- Rebalancing the Shards in a Cluster**  
The following example rebalances the slots in the Redis \(cluster mode enabled\) cluster `my-cluster` so that the slots are distributed as equally as possible\. The value of `--node-group-count` \(`4`\) is the number of shards currently in the cluster\.  
For Linux, macOS, or Unix:  

```
aws elasticache modify-replication-group-shard-configuration \
    --replication-group-id my-cluster \
    --node-group-count 4 \
    --apply-immediately
```
For Windows:  

```
aws elasticache modify-replication-group-shard-configuration ^
    --replication-group-id my-cluster ^
    --node-group-count 4 ^
    --apply-immediately
```

### Online shard rebalancing \(ElastiCache API\)<a name="redis-cluster-resharding-online-rebalance-api"></a>

You can use the ElastiCache API to reconfigure the shards in your Redis \(cluster mode enabled\) cluster online by using the `ModifyReplicationGroupShardConfiguration` operation\.

Use the following parameters with `ModifyReplicationGroupShardConfiguration`\.

**Parameters**
+ `ApplyImmediately=true` – Required\. Specifies the shard reconfiguration operation is to be started immediately\.
+ `ReplicationGroupId` – Required\. Specifies which replication group \(cluster\) the shard reconfiguration operation is to be performed on\.
+ `NodeGroupCount` – Required\. To rebalance the keyspaces across all shards in the cluster, this value must be the same as the current number of shards\.

The following process describes how to reconfigure the shards in your Redis \(cluster mode enabled\) cluster by rebalancing the shards using the ElastiCache API\.

**Example \- Rebalancing a Cluster**  
The following example rebalances the slots in the Redis \(cluster mode enabled\) cluster `my-cluster` so that the slots are distributed as equally as possible\. The value of `NodeGroupCount` \(`4`\) is the number of shards currently in the cluster\.  

```
https://elasticache.us-east-2.amazonaws.com/
    ?Action=ModifyReplicationGroupShardConfiguration
    &ApplyImmediately=true
    &NodeGroupCount=4
    &ReplicationGroupId=my-cluster
    &Version=2015-02-02
    &SignatureVersion=4
    &SignatureMethod=HmacSHA256
    &Timestamp=20171002T192317Z
    &X-Amz-Credential=<credential>
```