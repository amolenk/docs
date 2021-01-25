---
title: Dapr @ 20,000 feet
description: A high-level overview of what dapr is, what it does, and how it works
author: robvet 
ms.date: 11/01/2020
---

# Dapr @ 20,000 feet

In chapter 1, we discussed the appeal of distributed microservice applications. But, we also pointed out that they dramatically increase architectural and operational complexity. With that in mind, the question becomes, how can we "have our cake" and "eat it too?" That is, how can we take advantage of the agility, but minimize the complexity.

Ladies and Gentlemen, please welcome.... `Dapr`.

Dapr, or *Distributed Application Runtime* is a new way to build modern distributed applications.

What started as a prototype has evolved into a highly successful open-source project. Its sponsor, Microsoft, has closely partnered with customers and the open-source community to design and build Dapr. This concerted effort validates the willingness of developers from all backgrounds to solve some of the toughest challenges when developing distributed applications.     

This book looks at Dapr through the lens of the .NET Core platform. In this chapter, we help you build a solid conceptual understanding of Dapr and how it works. Later on, we present practical, hands-on instruction on how to use Dapr in your applications.

## What is Dapr?

Imagine flying in a jet at 20,000 feet. You look out the window and see the landscape from a wide perspective. Let's do the same for Dapr. Visualize yourself flying over at Dapr at 20,000 feet. What would you see?

At its core, Dapr is a `portable, event-driven runtime`. 

 - `Portable` - Your write your application code once and run it on public clouds and edge devices. With a configuration file, you specify the `external infrastructure capabilities` that you wish to include in your environment. These capabilities include different kinds of infrastructure services like pub-sub messaging and state management. They also include communication plumbing like service-to-service invocation. Later, should you move your app, you update the configuration with infrastructure and communication bindings for the new environment. Your service never holds direct references to infrastructure capabilities - Dapr does that.

  - `Event-driven` - Dapr supports event-driven resource binding and pub-sub messaging services. You can receive events from or sends events to external resources, such as datastores, streaming services, or web hooks. You can also publish and subscribe to events from message brokers - with minimal code and, again, no direct reference to the external service. 

 - `Runtime` - Dapr exposes a runtime engine that runs your services - whatever the programming platform. For example, Dotnet Core is a runtime engine - you run services within it. Dapr is *also* a runtime engine. You invoke the Dapr runtime and instruct it to start the dotnet runtime. Under the hood, Dapr implements a `distributed runtime` by operating along side with your application using a `sidecar` architecture. More about *sidecars* coming up.

## What does Dapr solve?

Dapr addresses a large challenge inherent in modern distributed applications: `Complexity`. 

Through an architecture of pluggable components, Dapr helps simplify plumbing concerns. It enables your services to bind to infrastructure services and other microservices. The runtime provides a `dynamic glue` that fuses together service component plumbing *without* tightly coupled references. For example, your application may require a state store. You could write custom code to wrap Azure Redis Cache and inject it into your service at runtime. However, Dapr greatly simplifies your experience. You instruct your  service to invoke a Dapr `building block` that dynamically binds to Redis Cache via a configuration. With this model, your service delegates the call to Dapr, which, calls Redis on your behalf. Your service has no SDK, library, or reference to Redis. You code against the common Dapr state management API, not the Redis Cache API.

Figure 2-1 shows Dapr from 20,000 feet.

![Dapr at 20,000 feet](./media/dapr-high-level.png)
**Figure 2-1**. Dapr at 20,000 feet.

In the top row of the figure, note how Dapr provides language-specific SDKs for popular development platforms. Dapr v 1.0 includes supports Go, Node.js, Python, .NET, Java, and JavaScript. This book will focus on the .NET Dapr SDK, which also provides direct support for ASP.NET Core services.

While language-specific SDKs enhance the developer experience, Dapr is  platform agnostic. Under the hood, Dapr's programming model exposes capabilities through standard HTTP/gRPC communication protocols. As the second row highlights, any programming platform can call Dapr via its native HTTP and gRPC APIs.  

The blue boxes across the center of the figure present the distributed building blocks. Each abstracts an infrastructure capability that your application can consume.

