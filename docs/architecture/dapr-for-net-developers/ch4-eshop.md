---
title: Introduction to eShop
description: An overview of the eShop .NET Core sample application 
author: amolenk 
ms.date: 10/20/2020
---

# eShopOnDapr

In the previous sections, you've learned about Dapr and how it can help you build distributed applications. But to appreciate the benefits, it's best to see Dapr in action. After all, the proof is in the pudding.

To showcase the benefits of Dapr in a realistic scenario, we've changed (or, *Daprized* if you will) the [eShopOnContainers .NET Core reference application](https://github.com/dotnet-architecture/eShopOnContainers) to use Dapr. In the next couple of sections, we'll quickly introduce eShop and show you how you can run the application yourself. You can find the result, *eShopOnDapr*, at <https://github.com/dotnet-architecture/eShopOnDapr>.

## eShop architecture

eShopOnContainers is a sample .NET Core reference application created by Microsoft. It uses a simplified microservices architecture and Docker containers. The reference application is cross-platform, thanks to .NET Core services capable of running on Linux or Windows containers depending on your Docker host.

We removed some of the functionality and services from the original eShop application to focus on the changes that we wanted to make by applying the Dapr building blocks. The diagram below shows the streamlined solution architecture:

![eShopOnContainers architecture](./media/eShopOnContainers-architecture.png)

The eShopOnContainers front-end consists of a Single Page Application written in Angular. The front-end sends requests to an application gateway implemented using [Envoy](https://www.envoyproxy.io/), an OSS high performant edge and service proxy. Envoy routes the incoming requests to the various back-end services. Most of the requests are simple CRUD requests (for example, get the list of brands from the catalog) and handled by a single back-end service. Some requests, however, are logically more complex and require multiple services to work together. For these cases, eShopOnContainers uses an aggregator service that mediates the work across the various services involved in the operation. [TODO Make this a numbered list, referencing the diagram]

## Benefits of applying Dapr to eShop

> This section needs a lot of editing. For now, it's just a list of benefits mailed to Mark ;-)

- eShopOnContainers uses a mix of HTTP/REST and gRPC for communication between services. This has all been replaced with Dapr Service Invocation. This solution gives us a single, easy way to communicate between services, while still giving us gRPC performance between sidecars (which are the most important calls from a perf perspective). The other benefits we get from using Dapr Service Invocation are the out-of-the-box features it provides such as mTLS and automatic retries.

- eShopOnContainers supports both Azure Service Bus and Rabbit MQ. This pattern allows you to use Service Bus while running in production on Azure, but use RabbitMQ for development and testing in local environments. An `IEventBus` abstraction layer is implemented to make this work. The implementation for this layer uses around 700 lines of code. In eShopOnDapr, we've created a single implementation of `IEventBus` that uses the pub/sub building block to communicate with the different pub/sub platforms. This implementation uses 35 lines of code. That's only 5% of the lines of code needed for the custom implementation in eShopOnContainers. Part of the reason for this reduction is the integration with ASP.NET Core for subscribing to events. Instead of having to write a separate message handler loop, we can use attributes on ordinary ASP.NET Core Controllers to subscribe to messages. This has the added benefit of having a single place where all external commands/events come in, whether it's via HTTP/REST, gRPC, or messaging. With Dapr, we now support many more pub/sub platforms in addition to Azure Service Bus and RabbitMQ, such as Redis Streams, Apache Kafka, and NATS.

- By using the Service Invocation and Publish & Subscribe building blocks, we've gained rich distributed tracing for both direct and pub/sub calls between services without having to write any code.

- The eShopOnContainers solution contained a *to-do* item for e-mailing an order confirmation to the customer. Using Dapr, we could very quickly implement this feature using an output binding. We didn't need to learn any external APIs or SDKs.

## Summary

### References

>[!div class="step-by-step"]
>[Previous](index.md)
>[Next](index.md)