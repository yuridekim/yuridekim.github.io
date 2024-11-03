---
title : 'Understanding the Kubernetes Deployment Controller: Handling Resources in NewDeploymentController'
date : 2024-09-21 14:13:00 +0900
categories : [Distributed Systems, Kubernetes Source Code]
tags : [go, Kubernetes, deployments] #소문자만 가능
pinned : 0
---
# Understanding the New Deployment Controller in Kubernetes

In Kubernetes, the **NewDeploymentController** plays a crucial role in managing the lifecycle of deployments. It acts as an event handler that responds to resource changes, specifically focusing on three main functions:

1. Handling **add/update/delete** operations for **Deployments**.
2. Managing **add/update/delete** operations for **ReplicaSets**.
3. Deleting **Pods**.

## Question: Why is there only a delete function for Pods?

At first glance, it may seem inconsistent that the NewDeploymentController only registers a delete function for Pods while it handles add and update functions for Deployments and ReplicaSets. The absence of add/update operations for Pods suggests that these actions are likely managed elsewhere—specifically, by the **ReplicaSet** controller.

To dig deeper, we can refer to the [ReplicaSet controller implementation](https://github.com/kubernetes/kubernetes/blob/25aa9cd074072307a1168259c92cfefb1eafc27a/pkg/controller/replicaset). This reinforces the idea that the Deployment controller primarily oversees the ReplicaSets, which in turn manage the Pods. Thus, the Deployment controller indirectly manages Pods through the ReplicaSets.

## Why is the deletePod function necessary?

The presence of the **deletePod** function raises another question: what purpose does it serve if the **ResourceEventHandler** merely checks and does not modify the resources? The key insight here is that during the process of redeploying, specifically with the **RecreateDeploymentStrategyType**, it is critical to ensure that all existing Pods are terminated before new ones are created.

According to [Kubernetes Issue #32567](https://github.com/kubernetes/kubernetes/issues/32567) and [PR #36748](https://github.com/kubernetes/kubernetes/pull/36748), the requirement states that at most one instance of user Pods should run at any given time during a deployment recreation. If a Pod is still terminating, it may not have fully stopped before the new Pod comes up. This is why the **deletePod** function is crucial: it ensures that the old Pod is entirely deleted before proceeding with the recreation of new Pods.

The code snippet from the NewDeploymentController illustrates this:

```go
if d.Spec.Strategy.Type == apps.RecreateDeploymentStrategyType {
    // Sync if this Deployment now has no more Pods.
    rsList, err := util.ListReplicaSets(d, util.RsListFromClient(dc.client.AppsV1()))
    if err != nil {
        return
    }
    podMap, err := dc.getPodMapForDeployment(d, rsList)
    if err != nil {
        return
    }
    numPods := 0
    for _, podList := range podMap {
        numPods += len(podList)
    }
    if numPods == 0 {
        dc.enqueueDeployment(d)
    }
}
```