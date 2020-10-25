---
title: The Resource Bindings building block
description: A description of the Resource Bindings building-block and how to apply it
author: edwinvw
ms.date: 10/18/2020
---

# The Resource bindings building block

One of the appeals of the highly popular serverless offerings by the different cloud vendors is its reactive nature. Messages come in from various systems that trigger certain business logic. This business logic will often yield new messages that it subsequently needs to publish using another messaging system.

Dapr supports building applications that use different inputs and outputs with resource bindings.

## What it solves

You might think that the reactive behavior described in the introduction seems similar to the Publish/Subscribe pattern we described in the [Publish/Subscribe section](publish-subscribe-buildingblock.md) in this chapter. Although there are similarities, there are also some significant differences between the two.

Resource bindings offer a way to connect to different inputs and outputs without using any specific library or SDK in your application code. They also support more message sources and targets than the Publish/Subscribe building block (that primarily focuses on message brokers). For instance, an input binding exists for triggering an application when a tweet with a certain term in the text is published on Twitter. Also, an output binding exists for sending an SMS using Twilio. Finally, an application can switch between bindings at runtime without any code changes, which is much harder to do with the Publish/Subscribe building block.

The main focus of Dapr resource bindings is to make developers more productive by removing the hassle of learning specific APIs or SDKs for integrating with all sorts of different messaging systems.

## How it works

As stated, you don't need an SDK or library to use resource bindings. The Dapr sidecar will use input bindings to trigger your application by invoking a public HTTP endpoint that you provide. You trigger output bindings from your application by calling an API on the Dapr sidecar - either using HTTP or gRPC. Let's dive in.

### Input bindings

When you configure an input binding for your application, you basically couple a public HTTP endpoint that your application provides to a particular external trigger. Here is an example of how this works:

![Input binding](media/bindings-input.png)

> In the example, the application handles HTTP calls on port 6000.

1. The Dapr sidecar picks up the binding configuration and subscribes to events that occur in the configured event source. In the example, the event source is Twitter. We will dive into binding configuration in more detail in the *Components* section.
2. When a Tweet that matches a configured query is published on Twitter, the Dapr sidecar consumes it.
3. The Dapr sidecar invokes the configured endpoint on your application. In the example, this is an HTTP POST on the `/tweet` endpoint of the application. The endpoint is equal to the name of the binding specified in the configuration. Because it is an HTTP POST, also the JSON payload of the event is passed in.
4. After handling the trigger, your application returns an HTTP status code `200 OK`.

Here is an example of an ASP.NET Core controller that handles the call triggered by the Twitter binding:

```csharp
[ApiController]
public class SomeController : ControllerBase
{

    ...

    [HttpPost("/tweet")]
    public ActionResult Post(JsonElement data)
    {
        // handle tweet

        ...

        // acknowledge message
        return Ok();
    }
}
```

An alternative scenario is that something goes wrong while your application is handling the trigger. In that case, you can return an HTTP status code other than `200 OK` (for instance `500 Error`). If the binding offers at least once delivery guarantees, the Dapr sidecar will retry the trigger when that happens. Check out [the documentation of the different bindings](https://github.com/dapr/docs/tree/master/concepts/bindings) to see whether they offer at least once or exactly once delivery guarantees.

### Output bindings

When you configure an output binding for your application, you can use it by invoking the bindings API on the Dapr sidecar of your application. Here is an example:

![Output binding](media/bindings-output.png)

1. Your application invokes the `/v1.0/bindings/sms` endpoint on the Dapr sidecar. In this case, it uses an HTTP POST to invoke the API. It is also possible to use gRPC.
2. The Dapr sidecar calls the external messaging system to send the message. The message will contain the payload of the HTTP call.

Here is an example of using the output binding from .NET:

```c#
private async Task SendSMSAsync(IHttpClientFactory clientFactory)
{
    var payload = new
    {
        data = new {
            subject = "Welcome!",
            message = "Welcome to this awesome service"
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

As you can see in the example, the code uses no specific library or SDK. Just a plain `HttpClient` to do an HTTP POST. We use the HTTP port that is used by the Dapr sidecar (in this case, the default HTTP port `3500`).

The structure of the payload that you use will differ per binding. In this case, the payload contains a `data` part with a subject and message. For other bindings, this will be different. For some bindings, there is also a metadata part that contains several fields (as we will see in the eShopOnDapr example later in this section). Each payload must also contain an `operation` field. In the example, we use the `create` operation. Each binding implementer determines which operations the binding supports. Common commands are:

- create
- get
- delete
- list

The documentation of the binding describes the possible operations and how to invoke them.

### Components

Resource bindings are implemented by components. Bindings are contributed by the community and are written in Go. So if you need to integrate with some external messaging system for which no Dapr binding exists yet, you can create one yourself. Check out the [Dapr components-contrib repo](https://github.com/dapr/components-contrib) to see how you can contribute a binding.

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

Each binding configuration contains a general `metadata` part with a `name` and `namespace` field. As stated, Dapr will use the specified name to determine the endpoint to invoke on your application when an event occurs. So in the example configuration, Dapr will invoke `/twitter-mention`.

In the `spec` part, you specify the type of the binding and binding specific `metadata`. In the example, you can see that you need to specify the credentials for accessing a Twitter account using its API. The metadata can differ based on whether you configure an input or an output binding. For using the Twitter binding as an input binding, you need to specify the text to search for in tweets using the `query` field. Every time a tweet that contains text that matches the query is published on Twitter, the Dapr sidecar will invoke the `/twitter-mention` endpoint on your application. It will also deliver the contents of the Tweet

Check out [the documentation of the different bindings](https://github.com/dapr/docs/tree/master/concepts/bindings) to get a complete list of the available bindings and their specific configuration settings.

## Reference case: eShopOnDapr

In eShopOnDapr, we use a *SendGrid* binding to send an email to the user when a new order is started. We have configured this binding in the `eshop-email.yaml` file in the components folder:

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

As you can see in this configuration, we use the Twilio SendGrid binding. We specify the API key for connecting to the service using a Dapr secret reference. This allows us to keep secrets outside of the configuration file. Read the [Dapr secrets section](secrets-buildingblocks.md) in this chapter to learn more about Dapr secrets.

By doing this, we have specified a binding component that we can invoke using the `/sendmail` endpoint on the Dapr sidecar. Here's a code snippet in which we use the binding to send an email when an order is started:

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

> We determine the HTTP port that the Dapr sidecar uses by retrieving the value of the `DAPR_HTTP_PORT` environment variable and store this in the private `_daprHttpPort` variable.

As you can see in this example, the payload for the SendGrid binding contains a `metadata` part that we use for specifying the email sender, recipient and the subject for the email message. We could also specify each of these metadata fields in the configuration file for this binding. The `data` field contains the message body. The `CreateEmailBody` method simply formats a string with the body text.

[TODO:input binding]

## Summary

In this section ...

### References

>[!div class="step-by-step"]
>[Previous](index.md)
>[Next](index.md)