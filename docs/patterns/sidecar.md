---
title: Sidecar pattern
description: Deploy supporting features of an application into a separate process or container with a shared lifecycle to provide modular abstraction and isolation of cross-cutting concerns.
author: claytonsiemens77
ms.author: pnp
ms.date: 02/03/2026
ms.topic: design-pattern
ms.subservice: cloud-fundamentals
---
# Sidecar pattern

Deploy components of an application into a separate process or container to provide isolation and encapsulation. This pattern can also enable applications to be composed of heterogeneous components and technologies.

This pattern is named *Sidecar* because it resembles a sidecar attached to a motorcycle. In the pattern, the sidecar is attached to a parent application and provides supporting features for the application. The sidecar also shares the same lifecycle as the parent application, being created and retired alongside the parent. The sidecar pattern is sometimes referred to as the sidekick pattern and is a decomposition pattern.

## Context and problem

Applications and services often require related functionality, such as monitoring, logging, configuration, and networking services. These peripheral tasks can be implemented as separate components or services.

If they're tightly integrated into the application, they can run in the same process as the application, making efficient use of shared resources. However, this also means they're not well isolated, and an outage in one of these components can affect other components or the entire application. Also, they usually need to be implemented using the same language as the parent application. As a result, the component and the application have close interdependence on each other.

If the application is decomposed into services, then each service can be built using different languages and technologies. While this gives more flexibility, it means that each component has its own dependencies and requires language-specific libraries to access the underlying platform and any resources shared with the parent application. In addition, deploying these features as separate services can add latency to the application. Managing the code and dependencies for these language-specific interfaces can also add considerable complexity, especially for hosting, deployment, and management.

## Solution

Co-locate a cohesive set of tasks with the primary application, but place them inside their own process or container, providing a homogeneous interface for platform services across languages.

![Diagram of the Sidecar pattern](./_images/sidecar.png)

A sidecar service isn't necessarily part of the application, but is connected to it. It goes wherever the parent application goes. Sidecars are supporting processes or services that are deployed with the primary application. On a motorcycle, the sidecar is attached to one motorcycle, and each motorcycle can have its own sidecar. In the same way, a sidecar service shares the fate of its parent application. For each instance of the application, an instance of the sidecar is deployed and hosted alongside it.

Advantages of using a sidecar pattern include:

- A sidecar is independent from its primary application in terms of runtime environment and programming language, so you don't need to develop one sidecar per language.

- The sidecar can access the same resources as the primary application. For example, a sidecar can monitor system resources used by both the sidecar and the primary application.

- Because of its proximity to the primary application, there's no significant latency when communicating between them.

- Even for applications that don't provide an extensibility mechanism, you can use a sidecar to extend functionality by attaching it as its own process in the same host or sub-container as the primary application.

The sidecar pattern is often used with containers and referred to as a sidecar container or sidekick container.

## Problems and considerations

Consider the following points as you decide how to implement this pattern:

- Consider the deployment and packaging format you will use to deploy services, processes, or containers. Containers are particularly well suited to the sidecar pattern.
- When designing a sidecar service, carefully decide on the interprocess communication mechanism. Try to use language- or framework-agnostic technologies unless performance requirements make that impractical.
- Before putting functionality into a sidecar, consider whether it would work better as a separate service or a more traditional daemon.
- Also consider whether the functionality could be implemented as a library or using a traditional extension mechanism. Language-specific libraries might have a deeper level of integration and less network overhead.

## When to use this pattern

Use this pattern when:

- Your primary application uses a heterogeneous set of languages and frameworks. A component located in a sidecar service can be consumed by applications written in different languages using different frameworks.
- A component is owned by a different team or a third-party.
- A component or feature must be co-located on the same host as the application.
- You need a service that shares the overall lifecycle of your main application, but can be independently updated.
- You need fine-grained control over resource limits for a particular resource or component. For example, you might want to restrict the amount of memory a specific component uses. You can deploy the component as a sidecar and manage memory usage independently of the main application.

This pattern might not be suitable when:

- When interprocess communication needs to be optimized. Communication between a parent application and sidecar services includes some overhead, notably latency in the calls. This might not be an acceptable trade-off for chatty interfaces.
- For small applications where the resource cost of deploying a sidecar service for each instance isn't worth the advantage of isolation.
- When the service needs to scale differently than or independently from the main applications. If so, it might be better to deploy the feature as a separate service.
- Your application platform already offers equivalent functionality without colocated processes.

