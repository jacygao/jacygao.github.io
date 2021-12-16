---
title: "Patterns of Distributed Transaction"
date: 2021-12-12T15:13:17+11:00
draft: false
tags: ["distributed-transaction", "transaction-outbox", "event-sourcing", "saga", "leader-election", "raft"]
categories: ["Architecture", "Distributed-Computing"] 
author: "Jacy Gao"
---

A distributed transaction is a set of operations on data that is performed across two or more data repositories. Consider 2 UPDATE SQL operations need to be performed across 2 separate databases where both operations need to be either successful or failed. In the case, SQL Transaction which is supported on a single node is no longer sufficient.

# Two-Phase Commit Protocol (2PC)

In such transaction processing across multiple nodes, a Two-Phase Commit Protocol is sometimes implemented to coordinate all participated processes in a distributed transaction on whether to commit or abort and roll back the transaction.

The protocol consists 2 phases:

1. Prepare phase
In this phase a coordinator node prepares all participated processes on their local node. If such process is successful on a single node, a "Yes" vote is responded to the coordinator.

2. Commit phase
In this phase, the coordinator node sends a commit command to all participated nodes to commit the prepared processes.

![2pc example](https://jgao.io/2pc-example.png)

2PC is a strongly consistent protocol with trade in availability. During the 2PC stage, data records from all participants must be locked to prevent conflicts from concurrent writes. This trade-off introduces several issues:

- Some failures can result in deadlock leading to some participants never being able to resolve their transactions

- After participants send an agreement message to the coordinator, data is blocked until a commit or rollback is received. This can be very expensive and cause performance bottleneck

- The protocol is complicated to implement which often leads to maintainance overheads

## The CAP Theorem

As there is no perfect system. As software engineer, one of the most important jobs is to make right trade-offs. The CAP theorem states that any distributed data store can only provide two of the following three guarantees:

- Consistency, every read receives the latest state of data

- Availability, every request receives a response without the guarantee that the data represents the latest state

- Partition Tolerance, system continue to operate despite messages being dropped or delayed caused by network between nodes

This means that in a distributed transaction where requests sent to multiple nodes must be all or nothing. If a network partition failure occurs, the system needs to decide either cancel the operation in which availability decreases or proceed with the operation but risk inconsistency.

## Eventual Consistency

Eventual Consistency guarantees that data being updated will be eventually consistent. This means that any reads may not return the most recently updated value immediately. The model is commonly used in distributed computing as it provides a way to gain some level of consistency while maintaining high availability. 

Eventual Consistency is classified as BASE in contrast to ACID which provides Strong Consistency. BASE is an acronym of the following terms:

- Basically Available, reads and writes are available as much as possible but may not be consistent

- Soft-state, the current state is a probability without guaranteed consistency

- Eventually Consistent, after writes, data will be eventually consistent after some time

2PC is a CP protocol (as in the CAP theorem). It provides strong consistency allowing data modified across multiple nodes but cannot guarantee the availability of the data. There are alternative patterns to handle distributed transaction leveraging eventual consistency.

# Outbox Pattern

The Outbox Pattern solves the problem when a transaction includes multiple writes to local database and message channels. For example, `CreateUser` command requires both insertion of a user object to the database and publishing of an event `UserCreated` to an event stream.

![outbox-scenario](https://jgao.io/outbox-example-1.png)

In this scenario, both actions performed by the User service needs to consistently succeed or fail. 2PC could be implemented but adds unnecessary complexity and affects availability.

Outbox Pattern provides consistency by using an additional database table as an "Outbox". Messages are added to the Outbox table as part of the same database transaction. The publishing of messages is managed by a background job - a poller which retrieves newly added messages and publish them to the message channel.

![outbox-explained](https://jgao.io/outbox-example-2.png)

The design above can ensure that the user record is stored and `UserCreated` event is published consistently leveraging a database feature. However, it does have some potential issues. Firstly, any consumer of `UserCreated` event will only receive the event sometime after the user record has been inserted. This may cause inconsistency on the UI when data need to be retrieved from API across multiple services. Secondly, the Poller guarantees "at least once delivery" which means that duplicate messages could be published to the channel. For example, if there is a crash after a message has been published but before it has been recorded, when the poller restarts, it will republish the same message. As a result, when using the Outbox pattern, consumers must be idempotent if the application does not wish duplicate messages to be consumed and actioned. One option is to implement a message filter which ignores messages with any existing correlation ID of consumed messages.

# Event Sourcing Pattern

[Event Sourcing](https://jgao.io/post/event-sourcing/) is one of the best ways to guarantee that event logs are captured when a state of a domain object changes. As a result, it solves the problem when a transaction must update the state while keeping the change event in a persistent data store.

The idea comes from transaction logs in SQL server databases. Transaction logs are critical to databases as they are the actual source of truth of the data. In case of a system failure, transaction logs are used to bring the database back to a consistent state.

For example, in the following scenario, any update to the order requires both writes to the order database and logging in the Event Store. In addition, the order service needs to publish `OrderPlaced` event for the warehouse service to consume. All 3 operations must be consistently succeed or fail. 

![Event Sourcing Example](https://jgao.io/event-sourcing-example-2.png)

In this case, we can use Event Sourcing where only a single record needs to be inserted to the datastore upon a change event. This record is both used to construct the current state of the domain object and stored as event log. The representation of the data is created as a [Materialised View](https://en.wikipedia.org/wiki/Materialized_view) and stored in memory for the frontends to retrieve. Moreover, the Event Store can trigger an action on OrderPlaced event which publishes the event to a message channel. Alternatively, the publishing of events can also be managed using a poller.

![Event Sourcing Explained](https://jgao.io/event-sourcing-example-3.png)

Event Sourcing pattern is a great fit when the architecture of the application is designed around events. However, this pattern faces a few challenges:

- Unfamiliar pattern to developers which may lead to a long learning curve

- Added complexity to maintain materialised view. CQRS may need to be adopted to handle complex data requirements by the frontend

- Eventual consistency may not be suitable for applications that require real-time updates to the views.

# Saga Pattern

Saga solves the problem when a workflow of business transactions span multiple services or systems must be performed consistently. For example, a successful order process involves order placement, stock update, payment and delivery. Each step is managed by a separate service with its own local database.

![Saga Example](https://jgao.io/saga-example-1.png)

The steps of the order process essentially form a workflow. In a real life scenario, any step could fail. If there isn't an automated way to recover such a system from failure, the cost of maintenance will be significant.

Saga uses a message channel to coordinate all services involved in a workflow. For large systems an event stream is often used where for small applications a message queue can be sufficient.

There are 2 types of coordination in Saga, Choreography and Orchestration.

## Choreography

In a Choreography Saga, services coordinate by communicating directly with each other via a message channel.

![Saga Choreography Example](https://jgao.io/saga-choreography-example.png)

The Choreography Saga is more suitable for simple workflows with few participants. It can be confusing to add more services to the Saga once deployed. It is also difficult to get an overview of what is happening during an end-to-end transaction since there isn't a coordinator.

## Orchestration

In an Orchestration Saga, services are coordinated by an orchestrator.

![Saga Orchestration Example](https://jgao.io/saga-orchestration-example.png)

The Orchestration Saga is more complex to implement. Also, the Orchestrator becomes a single point of failure. Nevertheless, it is much easier to add or remove steps as the orchestrator centrally controls the flow of activities. The status of workflow can be tracked in the Orchestrator. Services become more loosely coupled as they do not need to communicate to any other services other than the Orchestrator.

## Failures and Rollback

When a step of a workflow fails, there are a number of ways to recover the Saga from a failure. 

Firstly, services can retry the same message in case of any transient failures. To support retry, the implementation of message consumers must provide idempotence to ensure the consistency of data.

Secondly, the workflow can be rolled back from the failed step. The previous steps can either roll back or compensate the changes where applicable. However, the data already been committed to the local database of a service can't be rolled back.

Thirdly, in some situations, the workflow can go into a pending state. An operator can be alerted to fix the broken step and resume the workflow via an admin interface.

# Reliability Queue Pattern

A series of transactions must all succeed or fail can be broken into a list of sequential tasks. These tasks can be put onto a message queue to improve reliability. Competing Consumers can be implemented to allow horizontal scalability for processing large amount of tasks concurrently.

For example, a payment integration requires a list of calls to different API endpoints sequentially to complete the payment process.

![reliability queue use case](https://jgao.io/reliability-queue-use-case.png)

In this business scenario, 2 types of failures could happen:

- Transient failure, either caused by unstable network or downtime of the Payment Gateway

- Data logic failure, caused by invalid data type, structure or value in the request sent to the Payment Gateway

To recover from these failures, a reliability queue can be used to mitigate transient failures and allow invalid data to be repaired.

![reliability queue example](https://jgao.io/reliability-queue-example.png)

This design transforms the integration into a sequence of tasks. By using a message channel, it decouples the coordination logic from the message delivery. It also improves reliability and resiliency as requests will not be lost upon consumer failure or network outages. In case of data logic failure, a message filter can be implemented alongside the consumer logic to filter and fix malformed messages before it is actioned by the consumers.

There are a few things to watch out for when using this pattern:

- Retry interval should be defined in the message metadata to avoid excessive retry requests sent to the integrated systems by the consumers. Consider having an exponential backoff based on the number of attempts

- The integrated systems should support idempotence. It is possible that the same request is sent again by the consumer as systems could fail after a successful response but before it is added to the reply channel

# Coming Next
- Leader Election Pattern