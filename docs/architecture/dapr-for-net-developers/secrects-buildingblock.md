---
title: The Secrets Management building block
description: A description of the Secrets Management building-block and how to apply it
author: edwinvw
ms.date: 12/01/2020
---

# The Secrets Management building block

In almost every application, you have to deal with information that you should not disclose because of security reasons. Here are some examples:

- a database connection-string that contains a username and password, 
- an API key for calling an external RESTful API or 
- a client certificate for authenticating to some external system. 

Often, you make these values configurable so you can vary them per deployment environment. A popular way of doing this is to store them in an external settings-file that is read by the application at runtime. But how do you make sure you do not disclose the secrets in the settings-file? A common mistake often made is pushing the settings-file with the secrets to a public Git repository. 

Dapr offers a Secrets Management building block that prevents this and significantly simplifies working with secrets.

## What it solves

A popular solution to working with secrets is using a secrets management tool like **Hashicorp Vault** or **Azure Key Vault**. These tools allow you to centrally store the actual secrets securely and reference them from your application. Access to the secrets is secured using some form of authentication and authorization, so only an application with sufficient rights can access them. When running your application in Kubernetes, you can also leverage its built-in secrets management mechanism. 

It's great to see that several secrets management tools exist. But you need to know the differences between them. You are also required to learn all the specific APIs or SDKs they offer to use them from your code. This is where Dapr comes in. 

The Dapr Secrets Management building block offers an abstraction for working with secrets. You can use the abstraction from your code. Hence, you do not need to learn how to interact with the different secrets management tools. This is all handled by Dapr. 

## How it works

You can leverage the Secrets Management building block in two ways:

- Call it from your code to retrieve a secrets.
- Reference a secret from a Dapr component configuration.

### Call the Secrets Management API from code

First we'll address calling the building block from your code. As with the other building blocks, the Dapr sidecar offers an API for you to interact with the Secrets Management building block. You can call this API using HTTP or gRPC: 

```http
http://localhost:<daprPort>/v1.0/secrets/<secret-store-name>/<name>
```

Note the Dapr specific URL segments in the above call:

- `<daprPort>` provides the port number upon which the Dapr sidecar is listening.

- `<secret-store-name>` provides the name of the selected Dapr secrets management component.

- `<name>` provides the name of the secret you want to retrieve.

The response you will receive varies based on the type of secret you are retrieving. If the secret is just a single value, you will receive a JSON response containing the single value. Here's an example:

```http
http://localhost:3500/v1.0/secrets/secrets-store/customerdb
```

```json
{
  "customerdb": "Server=sqlsrv-p-06;Database=CRM;User Id=sql_p_customer;Password=h8%fa51Km9D4;"
}
```

If the secret contains multiple key-value pairs, you receive all the keys and values in a single JSON response like in the following example:

```http
http://localhost:3500/v1.0/secrets/secrets-store/interestPercentages
```

```json
{
  "tier1-percentage": "2.5",
  "tier2-percentage": "3.8",
  "tier3-percentage": "5.1"
}
```

Whether you can store multiple keys in a secret depends on the actual secret store you have configured.

### Reference a secret in a component configuration

The second way ...

### Using the Dapr .NET SDK

### Secrets Management components

### Configuring Secrets Management components

## Reference case: eShopOnDapr

## Summary

### References

>[!div class="step-by-step"]
>[Previous](serviceinvocation.md)
>[Next](bindings-buildingblock.md)