The bottom row highlights the portability of Dapr and the diverse environments across which it can run. Note that Dapr is supported across all public clouds.

## Dapr architecture

At this point, our jet turns around and flies back over Dapr, descending in altitude, giving us a closer look at how Dapr works.

### Building blocks

From our new perspective, we see that Dapr is built on the concept of `building blocks`. 

A building block is an HTTP or gRPC API that encapsulates an infrastructure capability. Figure 2-2 shows the available blocks for Dapr v 1.0.

![Dapr building blocks](./media/building-blocks.png)

**Figure 2-2**. Dapr building blocks.

The following table describes the services provided by each block.

| Building Block | Description |
| :-------- | :-------- |
| [Service-to-service communication](https://github.com/dapr/components-contrib/tree/master/nameresolution) | Invoke direct, secure service-to-service calls using platform agnostic protocols and well-known endpoints. |
| [Asynchronous messaging](https://github.com/dapr/components-contrib/tree/master/state) | Implement secure, scalable pub/sub messaging between services. |
| [State](https://github.com/dapr/components-contrib/tree/master/pubsub) | Support contextual information for long running stateful services. |
| [Observability](https://github.com/dapr/components-contrib/tree/master/bindings) | Monitor and measure message calls across networked services. |
| [Secrets](https://github.com/dapr/components-contrib/tree/master/middleware) | Securely access external secret stores. |
| [Actors](https://github.com/dapr/components-contrib/tree/master/secretstores) | Encapsulate logic and data in reusable actor objects. |
| [Resource bindings](https://github.com/dapr/components-contrib/tree/master/exporters) | Trigger code from events raised by external resources with bi-directional communication. |

Building blocks abstract the implementation of infrastructure capabilities from your services. Figure 2-3 shows this interaction. 

![Dapr building blocks](./media/building-block-integration.png)

**Figure 2-3**. Dapr building block integration.

Note how your service invokes a Dapr building block via HTTP or gRPC. Under the hood, the building block invokes pre-configured components that provide the concrete implementation for an external capability. Your code only knows about the building block. Your service takes no dependencies on external SDKs or libraries - Dapr handles the plumbing for you. Each building block is independent. You can use one, some, or all of them in your application. As a value-add, Dapr building blocks bake in industry best practices.

We provide detail explanation and code samples for each Dapr building block in the upcoming chapters. Now, our jet descends even more to give us a close look at the lower-level Dapr components layer.

### Components

While building blocks expose an API to invoke infrastructure capabilities, Dapr components provide the concrete implementation to make it happen. 

Consider, the Dapr `state store` component. It provides a uniform way to manage state for CRUD operations. Without any change to your service code, you could implement any of the following Dapr state components:

 - AWS DynamoDB
 - Aerospike
 - Azure Blob Storage
 - Azure CosmosDB
 - Azure Table Storage
 - Cassandra
 - Cloud Firestore (Datastore mode)
 - CloudState
 - Couchbase
 - Etcd
 - HashiCorp Consul
 - Hazelcast
 - Memcached
 - MongoDB
 - PostgreSQL
 - Redis
 - RethinkDB
 - SQL Server
 - Zookeeper 

Each component provides the necessary implementation through a common state management interface:

   ```go
   type Store interface {
	    Init(metadata Metadata) error
	    Delete(req *DeleteRequest) error
	    BulkDelete(req []DeleteRequest) error
	    Get(req *GetRequest) (*GetResponse, error)
	    Set(req *SetRequest) error
	    BulkSet(req []SetRequest) error
  }
   ```   

   > Note: The Dapr interface above along with all of Dapr has been written in the Golang, or Go, platform. Go is a popular language across the open source community and attests to cross-platform commitment of Dapr. 

Perhaps you start with Azure Redis Cache as your state store. You specify it with the following configuration:

   ```yaml
   apiVersion: dapr.io/v1alpha1
   kind: Component
   metadata:
     name: statestore
     namespace: default
   spec:
     type: state.redis
     metadata:
     - name: redisHost
       value: <HOST>
     - name: redisPassword
       value: <PASSWORD>
     - name: enableTLS
       value: <bool> # Optional. Allowed: true, false.
     - name: failover
       value: <bool> # Optional. Allowed: true, false.
   ```   
Note how the `spec` section specifies Azure Redis Cache. Dapr will send your state management calls to Redis Cache. Under the hood, Dapr binds to the Redis Cache component on your behalf.

If you're familiar with Kubernetes, you'll note how Dapr configuration files closely resemble Kubernetes manifest files. Essentially, they define a policy that affects how each Dapr instance behaves and the specific infrastructure component. Configuration can be applied to Dapr instances dynamically.

At a later time, you may want to migrate your state management to Azure Table Storage. Azure Table Storage provides state management capabilities that are affordable and highly durable.

While your service code would remain the same, you would modify the definition of your state component, as follows:

   ```yaml   
   apiVersion: dapr.io/v1alpha1
   kind: Component
   metadata:
     name: <NAME>
     namespace: <NAMESPACE>
   spec:
     type: state.azure.tablestorage
     metadata:
       - name: accountName
         value: <REPLACE-WITH-ACCOUNT-NAME>
       - name: accountKey
         value: <REPLACE-WITH-ACCOUNT-KEY>
       - name: tableName
         value: <REPLACE-WITH-TABLE-NAME>
   ```   
Note how the `spec` section now specifies Azure Table Storage. Based on the new configuration, Dapr will bind state management calls to Azure Table Storage.

With this model, the developer is never exposed to the Redis or Azure Table Storage APIs. Dapr abstracts these APIs away in its component and binding. 

A building block can use a combination of components. For example, the Actor and the State Management building blocks both consume a State component. As another example, the Pub/Sub building block consumes a Pub/Sub component.

At the time of this writing, the following component types are provided by Dapr:

| Component | Description |
| :-------- | :-------- |
| [Service discovery](https://github.com/dapr/components-contrib/tree/master/nameresolution) | Used by the Service Invocation building block to integrate with the hosting environment to provide service-to-service discovery. |
| [State](https://github.com/dapr/components-contrib/tree/master/state) | Provides uniform interface to interact with wide variety of state store implementations. |
| [Pub/sub](https://github.com/dapr/components-contrib/tree/master/pubsub) | Provides uniform interface to interact with wide variety of message bus implementations. |
| [Bindings](https://github.com/dapr/components-contrib/tree/master/bindings) | Provides uniform interface to trigger application events from external systems and invoke external systems with optional data payloads. |
| [Middleware](https://github.com/dapr/components-contrib/tree/master/middleware) | Allows custom middleware to plug into the request processing pipeline to perform additional actions on a request or response. |
| [Secret stores](https://github.com/dapr/components-contrib/tree/master/secretstores) | Provides uniform interface to interact with external secret stores, including cloud, edge, commercial, open-source services. |
| [Tracing exporters](https://github.com/dapr/components-contrib/tree/master/exporters) | Provides uniform interface to open telemetry wrappers. |

As our jet completes it fly over of Dapr, we look back once more and are able to see how it connects together.

### Sidecar Architecture

Dapr exposes its building blocks and components through a [sidecar architecture](https://docs.microsoft.com/azure/architecture/patterns/sidecar). This means that Dapr is deployed into a separate memory process or separate container alongside your service. Sidecars provide isolation and encapsulation as they aren't part of the service, but connected to it. This separation enables each to have its own runtime environment and be built upon different programming platforms. Figure 2-4 shows a sidecar pattern.

![Sidecar architecture](./media/sidecar-generic.png)

**Figure 2-4**. Sidecar architecture.

This pattern is named Sidecar because it resembles a sidecar attached to a motorcycle. In the previous figure, note how the Dapr sidecar is attached to your service to provide Dapr infrastructure capabilities for the service. The Dapr sidecar shares the same lifecycle as your service. It's created and retired alongside the parent. 

Figure 2-5 shows an example of a Dapr sidecar for state management.

![Sidecar architecture](./media/sidecar-state-mgmt.png)

**Figure 2-5**. Sidecar architecture for state management.

In this scenario, the search service, on the left, needs to manage application state with a state store. Note how the search service runs within its own runtime and contains a YAML-based configuration file specifying the state store implementation it wishes to consume from Dapr.

The Dapr runtime on the right exposes a State Management building block exposed as a side car. The YAML configuration specifies the exact state management component to use. 

In the middle, note how the search service and Dapr are decoupled and communicate via HTTP or gRPC.

It's important to understand that the search service has no direct dependencies to underlying state store. Instead it consumes the Dapr State Management building block, exposed as a stand-alone sidecar service. The sidecar API is responsible for calling into state store component on behalf of the search service. 

### Hosting Environments

Dapr can be hosted in multiple environments. It can self-host for local development. Dapr can also deploy to Kubernetes, a group of VMs, or edge environments such as Azure IoT Edge.

In [self-hosted mode](https://docs.dapr.io/concepts/overview/#self-hosted) Dapr runs as a separate sidecar, which your service code can invoke via HTTP or gRPC. Self-hosting modes allows for local development. Figure 2-6 shows an application and Dapr  hosted in two separate memory processes communicating via HTTP or gRPC.

![Sidecar architecture](./media/self-hosted-dapr-sidecar.png)

**Figure 2-6**. Self-hosted Dapr sidecar

Dapr also runs in [containerized environments](https://docs.dapr.io/concepts/overview/#kubernetes-hosted), such as Kubernetes. Figure 2-7 shows Dapr running in a separate side-car container along with the application container in the same Kubernetes pod.

![Sidecar architecture](./media/kubernetes-hosted-dapr-sidecar.png)

**Figure 2-7**. Kubernetes-hosted Dapr sidecar


 > Stop here -- The remaining document is still under construction


## Dapr performance considerations

Implementing a sidecar architecture, Dapr invokes multiple hops per call. Figure 2-7 presents an example of a Dapr traffic pattern.

![Dapr traffic patterns](./media/dapr-traffic-patterns.png)

**Figure 2-7**. Dapr traffic patterns

Looking at the previous figure, one might question the latency and overhead of incurred for each Dapr call. 

It's important to keep in mind that a tremendous amount of engineering effort has gone into to making Dapr efficient. As well, calls between Dapr sidecars are always made with gRPC, which delivers high performance and small, binary payloads. In most cases, the additional overhead should be less than 1 millisecond. 

To increase performance, developers should consider implementing calls across gRPC when invoking Dapr sidecars. 

gRPC is a modern, high-performance framework that evolves the age-old [remote procedure call (RPC)](https://en.wikipedia.org/wiki/Remote_procedure_call) protocol. gRPC uses HTTP/2 for its transport protocol which provides significant performance enhancements over HTTP RESTFul service, including:

- Multiplexing support for sending multiple parallel requests over the same connection - HTTP 1.1 limits processing to one request/response message at a time.
- Bidirectional full-duplex communication for sending both client requests and server responses simultaneously.
- Built-in streaming enabling requests and responses to asynchronously stream large data sets.

## Dapr and service meshes

https://docs.dapr.io/concepts/faq/

https://stackshare.io/dapr


 > See Dapr-runtime3.pdf and Dapr-runtime4.pdf
 > Orleans -- see pros-cons.pdf

Dapr can be used alongside any service mesh such as Istio and Linkerd. A service mesh is a dedicated network infrastructure layer
designed to connect services to one another and provide insightful telemetry. A service mesh doesn’t introduce new functionality to an
application.

Dapr introduces new functionality to an app’s runtime. Both service meshes and Dapr run as side-car services to your application, one
giving network features and the other distributed application capabilities.

1 dapr init --slim
Istio is not a programming model and does not focus on application level features such as state management, pub-sub, bindings etc. That
is where Dapr comes in. So, there are some similarities between the two. But, as suggested by the dapr documentation we can use them together as well. Dapr will not focus on network concerns like traffic routing, A/B testing etc. This is where service mesh like Istio will
provide value. Dapr will be more application centric







## Extra content bucket


A cloud native application often uses various cloud-based services, such
as storage services, secret stores, authentication servers, messaging backbones, and more. Dapr abstracts these
services into simple interfaces so that your applications can be reconfigured to use different service
implementations, including containerized services and hosted services, in different deployment environments


Regardless of how you deploy your Dapr sidecars, the sidecars provide local service endpoints that bring
various capabilities to your application that you can leverage without needing to worry about any infrastructural
details—this is the key value of Dapr. A cloud native application often uses various cloud-based services, such
as storage services, secret stores, authentication servers, messaging backbones, and more. Dapr abstracts these
services into simple interfaces so that your applications can be reconfigured to use different service
implementations, including containerized services and hosted services, in different deployment environments

Dapr reduces the complexity of constructing microservice and evevnt-driven applications. Companies can implement interservice-communication, state, resource binding, and asynchronous pub/sub messaging with open APIs and extensible components that are community-driven.

it greatly abstracts the complexity of distributed components and provides baked-in industry best practices.

*Dapr is a portable, event-driven runtime that reduces the complexity of building resilient, microservice stateless and stateful applications that run on the cloud and edge and embraces the diversity of languages and developer frameworks.*



 Dapr implements a sidecar architecture so that 
 dotnet core is a runtime engine - you run programs within it
 dapr is also a runtime engine - you run the dotnet program within the dapr runtime -- you tell dapr to start the service and that is with the dotnet runtime (dotnet run). So, the Dapr runtime encapsulates the .NET core runtime.


Dapr is a “runtime” — what does that mean?
I thought about this as well, when I first started exploring Dapr. In my specific case (Java middleware background), “runtime” was an application server that provided a “managed environment” (concurrency, security etc.) to my code (WAR, EAR etc.)
That’s not the case with Dapr. It is a “Runtime” that operates along with your application using a sidecar architecture — your application does not run “inside it”. In standalone mode, Dapr simply runs as a different process and in Kubernetes, it runs as a (sidecar) container in the same Pod as your application
Image for post

It is a “Runtime” that operates along with your application using a sidecar architecture — your application does not run “inside it”. In standalone mode, Dapr simply runs as a different process and in Kubernetes, it runs as a (sidecar) container in the same Pod as your application

you encapsulate and expose Dapr functionality via a side car architecture - your application does not include any Dapr runtime code.

  > Specfically, you're running the dot net service in a dapr sidecar. Now, other service can use dapr to call this service.

That’s not the case with Dapr. It is a “Runtime” that operates along with your application using a sidecar architecture—your application
does not run “inside it”. In standalone mode, Dapr simply runs as a different process and in Kubernetes, it runs as a (sidecar) container in
the same Pod as your application

It should be clear from the above picture that Dapr is a higher abstraction than Kubernetes. Kubernetes does not have any concept of application. You can use Dapr as a sidecar container with your application containers running in Kubernetes. You can also use Dapr as a process for traditional deployments as well.

For example, most applications or services that you create will require ancillary services: A database, cache, message broker, etc. These servcies are commonly referred to as `backing services.` Figure x shows backing services.

Envision that your service requires a distributed cache. Without a great of effort, you could install the Azure Redis Cache .NET libraries, write some quick code, and consume Redis Cache as a backing service. The problem here is you're now `tightly coupled` to Redis Cache. A key tentant of the `12 Factor App` calls for all backing services to be `attached services`.

You might then write a generic caching wrapper (interface) and a class that implements the wrapper with concrete Redis code. At startup, you regsiter the wrapper with your dependency injection container and inject an instance of the concrete Redis class. Now, you've decoupled your service from the caching backing service.

Dapr is not without challenges. Consider the following:

 - When consuming a service component from Dapr, you're communicating with it through an abstraction. The abstraction can expose common functionality that all such components would expose. Some components may have custom features that are not exposed by the Dapr common interfaces. In these scenarios, you may need to reference the component directly.

## End of extra content bucket


## Summary

In this chapter, we introduced Dapr blah. We provided a definition along with the key capabilities that blah. We looked at the types of applications that might justify this investment and effort.

### References

- [Cloud Native Computing Foundation](https://www.cncf.io/)

- [.NET Microservices: Architecture for Containerized .NET applications](https://dotnet.microsoft.com/download/thank-you/microservices-architecture-ebook)


>[!div class="step-by-step"]
>[Previous](index.md)
>[Next](index.md)