## Workload design

An architect should evaluate how the Sidecar pattern can be used in their workload's design to address the goals and principles covered in the [Azure Well-Architected Framework pillars](/azure/well-architected/pillars). For example:

| Pillar | How this pattern supports pillar goals |
| :----- | :------------------------------------- |
| [Security](/azure/well-architected/security/checklist) design decisions help ensure the **confidentiality**, **integrity**, and **availability** of your workload's data and systems. | By encapsulating these task and deploying them out-of-process, you can reduce the surface area of sensitive processes to only the code that's needed to accomplish the task. You can also use sidecars to add cross-cutting security controls to an application component that's not natively designed with that functionality.<br/><br/> - [SE:04 Segmentation](/azure/well-architected/security/segmentation)<br/> - [SE:07 Encryption](/azure/well-architected/security/encryption) |
| [Operational Excellence](/azure/well-architected/operational-excellence/checklist) helps deliver **workload quality** through **standardized processes** and team cohesion. | This pattern provides an approach to implementing flexibility in tool integration that might enhance the application's observability without requiring the application to take direct implementation dependencies. It enables the sidecar functionality to evolve independently and be maintained independently of the application's lifecycle.<br/><br/> - [OE:04 Tools and processes](/azure/well-architected/operational-excellence/tools-processes)<br/> - [OE:07 Monitoring system](/azure/well-architected/operational-excellence/observability) |
| [Performance Efficiency](/azure/well-architected/performance-efficiency/checklist) helps your workload **efficiently meet demands** through optimizations in scaling, data, code. | You can move cross-cutting tasks to a single process that can scale across multiple instances of the main process, which reduces the need to deploy duplicate functionality for each instance of the application.<br/><br/> - [PE:07 Code and infrastructure](/azure/well-architected/performance-efficiency/optimize-code-infrastructure) |

As with any design decision, consider any tradeoffs against the goals of the other pillars that might be introduced with this pattern.

## Example

The sidecar pattern is applicable to many scenarios. Some common examples:

- Dependency abstraction. A custom service that's deployed alongside each application, instead of a language-specific client library, to provide access to shared dependency capabilities. The service is loaded as a sidecar and exposes a consistent API for concerns such as logging, configuration, service discovery, state management, and health checks.

  [Dapr sidecar](https://docs.dapr.io/concepts/dapr-services/sidecar/) is an example implementation of this use case.

- Service mesh data plane. A sidecar proxy is injected alongside each service instance to handle cross‑cutting networking concerns such as traffic routing, retries, mTLS, policy enforcement, and telemetry. Service meshes such as Istio and Linkerd use sidecar proxies to implement these capabilities without requiring changes to application code.
- Ambassador sidecar. Deploy an [ambassador](./ambassador.yml) service as a sidecar. The application calls through the ambassador, which handles request logging, routing, circuit breaking, and other connectivity related features. This pattern is commonly implemented using data‑plane proxies such as Envoy.
- Protocol Adapters. A sidecar converts between incompatible protocols, data formats, or acts as a [message bridge](messaging-bridge.yml). This enables the application to use simpler or legacy interfaces.

- Telemetry enrichment. A sidecar preprocesses or enriches telemetry data (metrics, logs, and traces) before forwarding it to external monitoring systems. Components such as the [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/deploy/agent/) can run as sidecars to normalize, enrich, or route telemetry without embedding those responsibilities in the application.

## Next steps

- [Microservice APIs powered by Dapr](/azure/container-apps/dapr-overview). Azure Container Apps provides APIs powered by Distributed Application Runtime (Dapr) that help you write and implement simple, portable, resilient, and secured microservices. It uses sidecar.
- [Native sidecar mode for Istio-based service mesh add-on in Azure Kubernetes Service (AKS)](/azure/aks/istio-native-sidecar). Istio addresses the challenges developers and operators face with a distributed or microservices architecture. The Istio-based service mesh add-on provides an officially supported and tested integration for Azure Kubernetes Service (AKS).

## Related resources

- [Ambassador pattern](./ambassador.yml)