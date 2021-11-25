---
title: "Common Pitfalls of Event Sourcing"
date: 2021-08-29T23:20:45+10:00
draft: false
tags: ["event-sourcing"]
categories: ["Architecture", "Events", "DDD"] 
author: "Jacy Gao"
---

Event Sourcing defines a different way to represent state of the data in a domain. Instead of maintaining the latest state of a single domain object in the database (CRUD), Event Sourcing promotes storing a full history of event logs which can be played sequentially to construct the current or past state of a domain object.

I started working with Event Sourcing over 5 years ago. Many lessons were learned along the journey. This article talks about common pitfalls of Event Sourcing.

# Event Sourcing is a data model

Event Sourcing is not all about events. Event, in the Event Sourcing context, is the structure of data used to construct the state of a domain object. 

In additional to storing events, Event Sourcing stores the state of data as events. Different from a CRUD model where only the latest state of a domain object is maintained, Event Sourcing stores the state in any particular time from the creation of a domain object.

Consider the following example of a bank account. In a CRUD system, a single record holds the current state of an account:
```
Account   BSB      Balance   Date
4928341   565789   $5280     01-09-2021
```

When an event occurs such as a withdrawing $100 from this bank account, a CRUD system can perform and UPDATE operation:

```
UPDATE account 
SET Balance = Balance - 100 
WHERE Account = 4928341
```

With CRUD model, there isn't any event history recorded as part of the change by default. Therefore an additional event log is usually created and stored in a different database table which should also be committed in the same transaction.

Event Sourcing, on the other hand, stores a full series of actions taken on that domain object, as per example below:

```
Account   BSB      Event             Amount    Description   Date
4928341   565789   AmountAdded       $5500     Salary        01-09-2021
4928341   565789   AmountDeducted    $100      Resturant     01-09-2021
4928341   565789   AmountDeducted    $20       Taxi          01-09-2021
4928341   565789   AmountDeducted    $100      Shopping      01-09-2021
```

The event logs are essentially the state of the bank account. By playing all the events in sequence, we can get the same current state of the account.

As it can be seen that Event Sourcing represents state through the use of Events as its data structure. Such data structure has many benifits: 

- It captures intent, purpose, or reason in the data at any particular time such as when an order is placed or when an account is closed

- It provides a full history of change of data which can also be leveraged for auditing purpose out of the box

- It enables the ability to restore the data state or roll back changes to a particular time in the past

- It improves write performance as all changes of data are append-only where no complex query or indexing is required.

- It improves read performance by reading materialized view directly from memory

- It prevents concurrent updates from causing conflicts because it avoids directly updating objects in the database.

- It fits natually within an event based application. Little additional development effort is required

Event Sourcing, at its core, is about representing state of a domain object. Event is the structure of data used to construct the state at any given time. When implementing an Event Sourcing system, design around how states are constructed and represented from events.

# Event Driven is not Event Sourcing

Event Driven Architecture has grown popularity in recent years. This architeture type is based on components communicating asynchronously through events. 

For example, the creation of a user by the User Service triggers the creation of a shopping cart in the Order service. 

In a traditional request-response model, User service inserts a new record to its local database, then calls the order service to create a shopping cart.

