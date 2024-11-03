---
title : 'etcd: A Deep Dive into Key-Value Storage and RAFT Consensus'
date : 2024-10-04 02:11:00 +0900
categories : [Distributed Systems, Kubernetes Source Code]
tags : [database] #소문자만 가능
pinned : 1
image:
    path: /assets/img/posts/etcd_read.png
---
# What is etcd?

etcd is a distributed key-value store that plays a critical role in the control plane of many cloud-native applications. The name "etcd" combines "etc" with "distributed," highlighting its functionality in distributed systems. Two years into playing with Kubernetes and strange I have never heard of this.

### Key Features

- **Key-Value Storage**: Stores data as key-value pairs.
- **Distributed System**: Ensures reliability and fault tolerance across multiple nodes.
- **Control Plane**: Manages and coordinates services in a distributed environment.

### Core Concepts
etcd is built on the RAFT consensus algorithm, which enables leader election and ensures fault tolerance. Here's a breakdown of some of its components:

- **RAFT**: The consensus algorithm that etcd uses.
- **Leader Election**: A mechanism to select a leader node that manages data updates and replication.
- **Fault Tolerance**: The ability to maintain service availability even in the presence of failures.

### Architecture

![etcd Architecture](/assets/img/posts/etcd_arch.png)

Each etcd server participates in a cluster and communicates with others using gRPC. 

![etcd grpc](/assets/img/posts/etcd_grpc.png)

The underlying data store uses **BoltDB**, a golang key-value database that provides a simple way to persist data to disk.

![etcd Read Flow](/assets/img/posts/etcd_read.png)
![etcd Write Flow](/assets/img/posts/etcd_write.png)

### How It Works
Now hopping onto the etcd source code.

### Leader
You can see that etcd will always have a leader, with ticks to keep the system updated.
![etcd Leader](/assets/img/posts/etcd_leader.png)


When an etcd node receives a message, it implements the step function within the RAFT struct to handle the message appropriately based on its role (leader, follower, candidate).

#### Node Input

![etcd Node Input](/assets/img/posts/node_input.png)

#### Node Output

![etcd Node Output](/assets/img/posts/node_output.png)

### RAFT Code Implementation

Each node's state transitions occur within the RAFT implementation. During a step, the node performs actions based on whether it is a leader, follower, or candidate.

![etcd Raft](/assets/img/posts/etcd_raft.png)

### RAFT Logs

RAFT logs capture the changes that occur during the consensus process.

When a message is sent to the leader, it adds the message to its local message queue, and the communication between nodes occurs through a user-implemented transportation layer. Disk I/O also requires custom implementation for persistence.

## The Server Bootstrap Process

At its core, etcd's server initialization follows a clean, hierarchical pattern:
```go
EtcdServer.Start() 
  → start() 
  → run() (as goroutine)
  → raft.start()
```
This might look simple, but there's a lot happening under the hood. When the server starts, it kicks off an event handling loop that manages the distributed consensus protocol. This is critical for maintaining data consistency across the cluster.

We wait for a node to become ready:
![etcd Ready Node](/assets/img/posts/node_ready.png)

### Message Flow and Ready States
One of the most interesting aspects of etcd is how it handles message states. During node initialization, a Ready struct is created. When a message proposal comes in (a msg propose), it gets added to a local queue. The system then checks if it's ready for processing using a condition like this:

```go
if len(r.msgs) > 0 || len(r.raftLog.unstableEntries()) > 0 {
    return true
}

rd = n.rn.readyWithoutAccept()
readyc = n.readyc
```
This simple check is crucial - it ensures that messages are processed only when the system is truly ready to handle them.

### Channel Communication

In the etcd server, messages are sent using channels, which are defined for different message types. This allows for efficient message handling and processing.

```go
// Write-only channel example
func sendData(ch chan<- int) {
    ch <- 42 // Sending data into the channel
}

// Read-only channel example
func receiveData(ch <-chan int) {
    data := <-ch // Receiving data from the channel
    fmt.Println(data)
}
```
Regular messages go through writec, but there's special handling for snapshots. Why? Because snapshots can be massive, and they need a pipeline approach to handle their size efficiently.

### Logic

![etcd Pick](/assets/img/posts/etcd_pick.png)
A channel selection mechanism that handles different message types - uses pipelines for large snapshot messages (>1GB) and regular writec channels for normal messages

![etcd Send](/assets/img/posts/etcd_send.png)
The core sending logic that implements mutex locking for thread safety and uses the pick function to determine appropriate channel for message delivery

![etcd Process](/assets/img/posts/etcd_process.png)
Follower node message handling logic that processes incoming messages and handles configuration changes through the raft protocol

![etcd Run](/assets/img/posts/etcd_run.png)
Server initialization with metadata configuration including confState, snapshots, and term/index tracking

## Alternatives to etcd

When creating a kubeapiserver, there are several options for backend storage. While etcd is the default choice due to its robust performance and features, it may be resource-intensive for certain applications. Consequently, exploring alternative storage solutions can be beneficial, especially in environments with constrained resources.

### Third-Party Storage Solutions

Other third-party storage solutions can implement similar interfaces to etcd, allowing for seamless integration. Here are a few notable alternatives:

1. **Kine**: Kine is a lightweight database option that provides an etcd-compatible API. It is designed for use with K3s, a minimalistic Kubernetes distribution. Kine supports various backends, including SQLite, PostgreSQL, and MySQL.

2. **PostgreSQL**: Another promising alternative, PostgreSQL can be utilized to replace etcd, providing a relational database option for managing Kubernetes resources. The article [Goodbye ETCD, Hello PostgreSQL](https://betterprogramming.pub/goodbye-etcd-hello-postgresql-running-kubernetes-with-an-sql-database-7e1b2e9b5f8f) provides insights into this transition.

3. **KamaJi**: The KamaJi project utilizes alternatives to etcd, demonstrating the viability of different backend storage solutions. More information can be found on their [official site](https://kamaji.clastix.io/).

4. **Martin Heinz’s Blog**: This blog offers guidance on alternatives to etcd, highlighting various implementations and configurations. Check it out [here](https://martinheinz.dev/blog/100).

### Installation

To install Kine, you can follow these general steps, although the specific commands may vary based on your Kubernetes setup:

1. **Install K3s**: Since Kine is integrated with K3s, installing K3s will automatically set up Kine as the default datastore.

   ```bash
   curl -sfL https://get.k3s.io | sh -
   ```
2. **Configure Kine**: If you're using K3s, Kine is already configured. However, if you wish to use Kine with a standard Kubernetes setup, you can deploy it as a service using Helm or manifests, pointing to your preferred database backend.

3. **Verify Installation**: Once installed, you can verify Kine's operation by checking the Kubernetes pods:

```bash
kubectl get pods -n kube-system
```
4. **Modify Configuration**: If you're working with a kind cluster for testing, you can view and modify the configuration by running:

```bash
kubectl edit cm -n kube-system kubeadm-config
```