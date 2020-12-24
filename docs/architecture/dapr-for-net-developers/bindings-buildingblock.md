---
title: The Resource Bindings building block
description: A description of the Resource Bindings building-block and how to apply it
author: edwinvw
ms.date: 11/29/2020
---

# The Resource bindings building block

https://docs.dapr.io/developing-applications/building-blocks/bindings/bindings-overview/

https://docs.dapr.io/developing-applications/building-blocks/bindings/howto-triggers/
https://docs.dapr.io/developing-applications/building-blocks/bindings/howto-bindings/

https://www.sivamuthukumar.com/event-driven-app-with-dapr-azure-event-hubs

https://itnext.io/tutorial-using-azure-event-hubs-with-the-dapr-framework-81c749b66dcf

https://itnext.io/how-to-integrate-dapr-and-azure-event-hubs-using-kafka-bindings-44fb8de647cb




<!-->
Dapr terminology--Trigger is an input binding
-- Binding is an output binding 
-->

Cloud-based serverless offerings, such as Azure Functions and AWS Lambda, have gained significant popularity in recent years. A key benefit of these platforms is the ability to easily bind to a wide variety of external resources without complexity or extensive plumbing. External resources include datastores, message systems, and even web resources like Twitter and SendGrid.

The [Dapr resource binding building block](https://docs.dapr.io/developing-applications/building-blocks/bindings/bindings-overview/) provides direct resource binding capabilities for your services while abstracting away the complexity and plumbing.

## What it solves

The resource binding building block is two-fold: It exposes an event-driven mechanism for a service to consume inputs from an external system. It also provides a built-in mechanism to invoke external systems - both without taking dependencies on the external service.

For example, your service could consume an input binding from Twitter that raises an event when a tweet with a certain keyword is published. Your service could also implement an output binding that sends an SMS message about that tweet using Twilio. We'll expand upon that example in the section.

At first glance, the behaviors of resource bindings look similar to that of the [Publish/Subscribe pattern](publish-subscribe-buildingblock.md) described earlier in this book. Although they share similarities, there are significant differences: 

- Resource bindings support a wide variety of external resources while publish/subscribe focuses on message brokers.
- Resource bindings implement a push rather than pull mechanism.
- An application can easily switch resource bindings at runtime without code changes. Doing the same with the Publish/Subscribe building block can be more difficult. 

## How it works

With the Dapr resource binding building block, a service does not require SDKs or libraries to bind to external resources. Instead, a service binds to an external resource and receives events using an Dapr input binding and a Dapr sidecar. The same service can also trigger events on external resources using a Dapr output binding a sidecar. The following sections detail both scenarios.


<!--
The building block uses a Dapr sidecar to trigger input bindings configured in the target service. Output bindings in the service trigger a resource using a sidecare, using either HTTP or gRPC.

With Dapr, SDKs or libraries are not required to use resource bindings. 
Instead, the resource binding building block  The Dapr sidecar will use input bindings to trigger your application by invoking a public HTTP endpoint that you provide. You trigger output bindings from your application by calling an API on the Dapr sidecar - either using HTTP or gRPC. 

-->

### Input bindings

You start by adding a yaml-based file called a *binding configuration* to your service. This file describes the binding details (an example is presented later in the *Components* section.) You then couple a public HTTP endpoint for your service to the binding. Figure 8-1 shows the architecture:

![Input binding](media/bindings-input.png)

**Figure 8-1**. Dapr input binding.

The previous figure shows the steps for adding an input binding:

1. The Dapr sidecar reads the binding configuration file and subscribes to events specified for event source. In the example, the event source is Twitter. 
2. When a matching Tweet is published on Twitter, the Dapr sidecar consumes it.
3. The Dapr sidecar invokes the endpoint configured for the binding in the service. In the example, the service listens for an HTTP POST on the `/tweet` endpoint on port 6000. The endpoint is equal to the name of the binding specified in the configuration. Because it is an HTTP POST operation,  the JSON payload for the event is passed in the request body.
4. After handling the trigger, the service returns an HTTP status code `200 OK`.

As an example, the following ASP.NET Core controller handles the call triggered by the Twitter binding:

```csharp
[ApiController]
public class SomeController : ControllerBase
{

    //...

    [HttpPost("/tweet")]
    public ActionResult Post(JsonElement data)
    {
        // handle tweet

        //...

        // acknowledge message
        return Ok();
    }
}
```

If something were to go wrong handling the event, you would return an HTTP status code other than `200 OK`, a 400 or 500 level error. If the binding resource offers at-least-once-delivery guarantees, the Dapr sidecar will retry the trigger. Check out [the documentation of the different bindings](https://github.com/dapr/docs/tree/master/concepts/bindings) to see whether they offer at least once or exactly once delivery guarantees.

### Output bindings

Dapr also offers an *output binding*. It enables your service to invoke an external resource. Output bindings 
When you configure an output binding for your application, you can use it by invoking the bindings API on the Dapr sidecar of your application. Figure 8-2 shows the architecture of an output binding:

![Output binding](media/bindings-output.png)

**Figure 8-2**. Dapr output binding.

1. Your application invokes the `/v1.0/bindings/sms` endpoint on the Dapr sidecar. In this case, it uses an HTTP POST to invoke the API. It is also possible to use gRPC. We will dive into that when we describe bindings support in .NET SDK.
2. The Dapr sidecar calls the external messaging system to send the message. The message will contain the payload passed in the POST request.

As an example, the following method implements an output binding in a ASP.NET Core application:

```c#
private async Task SendSMSAsync(IHttpClientFactory clientFactory)
{
    var payload = new
    {
        data = "Welcome to this awesome service",
        metadata = {
          toNumber = "+31612345678"
        },
        operation = "create"
    };

    // send message using output-binding
    var content = new StringContent(
        JsonSerializer.Serialize(payload),
        Encoding.UTF8, "application/json");

    var httpClient = clientFactory.CreateClient();

    await httpClient.PostAsync("http://localhost:3500/v1.0/bindings/sms", content);
}
```

To start, note how the previous figure uses a standard `HttpClient` object to execute an HTTP POST operation. There are no references to a specific Dapr library or SDK. Note too the port is the same as used by the Dapr sidecar (in this case, the default HTTP port `3500`). 

<!--
# As you can see in the example, the code uses no Dapr specific library or SDK. Just the standard `HttpClient` to do an HTTP POST. We use the HTTP port that is used by the Dapr sidecar (in this case, the default HTTP port `3500`).
-->

The structure of the payload (that is, message sent) will vary per binding. In the example above, the payload contains a `data` element  with a message. Bindings to other types of external resources can be different, especially in terms of the metadata that is sent. Each payload must also contain an `operation` field, which define the operations that the binding will support. The above example specifies a  `create` operation as in *creating* an SMS message. Common operations include:

- create
- get
- delete
- list

The documentation for each binding describes the available operations and how to invoke them.

## Using the .NET SDK

The Dapr .NET SDK provides direct support for .NET core and ASP.NET Core developers. In the following example, the `DaprClient.InvokeBindingAsync()` method 
streamlines the effort to sends an SMS message to the configured output resource:

```csharp
private async Task SendSMSAsync([FromServices] DaprClient daprClient)
{
    var string message = "Welcome to this awesome service";
    var metadata = new Dictionary<string,string> { { "toNumber", "+31612345678"} };
    await DaprClient.InvokeBindingAsync("sms", "create", message, metadata);
}
```

The method expects a number of arguments including `metadata` and the actual `message`.

When used to invoke a binding, the `DaprClient` uses gRPC to call the Dapr API on the Dapr sidecar.

### Binding Components

Under the hood, resource bindings are implemented with Dapr binding components. Components are contributed by the community and are written in the [Go](https://golang.org/) language. If you need to integrate with an external resource for which no Dapr binding exists yet, you can create it yourself. Check out the [Dapr components-contrib repo](https://github.com/dapr/components-contrib) to see how you can contribute a binding.

You configure bindings using a component configuration file. Here's an example configuration for the Twitter binding:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: twitter-mention
  namespace: default
spec:
  type: bindings.twitter
  metadata:
  - name: consumerKey
    value: "****" # twitter api consumer key, required
  - name: consumerSecret
    value: "****" # twitter api consumer secret, required
  - name: accessToken
    value: "****" # twitter api access token, required
  - name: accessSecret
    value: "****" # twitter api access secret, required
  - name: query
    value: "dapr" # your search query, required
```

Each binding configuration contains a general `metadata` element with a `name` and `namespace` field. Dapr will determine the service endpoint to invoke based upon the configured `name` field. In the above example, Dapr will invoke `/twitter-mention` in your service when an event occurs.

In the `spec` element, you specify the `type` of the binding along with binding specific `metadata`. The example specifies credentials for accessing a Twitter account using its API. The metadata can differ between input and output bindings. For example, to use the Twitter as an input binding, you need to specify the text to search for in tweets using the `query` field. Every time a tweet the query, the Dapr sidecar will invoke the `/twitter-mention` endpoint on the service. It will also deliver the contents of the Tweet

A binding can be configured for input, output, or both. Interestingly, the binding does not explicitly specify input or output configuration. Instead, the direction is inferred by the usage of the binding along with required configuration settings.

The [Dapr documentation for resource bindings](https://docs.dapr.io/operations/components/setup-bindings/supported-bindings/) provides a complete list of the available bindings and their specific configuration settings.

### Cron binding

We call your attention to special binding: The Cron binding. It does not subscribe to events from an external system. Instead, this binding uses a configurable interval schedule to trigger your application at periodic intervals. It provides a simple way to implement a background worker to wake up and do some work on a regular interval, without the need to implement an endless loop with a configurable delay. Here's an example of a Cron binding configuration:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: checkOrderBacklog
  namespace: default
spec:
  type: bindings.cron
  metadata:
  - name: schedule
    value: "@every 30m"
```

In this example, Dapr triggers a service by invoking the `/checkOrderBacklog` endpoint every 30 minutes. There are several patterns available for specifying the `schedule` value. See the [Cron binding documentation](ocs.dapr.io/operations/components/setup-bindings/supported-bindings/cron/) for more information.

## Reference case: eShopOnDapr

The eShopOnDapr reference application a [SendGrid](https://docs.dapr.io/operations/components/setup-bindings/supported-bindings/sendgrid/) binding to send an email to a user when a new order is started. You can find this binding in the `eshop-email.yaml` file in the components folder:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: sendmail
  namespace: default
spec:
  type: bindings.twilio.sendgrid
  metadata:
  - name: apiKey
    secretKeyRef:
      name: sendGridAPIKey
auth:
  secretStore: eshop-secretstore
```

This configuration uses the Twilio SendGrid binding. Note how the API key for connecting to the service consumes a Dapr secret reference. This approach keeps secrets outside of the configuration file. Read the [Dapr secrets section](secrets-buildingblocks.md) in this chapter to learn more about Dapr secrets.

The binding specifies a binding component that can be invoked using the `/sendmail` endpoint on the Dapr sidecar. Here's a code snippet in which an email is sent whenever an order is started:

```csharp
public async Task Handle(OrderStartedDomainEvent notification, CancellationToken cancellationToken)
{
    var payload = new
    {
        metadata = new
        {
            emailFrom = "eShopOn@dapr.io",
            emailTo = notification.UserName,
            subject = $"Your eShopOnDapr order #{notification.Order.Id}",
        },
        data = CreateEmailBody(notification),
        operation = "create"
    };

    var content = new StringContent(
        JsonSerializer.Serialize(payload),
        Encoding.UTF8, "application/json");

    var httpClient = _clientFactory.CreateClient();
    var response = await httpClient.PostAsync($"http://localhost:{_daprHttpPort}/v1.0/bindings/sendmail", content);
}
```

> [!NOTE]
> We determine the HTTP port that the Dapr sidecar uses by retrieving the value of the `DAPR_HTTP_PORT` environment variable and store this in the private `_daprHttpPort` variable.

As you can see in this example, the payload for the SendGrid binding contains a `metadata` part that we use for specifying the email sender, recipient, and the subject for the email message. We could also specify each of these metadata fields in the configuration file for this binding. The `data` field contains the message body. The `CreateEmailBody` method simply formats a string with the body text.

[TODO:input binding]

## Summary

Dapr resource bindings enable you to integrate with different external resources and systems without using taking dependencies on their libraries or SDKs. These external systems do not necessarily have to be messaging systems like a service bus or message broker. Bindings also exist for datastores and web resources like Twitter or SendGrid.

Input bindings (or triggers) react to events occurring in the external system and subsequently invoke the configured corresponding public HTTP endpoint on your application. Dapr uses the name of the binding in the configuration to determine the endpoint to call on your application.

Output bindings will send messages to an external system. You trigger an output binding by doing an HTTP POST on the `/v1.0/bindings/<binding name>` endpoint on the Dapr sidecar. You can also use gRPC to invoke the binding. The .NET SDK offers a `InvokeBindingAsync` method to invoke Dapr bindings using gRPC.

You implement a binding with Dapr component. These components are contributed by the community. Each binding component's configuration has metadata that is specific for the external system it abstracts. Also, the commands it supports and the structure of the payload will differ per binding component.

### References

>[!div class="step-by-step"]
>[Previous](index.md)
>[Next](index.md)



<!--
Serverless computing is a cloud computing execution model in which the cloud provider runs ... Serverless vendors offer compute runtimes, also known as function as a service (FaaS) platforms, which execute application logic but do not store ...
Serverless offerings, such as Azure Functions, AWS Lambda, and Google Cloud Functions, have been widely adopted across organizations. A key feature are bindings which enable you to receive or send events to a variety of resources.  
One of the appeals of the highly popular serverless offerings by the different cloud vendors is its reactive nature. Messages come in from various systems that trigger certain business logic. This business logic will often yield new messages that it subsequently needs to publish using another messaging system.

Dapr supports building applications that use different inputs and outputs with resource bindings.
-->