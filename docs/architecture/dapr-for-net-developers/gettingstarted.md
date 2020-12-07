---
title: Getting started with Dapr
description:
author: amolenk 
ms.date: 12/04/2020
---

# Getting started with Dapr

Outline:

- Chapter intro

- Walkthrough: Install Dapr / Dapr CLI
  - Install Dapr in your local environment for testing and self-hosting
  - Prerequisites: Docker Desktop (https://docs.docker.com/install/)
  - https://docs.dapr.io/operations/hosting/self-hosted/self-hosted-no-docker/
  - Start by installing the Dapr CLI: https://docs.dapr.io/getting-started/install-dapr-cli/
  - Run `dapr init`
  - [Screenshot results]
  - You can verify the installation by making sure that the `daprio/dapr`, `openzipkin/zipkin`, and `redis` containers are running. `docker ps` 
- Introduction to Dapr .NET SDK
  - You are now ready to start developing with Dapr.
  - In this section you'll create an application that ... You will use Dapr self-hosted mode to run the application from the command line.

- Walkthrough: Use Dapr from .NET
  - Create a new .NET Web API 
  - Register Dapr components in Startup class
  - Add a new controller that uses Dapr state management
  - Explain how to run from CLI
  - Bonus: Visual Studio Code extension
- Explain component files (the sample above works due to the standard Redis component configuration installed with Dapr CLI)
  - Include namespaces & scopes introduction
- Explain that eShop uses Docker
- Walkthrough: Multiple services with Docker compose
  - Create solution with two services in Visual Studio
  - Daprize
  - Debug
- Get started on Kubernetes?
- Summary



## Summary


### References

https://docs.dapr.io/getting-started/install-dapr/



>[!div class="step-by-step"]
>[Previous](index.md)
>[Next](index.md)
