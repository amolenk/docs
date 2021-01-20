---
title: The Dapr secrets management building block
description: What exactly is the Dapr secrets management building block and how can I apply it to my applications
author: edwinvw
ms.date: 1/19/2020
---

# The Dapr secrets management building block

Enterprise applications require secrets in order to operate. Common examples include:

- A database connection string that contains a username and password.
- An API key for calling an external RESTful API.
- A client certificate for authenticating to an external system.

It's critical to never disclose these values outside of the application.

Not long ago, it was popular to store application secrets in a configuration file that was part of the application. .NET Developers may fondly recall the web.config file in the .NET Framework. While simple to implement, adding secrets to a configuration file was far from secure. A common misstep was to include the configuration file when pushing to a public GIT repository, exposing the secrets to the world! To address this shortcoming, the .NET Core platform included a [Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets?view=aspnetcore-5.0&tabs=windows#secret-manager) feature that stores sensitive data in a physical folder outside of the project tree. While secrets are outside of source control, this feature doesn't encrypt data and is designed for **development purposes** only. 

Presume secrets stored within a codebase to be an anti-pattern. A violation the [Twelve-Factor application](https://12factor.net/) methodology that states that configuration and secrets should be externalized outside of the code. A more modern and secure practice is to isolate secrets in a secrets management tool like **Hashicorp Vault** or **Azure Key Vault**.  These tools enable you to externally store secrets securely, vary credentials across deployments, and reference them from your application code. However, each tool has its costs: Complexity and a learning curve.

Dapr offers a building block that significantly simplifies working with secrets.





  You should consider all of these things to be anti-patterns for the
cloud. All of these situations prevent you from varying configuration
across environments while still maintaining your immutable
release artifact.


violated the [Twelve-Factor application methodology](https://12factor.net/) which states that configuration and secrets should be externalized though a tool outside of the code. stored and

Not long ago, it was popular to store application secrets in a configuration file that was part of the application. While simple to implement, the approach violated the [Twelve-Factor application methodology](https://12factor.net/) which states that configuration and secrets should be externalized though a tool outside of the code. stored and was far from secure. A common mistake was to include this configuration file when pushing to a public GIT repository, exposing the secrets to the world! To address this shortcoming, the .NET Core platform included a [Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets?view=aspnetcore-5.0&tabs=windows#secret-manager) feature which stores sensitive data in a physical folder outside of the project tree. While secrets are not checked into source control, this feature doesn't encrypt data and is designed for **development purposes** only. 

A more modern and secure practice is to store secrets in a secrets management tool like like **Hashicorp Vault** or **Azure Key Vault**.  These tools allow you to externally store secrets securely and reference them from your application. However, each tool has its differences, complexities, and learning curve. 

Dapr offers a building block that simplifies managing secrets.

## What it solves

The Dapr [Secrets Management building block](https://docs.dapr.io/developing-applications/building-blocks/secrets/secrets-overview/) abstracts away the complexity of working with secrets. 

 - It hides the underlying plumbing through a unified interface.
 - It supports a variety of secret stores, both locally and in production. 
 - Applications don't require direct dependencies on secret store libraries. 
 - Developers don't require detailed knowledge of each secret store.
 
All of the above concerns are handled by Dapr. 

Access to the secrets is secured through authentication and authorization. Only an application with sufficient rights can access secrets. Applications running in Kubernetes can also leverage its built-in secrets management mechanism.

## How it works

There are two ways an application can leverage the secrets management building block:

- Retrieve a secret from a direct call.
- Reference a secret indirectly from a Dapr component configuration.

Retrieving secrets is covered first. Referencing a secret from a Dapr component configuration is addressed later.

As with all Dapr building blocks, applications interact with a Dapr sidecar service. It exposes the secrets management API. The API can be called with either HTTP or gRPC using the following URL:

```http
http://localhost:<daprPort>/v1.0/secrets/<secret-store-name>/<name>?<metadata>
```

The URL contains the following segments:

- `<daprPort>` specifies the port number upon which the Dapr sidecar is listening.
- `<secret-store-name>` specifies the name of the Dapr secrets management component.
- `<name>` specifies  the name of the secret to retrieve.
- `<metadata>` provides additional information for the secret. This segment is optional. Metadata properties will differ per secret store. See the [secrets management API reference]([Secrets API reference | Dapr Docs](https://docs.dapr.io/reference/api/secrets_api/)) for more information about the available metadata properties.

 > [!NOTE]
 > The above URL represents the native Dapr API call available to any development platform supports HTTP or gRPC. Popular platforms like .NET, Java, and Go have their own custom APIs.

The JSON response contains the key and value of the secret.

Figure 10-1 shows how Dapr handles a request for the secrets management API:

![Diagram of retrieving a secret using the Dapr secrets management API.](media/secrets-management/retrieve-secret.png)

**Figure 10-1**. Retrieving a secret with the Dapr secrets management API

1. The service calls the Dapr secrets management API, along with the name of the secret store and secret to retrieve.
1. The Dapr sidecar retrieves the specified secret from the secrets store.
1. The Dapr sidecar returns the secret information back to the service.

Some secret stores support storing multiple key/value pairs in a single secret. For those scenarios, the response would contain multiple key/value pairs in a single JSON response as in the following example:

```http
GET http://localhost:3500/v1.0/secrets/secrets-store/interestRates?metadata.version_id=3
```

```json
{
  "tier1-percentage": "2.5",
  "tier2-percentage": "3.8",
  "tier3-percentage": "5.1"
}
```

## Using the Dapr .NET SDK

The Dapr .NET SDK streamlines Dapr secret management for .NET developers. Consider the `DaprClient.GetSecretAsync` method. It vastly simplifies retrieving secrets. Here is an example of retrieving a connection string secret for a SQL Server database:

```csharp
Dictionary<string, string> metadata = new Dictionary<string, string> { { "version_id", "3" } };
Dictionary<string,string> secrets = await daprClient.GetSecretAsync("secrets-store", "eshopsecrets", metadata);
string connectionString = secrets["customerdb"];
```

Arguments for the `GetSecretAsync` method include:

 - The name of the Dapr secret store component ('secrets-store')
 - The secret you wish to retrieve ('eshopsecrets')
 - Optional metadata key/value pairs ('version_id=3')

The method responds with a dictionary object as some secrets contain multiple key/value pairs. In the example above, the secret named `customerdb` is referenced from the collection to get return a connection string.

The Dapr .NET SDK also features a .NET configuration provider. It loads specified secrets into the .NET Core configuration API which makes a list of key/value pairs available to the running application. 

The secrets configuration provider is available from the `Dapr.Extensions.Configuration` NuGet package. The provider can be registered in the `Program.cs` of an ASP.NET Web API application:  

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration(config =>
        {
            var daprClient = new DaprClientBuilder().Build();
            var secretDescriptors = new List<DaprSecretDescriptor>
            {
                new DaprSecretDescriptor("eshopsecrets")
            };
            config.AddDaprSecretStore("secrets-store", secretDescriptors, daprClient);
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

The example loads the `eshopsecrets` secrets collection into the .NET configuration system upon startup. Registering the provider requires an instance of `DaprClient` to invoke the secrets API in the Dapr sidecar. Also required are  the name of the secrets management component and name of the secret as as argument to the `DaprSecretDescriptor` object.

At this point, you can retrieve secrets directly from application code:

```csharp
public void GetCustomer(IConfiguration config)
{
    var connectionString = config["eshopsecrets"]["customerdb"];
}
```

## Secrets management components

The secrets management building block supports several Dapr secret store components. At the time of writing, the following secret stores are available:

- Environment Variables
- Local file
- Kubernetes secrets
- AWS Secrets Manager
- Azure Key Vault
- GCP Secret Manager
- HashiCorp Vault

 > [!IMPORTANT]
 > The environment variables and local file components are only meant for development and not production workloads.

The following section shows how to configure a secrets management component.

### Configuration (TODO Move generic text to getting started)

You configure a Secrets Management component using a Dapr component configuration file. Here you see the structure of a config file:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: [component name]
  namespace: [namespace]
spec:
  type: secretstores.[secret store type]
  version: v1
  metadata:
  - name: [property name]
    value: [property value]
```

As with all the Dapr configuration files, you specify a `name` and an optional `namespace` for the component. You specify the actual secret store using the `type` field of the `metadata` section. The properties you specify in the `metadata` section differ per secret store. Once you have configured the secrets management component, you can use the Dapr Secrets Management API to retrieve secrets from your application code.

As stated before, another way of using secrets is by referencing them from other component configuration files. Here's an example using a state management component (see the [State Management building block chapter](state-management.md) for more information). It uses the Redis cache state component. The configuration for this component might look like this:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: eshop-basket-statestore
  namespace: eshop
spec:
  type: state.redis
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: e$h0p0nD@pr
```

The above configuration file contains a clear-text password for connecting to the Redis server. This is not a secure way to protect password. If you were to push this config file to a public Github repo for instance, you have disclosed the password. Storing the password in a secret store would make this more secure. The following examples demonstrate using several different secret stores.

### Local file

The local file component stores secrets in a JSON file on the local filesystem. As stated, this is only suitable for development scenarios. The  JSON file used in this example is named `eshop-secrets.json` and it contains a single secret containing the Redis password:

```json
{
  "eShopRedisPassword": "e$h0p0nD@pr"
}
```

You place this file in the Dapr components folder that you specify when running the application with Dapr.

The Secrets Management component configuration file uses the JSON file as a secret store. It is also placed in the components folder:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: eshop-local-secret-store
  namespace: eshop
spec:
  type: secretstores.local.file
  version: v1
  metadata:
  - name: secretsFile
    value: ./components/eshop-secrets.json
  - name: nestedSeparator
    value: ":"
```

The component type is `secretstore.local.file`. The `secretsFile` metadata element specifies the path to the secrets file.

> [!IMPORTANT]
> You can specify the path to the secrets file as an absolute path or a relative path. When using a relative path, you must specify it relative to the folder from where you will start the application. In this example, the `components` folder is a sub-folder of the folder that contains a .NET application.

When starting the application in Dapr stand-alone mode, you do this from the application folder and specify the components path on the command-line:

```bash
dapr run --app-id basket-api --components-path ./components dotnet run
```

This only works like this when running Dapr in stand-alone mode. For other hosting options, you should use a different mechanism to give Dapr access to the secrets file (e.g., volume mounts when hosting in Kubernetes).

The `nestedSeparator` in the config file specifies the character that Dapr will use to flatten the JSON hierarchy in the secrets file (if present). Imagine using the following JSON secrets file:

```json
{
    "redisPassword": "some password",
    "connectionStrings": {
        "customerdb": "some connection string",
        "productdb": "some connection string"
    }
}
```

With a colon as separator, you can then get at the value of the customerdb connection-string using the key `connectionStrings:customerdb`. The colon is the default separator (also when nothing is specified).

Now the secrets management component is configured, the State Management configuration file can reference this secret when specifying the password for connecting to the Redis server:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: eshop-basket-statestore
  namespace: eshop
spec:
  type: state.redis
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    secretKeyRef:
      name: eShopRedisPassword
      key: eShopRedisPassword
auth:
  secretStore: eshop-local-secret-store
```

You use the `secretKeyRef` to reference the secret containing the password. The literal clear-text value for the password is now gone. You can reference the secret using its name and the name of the key of the secret value, in this case both `eShopRedisPassword`.  You specify the secrets management component to get the secret from in the `auth` metadata element, in this case `eshop-local-secret-store`.

You might wonder why `eShopRedisPassword` is identical for both the name and the key in the secret reference. This is because the secrets are not identified by a separate name when using the local file secret store. This will be different in the next example, which uses Kubernetes secrets.

### Kubernetes secret

This second example focuses on a Dapr application running in Kubernetes. It leverages the standard secrets mechanism that Kubernetes offers. Use the Kubernetes CLI (`kubectl`) to create a secret named `eshop-redis-secret` that contains the password:

```bash
kubectl create secret generic eshopsecrets --from-literal=redisPassword=e$h0p0nD@pr
```

Now the secret is created, you can reference it in the State Management configuration file:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: eshop-basket-statestore
  namespace: eshop
spec:
  type: state.redis
  metadata:
  - name: redisHost
    value: redis:6379
  - name: redisPassword
    secretKeyRef:
      name: eshopsecrets
      key: redisPassword
auth:
  secretStore: kubernetes
```

The `secretKeyRef` specifies the name of the Kubernetes secret and the secret's key to use, in this case: `eshopsecrets` and `redisPassword` respectively. The `auth` metadata section instructs Dapr to use the Kubernetes secrets management component. This is the default by the way, so you can omit this part when using Kubernetes secrets.

In a production setting, you would never create the secret, as shown in this example. Typically, you create secrets as part of some automated CI/CD pipeline. This ensures that only people with sufficient access to this pipeline can access and change the secrets. Developers creating Dapr configuration files for deploying an application can then reference the secrets without knowing the actual values.

### Azure Key Vault

The next example will be a bit more elaborate and more geared toward a production scenario. It uses **Azure Key Vault** as the secret store. Azure Key Vault is an Azure service that allows you to store secrets securely in the cloud.

For this example to work, the following prerequisites must be satisfied:

- You have administrative access to an Azure subscription.
- An Azure Key Vault named `eshopkv` already exists that holds a secret named `redisPassword` containing the password for connecting to the Redis server.
- A service principal has been created. A service principal is an identity that can be used by applications to authenticate against certain Azure services. The service principal uses an X509 certificate. The application uses this certificate as a credential to authenticate itself.
- The X509 certificate for this service principal (containing both the public and private key) has been downloaded and is available as a `.pfx` file on the local filesystem.

Check out the [Dapr Azure Key Vault secret store documentation](https://docs.dapr.io/operations/components/setup-secret-store/supported-secret-stores/azure-keyvault/) for instructions on how to set this up.

To use the Azure Key Vault as the secret store for eShop, you must create a component configuration file.

#### Running in stand-alone mode

When running the application with Dapr in stand-alone mode, the configuration file looks like this:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: eshop-azurekv-secret-store
  namespace: eshop
spec:
  type: secretstores.azure.keyvault
  version: v1
  metadata:
  - name: vaultName
    value: eshopkv
  - name: spnTenantId
    value: "619926af-a7c3-4e95-93ed-4ecc4e3e652b"
  - name: spnClientId
    value: "6cf48032-6c38-43be-9d6f-2a43ce736b09"
  - name: spnCertificateFile
    value : "azurekv-spn-cert.pfx"
```

The type of the secrets management component is `secretstores.azure.keyvault`. The `metadata` element to configure access to the Azure Key Vault contains the following properties:

- The `vaultName` contains the name of the Azure Key Vault.
- The `spnTenantId` contains the *tenant id* of the service principal used to authenticate against the Key Vault.
- The `spnClientId` contains the *app id* of the service principal used to authenticate against the Key Vault.
- The `spnCertificateFile` contains the path to the certificate file for the service principal used to authenticate against the Key Vault.

> [!NOTE]
> You can retrieve all the necessary information of the service principal using the Azure CLI or using the Azure portal.

Now the application can retrieve the Redis password from the Azure Key Vault.

#### Running on Kubernetes

When running your application with Dapr on Kubernetes, you can also use the service principal to authenticate against the Azure Key Vault. First, you create a Kubernetes secret that contains the certificate file using the Kubernetes CLI tool:

```bash
kubectl create secret generic [k8s_spn_secret_name] --from-file=[pfx_certificate_file_local_path]
```

You have to specify the following command-line arguments:

- `[k8s_spn_secret_name]` is secret name in Kubernetes secret store.
- `[pfx_certificate_file_local_path]` is the path of X509 certificate file you downloaded above.

Once the Kubernetes secret is created, you can reference this in the secrets management component configuration file:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: eshop-azurekv-secret-store
  namespace: eshop
spec:
  type: secretstores.azure.keyvault
  version: v1
  metadata:
  - name: vaultName
    value: [your_keyvault_name]
  - name: spnTenantId
    value: "619926af-a7c3-4e95-93ed-4ecc4e3e652b"
  - name: spnClientId
    value: "6cf48032-6c38-43be-9d6f-2a43ce736b09"
  - name: spnCertificate
    secretKeyRef:
      name: [k8s_spn_secret_name]
      key: [pfx_certificate_file_local_name]
auth:
    secretStore: kubernetes
```

Now the eShop services running in Kubernetes can retrieve the Redis password from the Azure Key Vault.

> [!IMPORTANT]
> It is crucial to keep the file containing the public and private key-pair for the X509 certificate of the service principal in a safe place. One way of doing this, is to make sure you place the file in a well-known folder outside the source-code repository. The configuration file can then reference the file in this well-known folder. On your local dev machine, you place the certificate in this folder. An automated deployment pipeline places the certificate in this folder on the machine where you deploy the application. Also, it is a best practice to use a different service principal per environment. This ensures that the service principal used by the developers in the DEV environment cannot be used to access secrets for the PRODUCTION environment.

When running in AKS, it's also possible to use an Azure managed identity for authenticating against Azure Key Vault. This is outside of the scope of this book. See [Azure KeyVault with managed identities](https://docs.dapr.io/operations/components/setup-secret-store/supported-secret-stores/azure-keyvault-managed-identity/) in the Dapr documentation for instructions on how to set this up.

### Scoping secrets

Secret scopes allow you to control which secrets your application can access. You configure scopes in the Dapr sidecar configuration you can specify when running your application with Dapr. See [Dapr sidecar configuration documentation](https://v1-rc2.docs.dapr.io/operations/configuration/configuration-overview/) for instructions on how to specify a configuration file when starting an application.

Here's an example of a Dapr sidecar configuration file that contains secret scopes:

```yml
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: dapr-config
  namespace: eshop
spec:
  tracing:
    samplingRate: "1"
  secrets:
    scopes:
      - storeName: eshop-azurekv-secret-store
        defaultAccess: allow
        deniedSecrets: ["redisPassword", "apiKey"]
```

You specify the scopes per secret store (in the example, this is `eshop-azurekv-secret-store`). You configure access to secrets using the following properties:

| Property         | Value               | Description                                                                                                                       |
|------------------|---------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| `defaultAccess`  | `allow` or `deny`   | Allows or denies access to all secrets in the specified secret store. This property is optional end the default value is `allow`. |
| `allowedSecrets` | List of secret keys | Secrets specified in the array will be accessible. This property is optional.                                                     |
| `deniedSecrets`  | List of secret keys | Secrets specified in the array will NOT be accessible. This property is optional.                                                 |

The `allowedSecrets` and `deniedSecrets` properties take precedence over the `defaultAccess` property. Imagine you specify `defaultAccess: allowed` and specify an `allowedSecrets` list. In that case, only the secrets in the `allowedSecrets` list are accessible by the application.

## Reference architecture: eShopOnDapr

eShopOnDapr uses the secrets management building block for two secrets:

- the password for connecting to the Redis cache and
- the API-key for using the Twilio Sendgrid API. The application uses Twillio to send emails using a Dapr output binding (as described in the [Bindings building block chapter](bindings-buildingblock.md)).

When running the application using Docker Compose, the **local file** secrets management component is used. In the `dapr/components` folder of the eShopOnDapr repository, you find configuration file `eshop-secretstore.yaml`. It configures the local file secrets store component:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: eshop-secretstore
  namespace: default
spec:
  type: secretstores.local.file
  metadata:
  - name: secretsFile
    value: ./components/eshop-secretstore.json
```

This configuration references the local file `eshop-secretstore.json` in the same folder:

```json
{
    "redisPassword": "**********",
    "sendgridAPIKey": "**********"
}
```

The components folder is mounted as a local folder inside the Dapr sidecar container and specified in the command-line when starting it. Here you see a snippet from the `docker-compose.override.yml` file in the root of the repository:

```yaml
  ordering-backgroundtasks-dapr:
    command: ["./daprd",
      "-app-id", "ordering-backgroundtasks",
      "-app-port", "80",
      "-dapr-grpc-port", "50004",
      "-components-path", "/components",
      "-config", "/configuration/eshop-config.yaml"
      ]
    volumes:
      - "./dapr/components/:/components"
      - "./dapr/configuration/:/configuration"
```

 > [!Note]
 > The Docker Compose override file contains environmental specific configuration values.

Notice the `/components`  volume mount and the `--components-path` command-line argument passed in the `daprd` startup command.

Other component configuration files can reference the secrets. Here's an example of the configuration for the Publish/Subscribe component used in eShopOnDapr:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
  namespace: default
spec:
  type: pubsub.redis
  metadata:
  - name: redisHost
    value: redis:6379
  - name: redisPassword
    secretKeyRef:
      name: redisPassword
auth:
  secretStore: eshop-secretstore
```


Note how the example above references secrets from the local Redis store.

## Summary

The Dapr Secrets Management building block provides the capability of storing and retrieving sensitive configuration settings like passwords and connection-strings. This prevents you from accidentally disclosing these by pushing them to a public repo, for instance.

The building block supports several different secret stores and hides their complexity by making them available through the Dapr secrets management API.

The `DaprClient` in the Dapr .NET SDK provides a method to retrieve secrets. There is also a .NET configuration provider that retrieves secrets upon application startup using the secrets management API. It makes the secrets available as configuration values you can use in your .NET code from the .NET Core configuration system.

You can retrieve secrets in your application code or reference them from other Dapr component configuration files.

You can use secret scopes to control access to specific secrets.

>[!div class="step-by-step"]
>[Previous](observability.md)
>[Next](actors.md)