![Request Response](https://jgao.io/req-res-example.png)

However this solution has a few caveats. 

Firstly, the processes of user creation and cart creation should be transactional. In case of any failure either caused by unstable network or data errors, User service is responsible for managing rollback of the newly created user. This adds complexity. 

Secondly, both User service and Order service must be available for the request to be successful. This cannot be guaranteed when 2 services are maintained by seperate teams. 

Thirdly, more than one service may be instereted in the User Created event. This adds additional responsibility to the User service to ensure all downstream services can successfully process the request at the same time.

Event Driven Architecture allows requests to be queued until downstream services become available. This means that downstreams services now are responsible for processing the request. These services don't need to be available when user creation request is initiated. Furthermore, using an event stream such as Kafka enables PubSub messaging mechanism which allows many consumers react to the same event.

![Event Driven Architecture](https://jgao.io/event-driven-example.png)

All events published to the event stream can be captured and stored centrally in an Event Store. Some event streaming platforms provides an event store out of the box with additional cost. All events can be retrieved later for analytical or auditing purpose.

However, this event store should not be used for Event Sourcing.

Event sourcing is a more domain specific pattern. It does not care about any other domains and it does not require an event stream. It's sole purpose is to store its domain state as a sequence of events.These events are stored to record state changes rather than communicating.

Consider the following example of Event Sourcing:

![Event Sourcing Example](https://jgao.io/event-sourcing-example.png)

In this example, the state of a shopping cart is stored in a local event store as a sequence of events. These events are different from domain events as they are private to the order domain. These events are not intereted by other domains.

However `OrderCheckedOut` is a domain event which is published and consumed by Payment domain.

So any system which uses "event sourcing" as its core mechanics can be seen as also as an even-driven system, but the opposite is not true in general.

# Domain Events vs Event Sourcing Events

Domain Events are described as something that happens in a domain and is also important to other domains. These events are typically used in Event Driven systems as the communication mechanism between different domains. Domain events can be used as notification to inform something has happened, or state transfer from one domain to another by carrying necessary information as part of the event payload.

Examples of Domain Events can be:

- UserRegistered
- OrderPlaced
- PaymentDeadlineExpired

Domain Events are Public Events. These events are usually published to an event stream for other parts of the system to consume and process.

Event Sourcing events, on the other hand, aren't exposed to other domains. These events are usually private events processed within a specific domain. This is also called an aggregate. For example, `UserRegistration` aggregate could have the following events:

- 10:00 - EmailAddressEntered (email@example.com)
- 10:05 - VerificationCodeGenerated (123456)
- 10:06 - EmailAddressValidated
- 10:10 - UserDetailsSubmitted (`{firstname: John, lastname: Doe, gender: male, phone: +12345678}`)

These events can be replayed to construct the current and past state of the user. As in the above example, the state of User Registration is represented at any given time.

In contrast, Event Sourcing Events can be seen as private events. These events live within a specific domain. Domain Events are public events. These events move across domains.

# Command Sourcing vs Event Sourcing

[Martin Fowler described](https://youtu.be/STKCRSUsyP0?t=1392) a sucessful Event Sourcing system can fully recover an application state from a major failure by replaying the full history of events.

This aspect of Event Sourcing is very powerful but is also dangerous when Event Sourcing is implemented with Commands.

The difference between Events and Commands is subtle. Often the name of an event is just the past tense of its command. 

> "With a command you tell a system to do X. Events, with an event you let a system know that Y has happened." - [Martin Fowler](https://martinfowler.com/eaaDev/EventNarrative.html)

So a Command is a request made to the system to do something. At this point a lot of things can still happen. It can fail, it can be influenced by external state. Nevertheless, an event is something already happened and that cannot be changed.

If we look at the same order service example, the actions on the left are potentially commands. These actions can't be simply replayed. For instance replaying `Checkout` will retrigger OrderCheckedOut event to be consumed by payment service and we cannot guarantee payment service can process it as a replayed event.

However the events on the right side can be replayed because these events are private events to the order service. Any side effects can be well managed within the Order service.

A simple workflow representation of Event Sourcing can be descibed as `State -> Event -> State`. In this case, the event itself is deterministic. It has all required information to describe the state transition.

In contrast, Command Sourcing can be described as `State -> Command -> Event`. In this case, the command itself does not have all required information to determine a state change but rely on an external input. A consequence of this is that you can't simply replay a stream of logged commands at some arbitrary time and guarantee to get the same outputs as you would if they were handled immediatly. 

# TL,DR

- Event Sourcing is about representing "state", events are used as data structure
- Event Driven is not Event Sourcing, these two concepts are often implemented independently
- Domain Events are "public events" that move across domains where Event Sourcing Events are "private events" that live within a specific domain
- In Event Sourcing, events can be replayed to recover application state from failures but commands cannot be replayed in most cases

# References

https://martinfowler.com/eaaDev/EventSourcing.html

https://martinfowler.com/eaaDev/EventNarrative.html

https://www.slideshare.net/JAXLondon2014/event-sourcing-greg-young

https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing

https://verraes.net/2019/08/eventsourcing-state-from-events-vs-events-as-state/

https://www.innoq.com/en/blog/domain-events-versus-event-sourcing/

https://thinkbeforecoding.com/post/2013/07/28/Event-Sourcing-vs-Command-Sourcing
