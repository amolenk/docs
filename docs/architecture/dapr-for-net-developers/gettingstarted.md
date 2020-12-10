---
title: Getting started with Dapr
description:
author: amolenk 
ms.date: 12/04/2020
---

# Getting started with Dapr

**TODO** Introduction

- [ ] Explain component files (the sample above works due to the standard Redis component configuration installed with Dapr CLI)

- [ ] Include namespaces & scopes introduction
- [ ] Explain that eShop uses Docker
- [ ] Check numbering style

## Installing Dapr into your local environment

In this section we'll explain how to install Dapr into your local development environment. With Dapr installed locally, you can run Daprized applications in self-hosted mode. 

Follow these steps to install Dapr locally:

1. [Install the Dapr CLI](https://docs.dapr.io/getting-started/install-dapr-cli/). The Dapr CLI allows you to setup Dapr on your local environment or on a Kubernetes cluster. It also provides debugging support, and launches and manages Dapr instances.

2. Install [Docker Desktop](https://docs.docker.com/get-docker/). If you're running on Windows, make sure that **Docker Desktop for Windows** is configured to use Linux containers.

   > By default, Dapr uses Docker containers to give you the best out-of-the-box experience. If you want to install Dapr locally without requiring Docker, you can skip these steps and [execute a *slim* initialization](https://docs.dapr.io/operations/hosting/self-hosted/self-hosted-no-docker/) instead.  

3. [Initialize Dapr using the CLI](https://docs.dapr.io/getting-started/install-dapr/).

Now that Dapr is running in your local environment, let's build your first Dapr application!

## Building your first Dapr application

Now that you've installed Dapr into your local environment, you're ready to start developing with Dapr. In this walkthrough, we'll show you how to build a simple .NET Console application that uses the Dapr State Management building block.

To complete this walkthrough, you'll also need to install the [.NET Core 3 Development Tools](https://dotnet.microsoft.com/download/dotnet-core/3.1) for development with .NET Core 3.1.

#### Create the application

1. In your terminal, enter the following command to create a new .NET Console application:

   ```terminal
   ~$ dotnet new console -o DaprCounter
   ```

2. Then, navigate to the new directory created by the previous command:

   ```terminal
   ~$ cd DaprCounter
   ```

3. The template creates a simple "Hello World" application. The application displays "Hello World!" when you run it:

   ```terminal
   ~$ dotnet run
   Hello World!
   ```

#### Add Dapr State Management

In this part of the walkthrough, you'll use the Dapr State Management building block to implement a stateful counter in the program. 

While you can use both HTTP and gRPC to call the Dapr APIs, it's much nicer to use the Dapr .NET SDK. Using the Dapr .NET SDK feels more natural for a .NET developer because it provides a strongly typed client to call the Dapr APIs. It also provides integration with ASP.NET Core, making it even easier to use Dapr features such as pub/sub messaging and state management from ASP.NET Core applications.

1. Add the `Dapr.Client` NuGet package to your application:

   ```
   ~$ dotnet add package Dapr.Client
   ```

2. Now open the `Program.cs` file and change its contents to:

   ```c#
   using System;
   using System.Threading.Tasks;
   using Dapr;
   using Dapr.Client;
   
   namespace DaprCounter
   {
       class Program
       {
           static async Task Main(string[] args)
           {
               DaprClient daprClient = new DaprClientBuilder().Build();
   
               int counter = await daprClient.GetStateAsync<int>("statestore", "counter");
   
               while (true)
               {
                   Console.WriteLine($"Counter = {counter++}");
   
                   await daprClient.SaveStateAsync("statestore", "counter", counter);
   
                   await Task.Delay(1000);
               }
           }
       }
   }
   ```

3. Run your application using the Dapr CLI. 

   ```
   ~$ dapr run --app-id DaprCounter dotnet run
   ```

   



Why does this work? Explain that Dapr uses default components.





















- Introduction to Dapr .NET SDK

- - You are now ready to start developing with Dapr.
  - In this section you'll create an application that ... You will use Dapr self-hosted mode to run the application from the command line.

- Walkthrough: Use Dapr from .NET
  - Create a new .NET Web API 
  - Register Dapr components in Startup class
  - Add a new controller that uses Dapr state management
  - Explain how to run from CLI
  - Bonus: Visual Studio Code extension





## Docker Compose

eShopOnContainers uses Docker Compose to run locally.

Docker Compose is...

It allows us to combine the services and the infrastructure containers.

Let's see how to set it up from scratch:

> In this tutorial, you'll learn how to manage more than one container and communicate between them when using Container Tools in Visual Studio. Managing multiple containers requires *container orchestration* and requires an orchestrator such as Docker Compose, Kubernetes, or Service Fabric. Here, we'll use Docker Compose. Docker Compose is great for local debugging and testing in the course of the development cycle.'

To complete this walkthrough, you'll need to install the following prerequisites:

- [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads) with the **Web Development**, **Azure Tools** workload, and/or **.NET Core cross-platform development**workload installed
- [.NET Core 3 Development Tools](https://dotnet.microsoft.com/download/dotnet-core/3.1) for development with .NET Core 3.1.

### Create a Web Application project

In Visual Studio, create an **ASP.NET Core Web Application** project, named `WebFrontEnd`. Select **Web Application** to create a web application with Razor pages.

![Screenshot of creating the web project](https://docs.microsoft.com/en-us/visualstudio/containers/media/tutorial-multicontainer/vs-2019/new-aspnet-core-project1.png?view=vs-2019)

Do not select **Enable Docker Support**. You'll add Docker support later.

![Screenshot of creating the web project](https://docs.microsoft.com/en-us/visualstudio/containers/media/tutorial-multicontainer/vs-2019/new-aspnet-core-project.png?view=vs-2019)

### Create a Web API project

Add a project to the same solution and call it *MyWebAPI*. Select **API** as the project type, and clear the checkbox for **Configure for HTTPS**. In this design, we're only using SSL for communication with the client, not for communication from between containers in the same web application. Only `WebFrontEnd` needs HTTPS and the code in the examples assumes that you have cleared that checkbox. In general, the .NET developer certificates used by Visual Studio are only supported for external-to-container requests, not for container-to-container requests.

![Screenshot of creating the Web API project](https://docs.microsoft.com/en-us/visualstudio/containers/media/tutorial-multicontainer/vs-2019/web-api-project.png?view=vs-2019)



### Add code to call the Web API

1. In the `WebFrontEnd` project, open the *Index.cshtml.cs* file, and replace the `OnGet` method with the following code.

   C#Copy

   ```csharp
    public async Task OnGet()
    {
       ViewData["Message"] = "Hello from webfrontend";
   
       using (var client = new System.Net.Http.HttpClient())
       {
          // Call *mywebapi*, and display its response in the page
          var request = new System.Net.Http.HttpRequestMessage();
          // request.RequestUri = new Uri("http://mywebapi/WeatherForecast"); // ASP.NET 3 (VS 2019 only)
          request.RequestUri = new Uri("http://mywebapi/api/values/1"); // ASP.NET 2.x
          var response = await client.SendAsync(request);
          ViewData["Message"] += " and " + await response.Content.ReadAsStringAsync();
       }
    }
   ```

    Note

   In real-world code, you shouldn't dispose `HttpClient` after every request. For best practices, see [Use HttpClientFactory to implement resilient HTTP requests](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests).

   For .NET Core 3.1 in Visual Studio 2019 or later, the Web API template uses a WeatherForecast API, so uncomment that line and comment out the line for ASP.NET 2.x.

2. In the *Index.cshtml* file, add a line to display `ViewData["Message"]` so that the file looks like the following code:

   CSHTMLCopy

   ```cshtml
   @page
   @model IndexModel
   @{
       ViewData["Title"] = "Home page";
   }
   
   <div class="text-center">
       <h1 class="display-4">Welcome</h1>
       <p>Learn about <a href="/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
       <p>@ViewData["Message"]</p>
   </div>
   ```

3. (ASP.NET 2.x only) Now in the Web API project, add code to the Values controller to customize the message returned by the API for the call you added from *webfrontend*.

   C#Copy

   ```csharp
     // GET api/values/5
     [HttpGet("{id}")]
     public ActionResult<string> Get(int id)
     {
         return "webapi (with value " + id + ")";
     }
   ```

   With .NET Core 3.1, you don't need this, because you can use the WeatherForecast API that is already there. However, you need to comment out the call to [UseHttpsRedirection](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.httpspolicybuilderextensions.usehttpsredirection) in the `Configure` method in *Startup.cs*, because this code uses HTTP, not HTTPS, to call the Web API.

   C#Copy

   ```csharp
               //app.UseHttpsRedirection();
   ```

4. In the `WebFrontEnd` project, choose **Add > Container Orchestrator Support**. The **Docker Support Options** dialog appears.

5. Choose **Docker Compose**.

6. Choose your Target OS, for example, Linux.

   ![Screenshot of choosing the Target OS](https://docs.microsoft.com/en-us/visualstudio/containers/media/tutorial-multicontainer/docker-tutorial-docker-support-options.png?view=vs-2019)

   Visual Studio creates a *docker-compose.yml* file and a *.dockerignore* file in the **docker-compose** node in the solution, and that project shows in boldface font, which shows that it's the startup project.

   ![Screenshot of Solution Explorer with docker-compose project added](https://docs.microsoft.com/en-us/visualstudio/containers/media/tutorial-multicontainer/multicontainer-solution-explorer.png?view=vs-2019)

   The *docker-compose.yml* appears as follows:

   YAMLCopy

   ```yaml
   version: '3.4'
   
    services:
      webfrontend:
        image: ${DOCKER_REGISTRY-}webfrontend
        build:
          context: .
          dockerfile: WebFrontEnd/Dockerfile
   ```

   The *.dockerignore* file contains file types and extensions that you don't want Docker to include in the container. These files are generally associated with the development environment and source control, not part of the app or service you're developing.

   Look at the **Container Tools** section of the output pane for details of the commands being run. You can see the command-line tool docker-compose is used to configure and create the runtime containers.

7. In the Web API project, again right-click on the project node, and choose **Add** > **Container Orchestrator Support**. Choose **Docker Compose**, and then select the same target OS.

    Note

   In this step, Visual Studio will offer to create a Dockerfile. If you do this on a project that already has Docker support, you are prompted whether you want to overwrite the existing Dockerfile. If you've made changes in your Dockerfile that you want to keep, choose no.

   Visual Studio makes some changes to your docker compose YML file. Now both services are included.

   YAMLCopy

   ```yaml
   version: '3.4'
   
   services:
     webfrontend:
       image: ${DOCKER_REGISTRY-}webfrontend
       build:
         context: .
         dockerfile: WebFrontEnd/Dockerfile
   
     mywebapi:
       image: ${DOCKER_REGISTRY-}mywebapi
       build:
         context: .
         dockerfile: MyWebAPI/Dockerfile
   ```

8. Run the site locally now (F5 or Ctrl+F5) to verify that it works as expected. If everything is configured correctly with the .NET Core 2.x version, you see the message "Hello from webfrontend and webapi (with value 1)." With .NET Core 3, you see weather forecast data.

   The first project that you use when you add container orchestration is set up to be launched when you run or debug. You can configure the launch action in the **Project Properties** for the docker-compose project. On the docker-compose project node, right-click to open the context menu, and then choose **Properties**, or use Alt+Enter. The following screenshot shows the properties you would want for the solution used here. For example, you can change the page that is loaded by customizing the **Service URL** property.

   ![Screenshot of docker-compose project properties](https://docs.microsoft.com/en-us/visualstudio/containers/media/tutorial-multicontainer/launch-action.png?view=vs-2019)

   Here's what you see when launched (the .NET Core 2.x version):

   ![Screenshot of running web app](https://docs.microsoft.com/en-us/visualstudio/containers/media/tutorial-multicontainer/webfrontend.png?view=vs-2019)

   The web app for .NET 3.1 shows the weather data in JSON format.

## Get started on Kubernetes?

**TODO**

## Summary


### References

https://docs.dapr.io/getting-started/install-dapr/

https://docs.dapr.io/reference/cli/



>[!div class="step-by-step"]
>[Previous](index.md)
>[Next](index.md)
