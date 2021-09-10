---
title: "Use cases for Event Sourcing"
date: 2021-08-30T23:58:26+10:00
draft: true
---

This is the 2nd blog of the series on Event Sourcing. If you are not familiar with the concept, make sure to check out [Demystifying Event Sourcing](https://jgao.io/post/event-sourcing/).

# Status
In Progress

# Event Sourcing in real life

The concept of Event Sourcing was inspired by real world examples. The primary example - the double entry accounting ledger is considered one of the earliest implementations of Event Sourcing in real life.

Why is it a great use case of Event Sourcing? If you have heard of "accountants don't use erasers", you know that in accounting, ledger entries cannot be erased and any incorrect entries are rectified by appending a new entry instead of erasing it.

The 2 takeaways from this example is auditability and immutability. By keeping every single entry to an account, we effectively get a full insights of every change to the account, even the mistakes we made.

Accounting Ledger is not the only example, consider how lawyers append terms to a contract, doctors append description to your medical records. The benefit is clear, by using Event Sourcing, you have the full auditability of every event happened to an entity. 

# Auditing vs logging

It is worth to compare the 2 concepts. There are a lot of overlapping between the two, however fundamentally they are 2 seperate concepts.

Logging is an instrument that provides visibility to the request flow of an application system which helps developers with troubleshooting. Auditability is a feature of an application system. Every system needs logging, some systems need auditing.

From an implementation perspective, logging often happens after an event or a process. For example, UserRegistered event triggers and update of the user record in the database, then emit a log message "user registered". Note that the update and logging are 2 seperate processes which means that a system failure could happen in between which can cause the loss of the log. In most situations it does not matter as there are other mechanisms to help developers understand what went wrong.

Auditability, on the other hand, cannot torlerate such failure. Therefore, certain concensus algorithms such as 2PC are often applied to ensure the ACID for the audit log. Event Sourcing addresses this problem by making "logging" the only operation to store both state and logs which lead to consistency of auditability.

# Examples of Event Sourcing Implementations

Git is often referred by Event Sourcing, However, Git is not an implementation of Event Sourcing. Git uses the snapshotting technique to keep its history of commits. This technique is commonly used with Event Sourcing. However snapshots are not events.

Database management systems are good examples of Event Sourcing. Records presented in the database are materialised by applying a series of transaction logs. The source of truth are the transaction logs. In case of any system or operation failures such as crashes or hardware failures, the records can be effectively restored or rolled back from reviewing the transaction logs.

Order Management is often partially implemented with Event Sourcing. The main intension is to address the issue with consistency across multiple operations. An event store can be used as both database and queue.

# Benifits of Event Sourcing

# Use cases of Event Sourcing

# References

https://en.wikipedia.org/wiki/Transaction_log