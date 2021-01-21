---
title: The Dapr secrets management building block
description: What exactly is the Dapr secrets management building block and how can I apply it to my applications
author: edwinvw
ms.date: 1/19/2020
---

# The Dapr secrets management building block

Enterprise applications require secrets to operate. Common examples include:

- A database connection string that contains a username and password.
- An API key for calling an external RESTful API.
- A client certificate for authenticating to an external system.

It's critical to manage secrets so that they're never disclosed outside of the application.

Not long ago, it was popular to store application secrets in a configuration file that was part of the application codebase. .NET Developers may fondly recall the web.config file in the .NET Framework. While simple to implement, adding secrets to the app configuration file was far from secure. A common misstep was to include the configuration file when pushing to a public GIT repository, exposing the secrets to the world! 


 > This is point to add 12e factor

Presume secrets stored within a codebase to be an anti-pattern. A violation the [Twelve-Factor application](https://12factor.net/) methodology that states that configuration and secrets should be externalized outside of the code.

To address this shortcoming, the .NET Core platform included a [Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets?view=aspnetcore-5.0&tabs=windows#secret-manager) feature that stores sensitive data in a physical folder outside of the project tree. While secrets are outside of source control, this feature doesn't encrypt data and is designed for **development purposes** only. 

A more modern and secure practice is to isolate secrets in a secrets management tool like **Hashicorp Vault** or **Azure Key Vault**.  These tools enable you to externally store secrets securely, vary credentials across deployments, and reference them from your application code. However, each tool has its costs: Complexity and a learning curve.

Dapr offers a building block that significantly simplifies working with secrets.






  You should consider all of these things to be anti-patterns for the
cloud. All of these situations prevent you from varying configuration
across environments while still maintaining your immutable
release artifact.


violated the [Twelve-Factor application methodology](https://12factor.net/) which states that configuration and secrets should be externalized though a tool outside of the code. stored and

Not long ago, it was popular to store application secrets in a configuration file that was part of the application. While simple to implement, the approach violated the [Twelve-Factor application methodology](https://12factor.net/) which states that configuration and secrets should be externalized though a tool outside of the code. stored and was far from secure. A common mistake was to include this configuration file when pushing to a public GIT repository, exposing the secrets to the world! To address this shortcoming, the .NET Core platform included a [Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets?view=aspnetcore-5.0&tabs=windows#secret-manager) feature that stores sensitive data in a physical folder outside of the project tree. While secrets aren't checked into source control, this feature doesn't encrypt data and is designed for **development purposes** only. 

A more modern and secure practice is to store secrets in a secrets management tool like **Hashicorp Vault** or **Azure Key Vault**.  These tools allow you to externally store secrets securely and reference them from your application. However, each tool has its differences, complexities, and learning curve. 

Dapr offers a building block that simplifies managing secrets.

## What it solves

The Dapr [Secrets Management building block](https://docs.dapr.io/developing-applications/building-blocks/secrets/secrets-overview/) abstracts away the complexity of working with secrets. 

 - It hides the underlying plumbing through a unified interface.
 - It supports various *pluggable* secret stores components, which can vary between development and production. 
 - Applications don't require direct dependencies on secret store libraries. 
 - Developers don't require detailed knowledge of each secret store.
 
Dapr handles all of the above concerns. 

Access to the secrets is secured through authentication and authorization. Only an application with sufficient rights can access secrets. Applications running in Kubernetes can also use its built-in secrets management mechanism.

## How it works

Applications can use the secrets management building block in two ways:

- Retrieve a secret directly from the application block.
- Reference a secret indirectly from a Dapr component configuration.

Retrieving secrets is covered first. Referencing a secret from a Dapr component configuration is addressed in a later section.

As with all Dapr building blocks, applications interact with a Dapr sidecar service. It exposes the secrets management API. The API can be called with either HTTP or gRPC using the following URL:

```http
http://localhost:<daprPort>/v1.0/secrets/<secret-store-name>/<name>?<metadata>
```

The URL contains the following segments:

- `<daprPort>` specifies the port number upon which the Dapr sidecar is listening.
- `<secret-store-name>` specifies the name of the Dapr secrets management component.
- `<name>` specifies  the name of the secret to retrieve.
- `<metadata>` provides additional information for the secret. This segment is optional and metadata properties will differ per secret store. For more information about metadata properties, see the [secrets management API reference]([Secrets API reference | Dapr Docs](https://docs.dapr.io/reference/api/secrets_api/)).

 > [!NOTE]
 > The above URL represents the native Dapr API call available to any development platform that supports HTTP or gRPC. Popular platforms like .NET, Java, and Go have their own custom APIs.

The JSON response contains the key and value of the secret.

Figure 10-1 shows how Dapr handles a request for the secrets management API:

![Diagram of retrieving a secret using the Dapr secrets management API.](media/secrets-management/retrieve-secret.png)

**Figure 10-1**. Retrieving a secret with the Dapr secrets management API

1. The service calls the Dapr secrets management API, along with the name of the secret store, and secret to retrieve.
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

For .NET developers, the Dapr .NET SDK streamlines Dapr secret management. Consider the `DaprClient.GetSecretAsync` method. It enables you to retrieve a secret directly from any Dapr secret store with minimal effort. Here's an example of fetching a connection string secret for a SQL Server database:

```csharp
Dictionary<string, string> metadata = new Dictionary<string, string> { { "version_id", "3" } };
Dictionary<string,string> secrets = await daprClient.GetSecretAsync("secrets-store", "eshopsecrets", metadata);
string connectionString = secrets["customerdb"];
```

Arguments for the `GetSecretAsync` method include:

 - The name of the Dapr secret store component ('secrets-store')
 - The secret you wish to retrieve ('eshopsecrets')
 - Optional metadata key/value pairs ('version_id=3')

The method responds with a dictionary object as a secret can contain multiple key/value pairs. In the example above, the secret named `customerdb` is referenced from the collection to return a connection string.

The Dapr .NET SDK also features a .NET configuration provider. It loads specified secrets into the underlying [.NET Core configuration API](https://docs.microsoft.com/aspnet/core/fundamentals/configuration/?view=aspnetcore-5.0). The running application can then reference secrets from the IConfiguration dictionary from the service container or using dependency injection. 

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

The above example loads the `eshopsecrets` secrets collection into the .NET configuration system at startup. Registering the provider requires an instance of `DaprClient` to invoke the secrets API from the Dapr sidecar. The other arguments include the name of the secrets management component and a DaprSecretDescriptor object with the name of the secret.

Once loaded, you can retrieve secrets directly from application code:

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
 > The environment variables and local file components are designed for development workloads only.

The following section shows how to configure a secrets management component.

### Configuration (TODO Move generic text to getting started)

You configure a secrets management component using a Dapr component configuration file. The typical structure of the file is shown below:

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

All Dapr component configuration files require a `name` along with an optional `namespace` value. You specify the type of secret store component using the `type` field in the `spec` section. The properties in the `metadata` section differ per secret store. 

### Indirectly consuming Dapr secrets

As mentioned earlier in this chapter, applications can also consume secrets by referencing them in component configuration files. Consider a [state management component](state-management.md) that uses Redis cache for storing state:

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

The above configuration file contains a **clear-text** password for connecting to the Redis server. **Hardcoded** passwords are always a bad idea. Pushing this configuration file to a public repository would expose your password. Storing the password in a secret store would dramatically improve this scenario. 

The following examples demonstrate using several different secret stores.

### Local file

The local file component is designed for development scenarios. It stores secrets on the local filesystem inside a JSON file. Here's an example named `eshop-secrets.json`. It contains a single secret - a password for Redis:

```json
{
  "eShopRedisPassword": "e$h0p0nD@pr"
}
```

You place this file in a `components` folder that you specify when running the Dapr application.

The following secrets management component configuration file consumes the JSON file as a secret store. It's also placed in the `components` folder:

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
> You can specify the path to a secrets file as an absolute or a relative path. Specify a relative path based on the folder from where the application starts. In the example, the `components` folder is a sub-folder of the directory that contains the .NET application.

From the application folder, start the Dapr application specifying the `components` path as a command-line argument:

```bash
dapr run --app-id basket-api --components-path ./components dotnet run
```

 > [!NOTE]
 > This above example applies to running Dapr in stand-alone mode. For Kubernetes hosting, consider using volume mounts.

The `nestedSeparator` in a Dapr configuration file specifies a character for Dapr to *flatten* any JSON hierarchy. Consider the following snippet:

```json
{
    "redisPassword": "some password",
    "connectionStrings": {
        "customerdb": "some connection string",
        "productdb": "some connection string"
    }
}
```

Using a colon as a separator, you can retrieve the `customerdb` connection-string using the key `connectionStrings:customerdb`. 

 > [!NOTE]
 > The colon is the default separator value.

In the next example, a state management configuration file references the local secret store component to obtain the password for connecting to the Redis server:

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

You use the `secretKeyRef` element to reference the secret containing the password. Doing so replaces the earlier *clear-text* value. You reference the secret using its name and the name of the key of the secret value: `eShopRedisPassword`.  You specify the name of the secret management component `eshop-local-secret-store` in the  `auth` metadata element.

You might wonder why `eShopRedisPassword` is identical for both the name and key in the secret reference? In the local file secret store, secrets aren't identified with a separate name. The scenario will be different in the next example using Kubernetes secrets.

### Kubernetes secret

This second example focuses on a Dapr application running in Kubernetes. It uses the standard secrets mechanism that Kubernetes offers. Use the Kubernetes CLI (`kubectl`) to create a secret named `eshop-redis-secret` that contains the password:

```bash
kubectl create secret generic eshopsecrets --from-literal=redisPassword=e$h0p0nD@pr
```

Once created, you can reference the secret in the state management component configuration file:

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

The `secretKeyRef` element specifies the name of the Kubernetes secret and the secret's key, `eshopsecrets`, and `redisPassword` respectively. The `auth` metadata section instructs Dapr to use the Kubernetes secrets management component. 

 > [!NOTE]
 > Auth is the default value when using Kubernetes secrets.

In a production setting, secrets are typically created as part of an automated CI/CD pipeline. Doing so ensures only people with sufficient permissions can access and change the secrets. Developers create configuration files without knowing the actual value of the secrets.

### Azure Key Vault

The next example is geared toward a real-world production scenario. It uses **Azure Key Vault** as the secret store. Azure Key Vault is an Azure service that enables you to store secrets securely in the cloud.

For this example to work, the following prerequisites must be satisfied:

- You've secured administrative access to an Azure subscription.
- You've provisioned an Azure Key Vault named `eshopkv` that holds a secret named `redisPassword` containing the password for connecting to the Redis server.
- You've created [service principal](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal) in Azure Active Directory.
- You've installed an X509 certificate for this service principal (containing both the public and private key) on the local filesystem.

 > [!NOTE]
 > A service principal is an identity that can be used by applications to authenticate against certain Azure services. The service principal uses an X509 certificate. The application uses this certificate as a credential to authenticate itself.

The [Dapr Azure Key Vault secret store documentation](https://docs.dapr.io/operations/components/setup-secret-store/supported-secret-stores/azure-keyvault/) provides step-by-step instructions to create and configure the KeyVault environment.

#### Running KeyVault in stand-alone mode

Consuming Azure KeyVault in Dapr stand-alone mode requires the following configuration file:

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

The secrets management component type is `secretstores.azure.keyvault`. The `metadata` element to configure access to Key Vault requires the following properties:

- The `vaultName` contains the name of the Azure Key Vault.
- The `spnTenantId` contains the *tenant id* of the service principal used to authenticate against the Key Vault.
- The `spnClientId` contains the *app id* of the service principal used to authenticate against the Key Vault.
- The `spnCertificateFile` contains the path to the certificate file for the service principal to authenticate against the Key Vault.

> [!NOTE]
> You can copy the service principal information from the Azure portal or Azure CLI .

Now the application can retrieve the Redis password from the Azure Key Vault.

#### Running on Kubernetes

Consuming Azure KeyVault with Dapr and Kubernetes also requires a service principal to authenticate against the Azure Key Vault. 

First, create a *Kubernetes secret* that contains a certificate file using the kubectl CLI tool:

```bash
kubectl create secret generic [k8s_spn_secret_name] --from-file=[pfx_certificate_file_local_path]
```

You'll need to include two command-line arguments:

- `[k8s_spn_secret_name]` is the secret name in Kubernetes secret store.
- `[pfx_certificate_file_local_path]` is the path of X509 certificate file.

Once created, you can reference the Kubernetes secret in the secrets management component configuration file:

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

At this point, an application running in Kubernetes can retrieve the Redis password from the Azure Key Vault.

> [!IMPORTANT]
> It's critical to keep the X509 certificate file for the service principal in a safe place. It's best to place the file in a well-known folder outside the source-code repository. The configuration file can then reference the certificate file from this well-known folder. On a local development machine, you're responsible for copying the certificate to the folder. For automated deployments, the pipeline will copy the certificate on the machine to which you deploy the application. It's also a best practice to use a different service principal per environment. Doing so prevents the service principal in the DEVELOPMENT environment to access secrets in the PRODUCTION environment.

When running in AKS, it's preferable to use an [Azure managed identity](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) for authenticating against Azure Key Vault. Managed identities are outside of the scope of this book. See [Azure KeyVault with managed identities](https://docs.dapr.io/operations/components/setup-secret-store/supported-secret-stores/azure-keyvault-managed-identity/) from the Dapr documentation for step-by-step instructions.

### Scoping secrets

Secret scopes allow you to control which secrets your application can access. You configure scopes in a Dapr sidecar configuration file. [Dapr sidecar configuration documentation](https://v1-rc2.docs.dapr.io/operations/configuration/configuration-overview/) provides step-by-step instructions for scoping secrets.

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

You specify scopes per secret store. In the above example, the secret store is named `eshop-azurekv-secret-store`. You configure access to secrets using the following properties:

| Property         | Value               | Description                                                                                                                       |
|------------------|---------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| `defaultAccess`  | `allow` or `deny`   | Allows or denies access to all secrets in the specified secret store. This property is optional with a default value of `allow`. |
| `allowedSecrets` | List of secret keys | Secrets specified in the array will be accessible. This property is optional.                                                     |
| `deniedSecrets`  | List of secret keys | Secrets specified in the array will NOT be accessible. This property is optional.                                                 |

The `allowedSecrets` and `deniedSecrets` properties take precedence over the `defaultAccess` property. Imagine specifying `defaultAccess: allowed` and an `allowedSecrets` list. In this case, only the secrets in the `allowedSecrets` list would be accessible by the application.

## Reference architecture: eShopOnDapr

The eShopOnDapr reference application uses the secrets management building block for two secrets:

- the password for connecting to the Redis cache and
- the API-key for using the Twilio Sendgrid API. The application uses Twillio to send emails using a Dapr output binding (as described in the [Bindings building block chapter](bindings-buildingblock.md)).

When running the application using Docker Compose, the **local file** secrets management component is used. The component configuration file `eshop-secretstore.yaml` is found in the `dapr/components` folder of the eShopOnDapr repository. It configures the local file secrets store component:

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

The configuration file references the local store file `eshop-secretstore.json` located in the same folder:

```json
{
    "redisPassword": "**********",
    "sendgridAPIKey": "**********"
}
```

The `components` folder is specified in the command-line and mounted as a local folder inside the Dapr sidecar container. Here's a snippet from the `docker-compose.override.yml` file in the root of the repository that specifies the volume mount:

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

Note the `/components`  volume mount and `--components-path` command-line argument passed in the `daprd` startup command.

Once configured, other component configuration files can also reference the secrets. Here's an example of the configuration for the Publish/Subscribe component used in eShopOnDapr:

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

Note how the example references secrets from the local Redis store.

## Summary

The Dapr Secrets Management building block provides capabilities for storing and retrieving sensitive configuration settings like passwords and connection-strings. It keeps secrets private and prevents them from being accidentally disclosed.

The building block supports several different secret stores and hides their complexity with the Dapr secrets management API.

The Dapr .NET SDK provides a method, `DaprClient`, to retrieve secrets and a .NET configuration provider that retrieves secrets at application startup. It exposes secrets as configuration values you can consume in your .NET code from the .NET Core configuration system.

You can retrieve secrets in your application code and reference them from Dapr component configuration files.

You can use secret scopes to control access to specific secrets.

>[!div class="step-by-step"]
>[Previous](observability.md)
>[Next](actors.md)
