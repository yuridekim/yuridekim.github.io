---
title : 'Kubernetes Source Code: Kubernetes Scheduler code walkthrough'
date : 2024-08-30 14:14:00 +0900
categories : [Distributed Systems, Kubernetes Source Code]
tags : [database] #소문자만 가능
pinned : 0
image:
    path: /assets/img/posts/schedulePod.png
---
# Understanding Kubernetes Scheduling: Code Walkthrough

## Run()
In the `Run()` function, the following code is executed:

```go
go wait.UntilWithContext(ctx, sched.ScheduleOne, 0)
```
- The function **blocks until the context is canceled.**
- The comment explains that the ScheduleOne function is run in a `separate goroutine` because it is blocking (i.e., it waits until the next item is available from the SchedulingQueue).
- If this blocking code were run in the same goroutine as the main `Run` function, it could prevent the `SchedulingQueue` from shutting down properly, resulting in a deadlock, where the program would freeze and be unable to proceed or shut down.
- `wait.UntilWithContext(ctx, sched.ScheduleOne, 0)`: This function call runs `sched.ScheduleOne` repeatedly in the new goroutine. It continues running `sched.ScheduleOne` until the provided context ctx is canceled. The second parameter 0 likely means there is no delay between consecutive calls.

After scheduling, the context waits for a signal to close the scheduling queue:

```go
<-ctx.Done()
    sched.SchedulingQueue.Close()
```
The SchedulingQueue is closed when the context is done.

### Types of Queues
- Active Queue: This queue contains pods that are scheduled.
- Backoff Queue: This queue holds pods that have failed to schedule.
These queues are implemented using a PriorityQueue.

![priority_queue](/assets/img/posts/priority_queue.png)
If you take a notice at how the Priority Queue is actually implemented, internally, it uses a heap.
- some notable field is `Unschedulable Pods`: Some pods cannot be scheduled at all.


## scheduleOne
Within the scheduleOne function, pod scoring is managed through the schedulingCycle.

![schedulePod](/assets/img/posts/schedulePod.png)
### Logic Breakdown

1. findNodesThatFitPod

If a pod is UnschedulableAndUnresolvable, it cannot be forcibly assigned to a node via preemption, leading to the suspension of the scheduling cycle.
For pods that are currently unschedulable but might be schedulable later, other filters are checked.

```go
checkNode := func(i int) {
    // We check the nodes starting from where we left off in the previous scheduling cycle,
    // ensuring all nodes have an equal chance of being examined across pods.
    nodeInfo := nodes[(sched.nextStartNodeIndex+i)%numAllNodes]
    status := fwk.RunFilterPluginsWithNominatedPods(ctx, state, pod, nodeInfo)
}
```
Nodes are examined starting from the last node checked in the previous scheduling cycle, ensuring a fair chance for all nodes.

2. Scoring Pods
Pods are scored based on how well they fit the available nodes.

3. selectHost
![selectHost](/assets/img/posts/selectHost.png)
The selection process employs reservoir sampling, where the probability of the j-th node being chosen is 1/j.


## Binding
When a node is selected for scheduling, it is assumed to be bound, with actual binding occurring asynchronously in a goroutine.

![binding](/assets/img/posts/binding.png)

# Framework
The Framework interface manages the set of plugins utilized in Kubernetes scheduling, which include:

- Filter Plugin: Filters nodes based on specified criteria.
- Score Plugin: Assigns scores to nodes based on how well they fit the pods.
- Prefilter Plugin: Additional filtering before the main scheduling process.
- Bind Plugin: Handles the binding of pods to nodes.

![plugins](/assets/img/posts/plugins.png)

This is how the the filter plugins are instantiated.
If no score plugin is defined, a simple round-robin approach is utilized for scheduling.