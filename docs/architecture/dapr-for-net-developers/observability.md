title: Dapr Observability
description: A description of the observability features of Dapr and how to apply them
author: edwinvw
ms.date: 01/09/2020

# Dapr observability

When building a distributed system (say a microservices based solution), it is extremely important to have a good idea of what's going on with the services while they run in production. We often refer to this as **observability**. Having observability, assures insight into the health of the application at all times. This is invaluable for effectively monitoring and troubleshooting the application. 

The information used to gain observability can roughly be divided into the following four categories: 

1. The **health** of the services. This gives insight into whether all services are up & running and accepting requests?
1. **End-to-end tracing** of the communication between the services. Which messages are exchanged between the services and which services are involved in executing some business scenario? This information includes request/response and message-based communication.  
1. The **Logging** from the services. Which code is being executed and did any errors occur?
1. Several **metrics**: how is the performance of a service and how many resources does it consume?

Whether or not the information in all categories is available, depends on the observability features offered by the platform your application runs on. An example of a platform that offers all the categories is Microsoft Azure for instance. Azure offers a service called **Application Insights**. This service is automatically configured for most of the available Azure services. When an application is built using these services and it is deployed and running in Azure, several components gather information from the services and automatically sent this to App Insights. This includes logging from the application code, exceptions that occurred in the code, metrics about the resource utilization of the services, duration and status-code of all requests sent to service and lots more. App Insights is even capable of automatically drawing a diagram with the dependencies between services based on their communication.   

## What it solves

The former section introduced Azure Application Insights as a tool for offering observability out of the box for several Azure services. This is some pretty awesome tech! But what to do when the application is not comprised of standard Azure services? Is it still possible to leverage something like App Insights for such an application? Well, there are several libraries available from Microsoft to integrate an application with App Insights. The way the application emits logging information probably has to be changed. And during the startup of the application, some initialization code for App Insights has to be executed This is all great! But, a drawback of this approach is that the application is now tightly coupled to Application Insights (through its libraries) and contains App Insights specific code. This is what Dapr can solve.

Dapr also offers several observability features out of the box. As stated in [Chapter X: TODO: link to chapter that describes sidecar pattern](#), Dapr uses a sidecar architecture pattern. The Dapr sidecars are able to collect logging from the services and intercept all communication between them. This information is published by the sidecar using protocols that are based on open-standards like **OpenTelemetry** (this will be explained in more detail later in this chapter). Because of this design, Dapr is able to integrate with several different monitoring backends, including Application Insights. And the cool thing is: the application code does not need to know anything about observability - it's all handled by Dapr. 

## How it works

**TODO**

## Using the Dapr .NET SDK

**TODO**

## Observability components

**TODO**

### Configuration (TODO Move generic text to getting started)

**TODO**

## Reference architecture: eShopOnDapr

**TODO**

## Summary

**TODO**

## References

**TODO**