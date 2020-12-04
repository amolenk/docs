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



## Applying Dapr to eShop

Adding sidecars to services.

For the Dapr version of eShop, each service now has a Dapr sidecar.
* That makes it possible to use the Dapr building blocks.
* You can see that we also added a sidecar to the Envoy API Gateway. 
* That enables us to use the service invocation building block to call the backend services.



[Overview of building block functionality in eShop]

Link to next building block chapter for details.




## Summary


### References

https://docs.dapr.io/getting-started/install-dapr/



>[!div class="step-by-step"]
>[Previous](index.md)
>[Next](index.md)