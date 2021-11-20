---
title: "Patterns of Distributed Transation"
date: 2021-11-10T15:13:17+11:00
draft: true
---

A distributed transaction is a set of operations on data that is performed across two or more data repositories. Consider 2 UPDATE SQL operations need to be performed across 2 seperate databases where both operations need to be either successful or failed. In the case, SQL Transaction which is supported on a single node is no longer sufficient.

# Two-Phase Commit Protocol (2PC)

In such transaction processing across multiple nodes, a Two-Phase Commit Protocol is sometimes implemented to coordinate all participated processes in a distributed transaction on whether to commit or abort and roll back the transaction.

The protocol consists 2 phases:

1. commit-request phase, in which the coordinator prepares all partipated processes on their local node. If such process is successfuly on a single node, a "Yes" vote is responded to the coordinator.
2. commit phase, 