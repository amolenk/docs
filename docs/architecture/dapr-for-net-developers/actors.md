---
title: The Dapr actors building block
description: A deep dive into the Dapr actors building block and how to apply it
author: amolenk
ms.date: 01/07/2021
---

# The Dapr actors building block

The actor model originated in 1973. It was proposed by Carl Hewitt as a conceptual model of concurrent computation, a form of computing in which several computations are executed at the same time. Highly parallel computers weren't yet available at that time, but the more recent advancements of multi-core CPUs and distributed systems have made the actor model popular.

In the actor model, the *actor* is an independent unit of compute and state. Actors are completely isolated from each other and they will never share memory. Actors communicate with each other using messages. When an actor receives a message, it can change its internal state, and send messages to other (possibly new) actors.

The reason why the actor model makes writing concurrent systems easier is that it provides a turn-based (or single-threaded) access model. Multiple actors can run at the same time, but each actor will process received messages one at a time. This means that you can be sure that at most one thread is active inside an actor at any time. Therefore, there is no need for any synchronization mechanisms (like locks) for data access.

## What it solves

Actor model implementations are usually tied to a specific language or platform. With the Dapr actors building block however, you can leverage the actor model from any language or platform.

Dapr's implementation is based on the [Virtual Actor pattern](https://www.microsoft.com/en-us/research/project/orleans-virtual-actors/?from=https%3A%2F%2Fresearch.microsoft.com%2Fen-us%2Fprojects%2Forleans%2F), originally introduced by Project "Orleans". In a Virtual Actor pattern, you don't need to explicitly create actors. Actors are automatically activated the first time a message is sent to the actor. The Dapr actor runtime manages how, when and where each actor runs, and also routes messages between actors. When an actor hasn't received any new message for a while, the runtime deactivates the actor and removes it from memory. Any state managed by the actor is persisted using the state management building block, and will be available when the actor re-activates.

While the actor model can provide great benefits, it's important to carefully consider the actor design. For example, having many clients call the same actor will result in poor performance because the actor cannot process more than one request at a time. The Dapr documentation provides a number of scenarios where the actor model is a good fit:

- Your problem space involves a large number (thousands or more) of small, independent, and isolated units of state and logic.
- You want to work with single-threaded objects that do not require significant interaction from external components, including querying state across a set of actors.
- Your actor instances won't block callers with unpredictable delays by issuing I/O operations.

## How it works



### References

- [Dapr supported state stores](https://docs.dapr.io/operations/components/setup-state-store/supported-state-stores/)

>[!div class="step-by-step"]
>[Previous](index.md)
>[Next](index.md)
