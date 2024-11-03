---
title : 'Working with Unix Sockets'
date : 2024-11-02 01:23:00 +0900
categories: [Distributed Systems, Networking]
tags: [go, programming, sockets, error-handling, unix-domain-sockets]
pinned : 0
---

Previously, I shared a post on how my RPC calls were failing due to a race condition. This was coming from unix sockets(implemented to mirror a distributed system on a local environment). So what are unix sockets and how do they work?

# What is a Socket?
A socket servers as an endpoint for communication, allowing two processes to exchange data using system calls such as `send()` and `recv()`. Sockets can differ based on several properties, including:

- **Data type**: Stream (TCP) or datagram (UDP).
- **Communication scope**: Local-only or over the Internet.
- **Connection type**: Stateful or stateless.

And a unix domain socket, is a type of the many kinds of sockets.

## What is a Unix Domain Socket?

A Unix domain socket allows two local process on the same machine to communicate with each other. You can say it is somewhat similar in a sense to a TCP connection except that a TCP connection would need the four attributes `<src_addr, src_port, dst_addr, dst_port>`. Whereas a Unix domain socket is simply signified by a file path on your local system (e.g., `/var/tmp/12345/my_socket.sock`). You can connect to a listening socket by `connect()` and the OS would recognize this as special and connect to it. And then you can use the standard TCP/UDP socket system calls (`accept()`, `send()`, `recv()`) after the connection has been made.

Common directories for Unix domain socket files include:

- `/var/tmp`
- `/tmp`
- `/var/run`
- `/run`

However unlike TCP connections, where the OS cleans up the connection after it terminates. Unix socket files remains until you manually clean it up.

Unix domain socket errors are reported instantly since the operating system is aware of the status of both endpoints on the local machine. In contrast, TCP socket errors may not be communicated until a long timeout occurs (up to ~2 minutes).

## What is a Connection Reset?

A connection reset occurs when data is sent to an endpoint that no longer exists. The endpoint might have existed previously, but it has since been cleaned up by the operating system, leading to the data arriving at a non-existent address.

The reset is triggered by the remote endpoint. As a result, errors are not reported immediately and are most likely encountered during subsequent socket operations.

**Process A**    |  **Process B**
-----------------|-------------------
                 |  `close()`
  `send()`       |  
                 |  `RESET`
  `GotRST`       |  
                 |  `recv()`
                 |  
  `ECONNRESET`   |  

In this scenario, **Process A** attempts to send data to **Process B**, which has already closed the connection. When **Process B** receives the data, it sends a reset signal to **Process A** to indicate that the connection is no longer available, resulting in an `ECONNRESET` error during the next `recv()` call.

## What is a Broken Pipe?

A broken pipe error is similar to a connection reset, but instead of the remote endpoint indicating that the connection is broken, the local operating system provides the notification: "the connection is broken." This allows you to avoid needing to check the status of the remote endpoint before receiving a response.

**Process A**    |  **Process B**
-----------------|-------------------
                 |  `close()`
  `send()`       |  
                 |  
  `EPIPE + SIGPIPE` |  
                 |  
  `Terminated`   |  

In Unix domain sockets, the operating system can immediately detect when the remote endpoint is no longer active, but **Process A** only receives this notification during the next socket operation. For broken pipes, this notification manifests in two ways:

1. The socket operation returns an `EPIPE`, which you need to handle in **Process A**.
2. **Process A** receives a `SIGPIPE` signal that terminates the process. In Go, this signal is typically suppressed by default.

Detecting broken pipes is straightforward in Unix sockets, but this isn’t always true for other socket types. In TCP, the operating system must wait until the connection times out before it can confirm that the pipe is broken, which may take a significant amount of time.
