---
title: "Patterns of Distributed Transation"
date: 2021-11-10T15:13:17+11:00
draft: true
---

A distributed transaction is a set of operations on data that is performed across two or more data repositories. Consider 2 UPDATE SQL operations need to be performed across 2 seperate databases where both operations need to be either successful or failed. In the case, SQL Transaction which is supported on a single node is no longer sufficient.

# Two-Phase Commit Protocol (2PC)

In such transaction processing across multiple nodes, a Two-Phase Commit Protocol is sometimes implemented to coordinate all participated processes in a distributed transaction on whether to commit or abort and roll back the transaction.

The protocol consists 2 phases:

1. Prepare phase
In this phase a coordinator node prepares all partipated processes on their local node. If such process is successful on a single node, a "Yes" vote is responded to the coordinator.

2. Commit phase
In this phase, the coordinator node sends a commit command to all partipated nodes to commit the prepared processes.

![2pc example](https://jgao.io/2pc-example.png)

2PC is a strongly consistent protocol with trade in availability. During the 2PC stage, data records from all participants must be locked to prevent conflicts from concurrent writes. This trade-off introduces a number of issues:

- Some failures can result in deadlock leading to some participants never being able to resolve their transactions

- After participants send an agreement message to the coordinator, data is blocked until a commit or rollback is received. This can be very expensive and cause performance bottleneck

- The protocol is complicated to implement which often leads to maintainance overheads

## The CAP Theory

As there is no perfect system. As software engineer, one of the most important jobs is to make right trade-offs. The CAP Theory states that any distributed data store can only provide two of the following three guarantees:

- Consistency, every read receives the the latest state of data

- Availablity, every request receives a response without the guarantee that the data represents the latest state

- Partition Tolerance, system continoue to operate despite messages being dropped or delayed impacted by network between nodes

This means that in a distributed transaction where requests sent to multiple nodes must be all or nothing, if a network partition failure occurs, the system needs to decide either cancel the operation in which availablity decreases or proceed with the operation but risk inconsistency.

2PC is a CP protocol. It provides strong consistency to allow data modified across multiple nodes but cannot guarantee the availability of the data.

# Outbox Pattern


