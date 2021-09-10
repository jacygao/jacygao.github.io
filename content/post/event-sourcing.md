---
title: "Demystifying Event Sourcing"
date: 2021-08-29T23:20:45+10:00
draft: false
tags: ["event-sourcing"]
categories: ["Architecture", "Events", "DDD"] 
author: "Jacy Gao"
---

The inspiration behind this blog is from numerous great conversations while working with ThoughtWorks on a Greenfield International Finance Platform. As I started to gather relevant information from all over the place, I decided to write this blog to organise and share my thoughts.

# Definition

I'll start by looking at the definitions.

Martin Fowler and Greg Young have both written great definitions of Event Sourcing:

[Martin Fowler, 2005](https://martinfowler.com/eaaDev/EventSourcing.html):

>"Event Sourcing ensures that all changes to application state are stored as a sequence of events."

[Greg Young, 2014](https://www.slideshare.net/JAXLondon2014/event-sourcing-greg-young):

>"Event Sourcing says all state is transient and you only store facts."

Another good definition I came cross is in this [Microsoft's article]((https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)):

>"Instead of storing just the current state of the data in a domain, use an append-only store to record the full series of actions taken on that data."

Two key words here are "state" and "events". One for my early confusions about Event Sourcing was because I was overly focusing on "events" but overlooked "state". In fact, Event Sourcing, at its core, is about storing and representing "state as events".

# State as Event vs State from Event

Typically, state is maintained in a datatbase. Event logs are sometimes emitted when we update the state but the source of truth is the state in the database. Event Sourcing, on the other hand, promotes the event log to be the source of truth. The storage used to keep event logs is often called Event Store. All updates are only appended as new events, and no longer as state changes.

The double-entry accounting ledger is the primary example of Event Sourcing. Consider the following record of the balance of a bank account:
```
Account   BSB      Balance   Date
4928341   565789   $5280     01-09-2021
```
On every transaction, the account balance needs to be updated. An event log of that transaction is usually stored in a different database table or a separate storage.

With Event Sourcing, data is stored as a sequence of events, as per example below:
```
Account   BSB      Type     Amount    Description   Date
4928341   565789   Credit   $5500     Salary        01-09-2021
4928341   565789   Debit    $100      Resturant     01-09-2021
4928341   565789   Debit    $20       Uber          01-09-2021
4928341   565789   Debit    $100      Shopping      01-09-2021
```
Now the event logs are essentially the state of the bank account. By playing all the events in memory, we can get the exactly same current state of the account.

Event Sourcing can be described as "State as Event". It should be differentiated from "State from Events". The distinction is important. “State from Events” consumes events from an existing event stream or event bus, regardless of how it was produced, and projects state from it. This approach is often associated with an Event Driven Architecture and Domain Driven Design.

# Domain Events vs Event Sourcing Events

Domain Events are described as something that happens in the domain and is important to domain experts. Such events typically occur regardless of whether or to what extent the domain is implemented in a software system.

Examples of Domain Events can be:

- UserRegistered
- OrderPlaced
- PaymentDeadlineExpired

Domain events are typically used for communication and collaboration between domains, such as an event notification to inform something has happened, or state transfer from one domain to another by carrying necessary information as part of the event payload.

Domain Events are Public Events. These events are published to an event stream for other parts of the system to consume and process. 

Event Sourcing events, on the other hand, aren't exposed to other domains. These events are usually private events processed within a specific domain. This is also call an aggregate. For example, `UserRegistration` aggregate could have the following events:

- EmailAddressEntered (email@example.com)
- VerificationCodeGenerated (123456)
- EmailAddressValidated
- UserDetailsSubmitted (`{firstname: John, lastname: Doe, gender: male, phone: +12345678}`)

By playing the series of events, we have the current state of the user.

```
email               ver_code    isValid    details
email@example.com   123456      true       {firstname: John, lastname: Doe, gender: male, phone: +12345678}
```

In contrast, Event Sourcing Events can be seen as private events. These events live within a specific domain. Domain Events are public events. These events move across domains.

# Command Sourcing vs Event Sourcing

Replayablily is often mentioned as one of the benefits of Event Sourcing. However, many times people are referring to Command Sourcing.

[Martin Fowler 2006](https://martinfowler.com/eaaDev/EventNarrative.html):

>"With a command you tell a system to do X. Events, however, just communicate that something happened - with an event you let a system know that Y has happened."

According to the definition from Martin, a Command is a request made to the system to do something. At this point a lot of things can still happen. It can fail, it can be influenced by external state. Nevertheless, an event is something already happened and that cannot be changed.

A simple workflow representation of Event Sourcing can be descibed as `State -> Event -> State`. In this case, the event itself is deterministic. It has all required information to make the state transition and perform an "closure".

In contrast, Command Sourcing can be described as `State -> Command -> Event`. In this case, the command itself does not have all required information to determine a state change but rely on an external input. A consequence of this is that you can't simply replay a stream of logged commands at some arbitrary time and guarantee to get the same outputs as you would if they were handled immediatly. 

The replayablily of Event Sourcing is about replaying the events in an event store, sometimes from a most recent snapshot, to recreate the current state of the records as opposed to replaying the commands in which in that case, is Command Sourcing.

# TL,DR

- Event Sourcing is first and foremost about "state", and then "events"
- Event Sourcing is about "State as Event" where Event Driven is about "State from Event". Event Sourcing is different from Event Driven
- Domain Events are "public events" that move across domains where Event Sourcing Events are "private events" that live within a specific domain
- Be very careful with the replayability of Event Sourcing. Often it is misintepreted as something else such as Command Sourcing

# References

https://martinfowler.com/eaaDev/EventSourcing.html

https://martinfowler.com/eaaDev/EventNarrative.html

https://www.slideshare.net/JAXLondon2014/event-sourcing-greg-young

https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing

https://verraes.net/2019/08/eventsourcing-state-from-events-vs-events-as-state/

https://www.innoq.com/en/blog/domain-events-versus-event-sourcing/

https://thinkbeforecoding.com/post/2013/07/28/Event-Sourcing-vs-Command-Sourcing
