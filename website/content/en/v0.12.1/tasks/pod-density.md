---
title: "Control Pod Density"
linkTitle: "Control Pod Density"
weight: 20
description: >
  Learn ways to specify pod density with Karpenter
---

Pod density is the number of pods per node.

Kubernetes has a default limit of 110 pods per node. If you are using the EKS Optimized AMI on AWS, the [number of pods is limited by instance type](https://github.com/awslabs/amazon-eks-ami/blob/master/files/eni-max-pods.txt) in the default configuration.

## Max Pods

Do not use the `max-pods` argument to kubelet. Karpenter is not aware of this value. For example, Karpenter may provision an instance expecting it to accommodate more pods than this static limit.

## Increase Pod Density

### Networking Limitations

*☁️ AWS Specific*

By default, the number of pods on a node is limited by both the number of networking interfaces (ENIs) that may be attached to an instance type and the number of IP addresses that can be assigned to each ENI.  See [IP addresses per network interface per instance type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI) for a more detailed information on these instance types' limits.

Karpenter can be configured to disable nodes' ENI-based pod density.  This is specially useful for small to medium instance types which have a lower ENI-based pod density.  Set the environment variable `AWS_ENI_LIMITED_POD_DENSITY: "false"` (or the argument  `--aws-eni-limited-pod-density=false`) in the Karpenter controller to allow nodes to host up to 110 pods.

Environment variables for the Karpenter controller may be specified as [helm chart values](https://github.com/aws/karpenter/blob/c73f425e924bb64c3f898f30ca5035a1d8591183/charts/karpenter/values.yaml#L15).

{{% alert title="Note" color="primary" %}}
When using small instance types, it may be necessary to enable [prefix assignment mode](https://aws.amazon.com/blogs/containers/amazon-vpc-cni-increases-pods-per-node-limits/) in the AWS VPC CNI plugin to support 110 pods per node.  Prefix assignment mode was introduced in AWS VPC CNI v1.9 and allows ENIs to manage a broader set of IP addresses.  Much higher pod densities are supported as a result.
{{% /alert %}}

## Limit Pod Density

Generally, increasing pod density is more efficient. However, some use cases exist for limiting pod density.

### Topology Spread

You can use [topology spread]({{< relref "scheduling.md#topology-spread" >}}) features to reduce blast radius. For example, spreading workloads across EC2 Availability Zones.


### Restrict Instance Types

Exclude large instance sizes to reduce the blast radius of an EC2 instance failure.

Consider setting up upper or lower boundaries on target instance sizes with the node.kubernetes.io/instance-type key.

The following example shows how to avoid provisioning large Graviton instances in order to reduce the impact of individual instance failures:

```
-key: node.kubernetes.io/instance-type
    operator: NotIn
    values:
      'm6g.16xlarge'
      'm6gd.16xlarge'
      'r6g.16xlarge'
      'r6gd.16xlarge'
      'c6g.16xlarge'
```
