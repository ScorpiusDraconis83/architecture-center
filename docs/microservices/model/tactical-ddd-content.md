Domain-driven design (DDD) opposes the idea of having a single unified model for the entire system. Instead, it encourages dividing the system into bounded contexts, each with its own model. The [domain analysis](domain-analysis.md) article covers the strategic phase of DDD. This article continues the steps described in that article by applying tactical DDD patterns to define the domain models with more precision within their bounded context.

In a microservices architecture, where each bounded context is a microservice candidate, the entity and aggregate patterns are of note. Applying these patterns helps identify natural boundaries for the services in your application. For more information, see [Identify microservice boundaries](./microservice-boundaries.yml). As a general principle, a microservice should be no smaller than an aggregate and no larger than a bounded context.

This article reviews the tactical patterns and then applies them to the Shipping bounded context in the Drone Delivery application.

## Overview of the tactical patterns

This section provides a brief summary of the tactical DDD patterns. If you're familiar with DDD, you might choose to skip it. These patterns are described in more detail in Eric Evans' *Domain-Driven Design*, the book that first introduced the term. Another good reference is *Learning Domain-Driven Design* by Vlad Khononov for a practical, modern treatment of the subject.

:::image type="complex" border="false" source="../images/ddd-patterns.png" alt-text="Diagram of tactical patterns in DDD." lightbox="../images/ddd-patterns.png":::
   The diagram has five key sections. An arrow points from Application service to Domain service. One arrow points from Domain service to the Aggregate section. Another arrow points from Domain service to an Aggregate section that contains Root entity, Entity, and Value object. A line points from the first Aggregate section to the Domain event section.
:::image-end:::

### Entities

An entity is an object with a unique identity that persists over time. For example, in a banking application, customers and accounts would be entities.

- An entity has a unique identifier in the system, which can be used to look up or retrieve the entity.

  - Two entity instances that share the same identity represent the same domain concept, even if their attributes differ at a given point in time. For instance, a person's name or address might change, but they remain the same individual. Conversely, two instances with identical attributes but different identities are distinct entities.

  - The identifier is always exposed directly to users. It could be a GUID or a primary key in a database.

    The choice of identity strategy matters: natural keys (such as an order number or government-issued ID) carry business meaning and can be recognized across systems, while surrogate keys (such as GUIDs) are generated without business meaning but avoid coupling to external systems. In a microservices architecture, other services reference entities by their identifiers, so the identity must be stable and meaningful across service boundaries. An identity can span multiple bounded contexts and might persist beyond the lifetime of the application.

- Entities should encapsulate behavior, not just carry data. An entity that contains only properties with getters and setters, while all business logic lives in external service classes, is an *anemic domain model*. This anti-pattern loses the core benefit of DDD: expressing business rules in the domain model itself. Place validation, state transitions, and business rules inside the entity. For example, a `Delivery` entity should contain the logic for whether it can be canceled, rather than delegating that decision to an external service.

### Value objects

A value object has no identity. It's defined only by the values of its attributes. Two value objects with the same attribute values are interchangeable. Common examples include colors, dates and times, currency amounts, and measurements.

- Value objects are immutable. To update a value object, you create a new instance to replace the old one. Immutable objects are safe to share across threads, can be cached without defensive copying, and are easier to reason about in distributed systems.

- Value objects can include methods that encapsulate domain logic, but those methods should not produce side effects. They return new value objects instead.

Prefer value objects as your default modeling choice. Only promote a concept to an entity when you need to track its identity over time. For example, an `Address` is typically a value object — two addresses with the same street, city, and postal code are interchangeable. But if your domain needs to track a specific address record over time (for example, for audit purposes), then it becomes an entity.

### Aggregates

An aggregate defines a consistency boundary around one or more entities. Exactly one entity in an aggregate is the root. Lookup is done using the root entity's identifier. Any other entities in the aggregate are children of the root, and are referenced by following pointers from the root.

The purpose of an aggregate is to model transactional invariants. Things in the real world have complex webs of relationships. Customers create orders, orders contain products, and products have suppliers. If the application modifies several related objects, how does it guarantee consistency? How do we keep track of invariants and enforce them?

Traditional applications have often used database transactions to enforce consistency. In a distributed application, however, that's often not feasible. A single business transaction might span multiple data stores, or might be long running, or might involve third-party services. Ultimately it's up to the application, not the data layer, to enforce the invariants required for the domain. That's what aggregates are meant to model.

> [!NOTE]
> An aggregate might consist of a single entity, without child entities. What makes it an aggregate is the transactional boundary.

When you design aggregates, keep these rules in mind:

- **Design small aggregates.** Include only the data that must be consistent within a single transaction. In the drone delivery example, Delivery, Package, Drone, and Account are each separate aggregates because they have independent lifecycles. Combining them would force unrelated updates to contend for the same locks.

- **Reference other aggregates by identity only.** The Delivery aggregate stores a `DroneId` and a `PackageId`, not direct references to those objects. This keeps aggregates decoupled — a property that maps directly to microservice boundaries.

- **Use eventual consistency across aggregates.** When a business process spans multiple aggregates, use domain events rather than a single transaction. When a delivery is completed, the Delivery aggregate raises a `DeliveryCompleted` event that other services react to asynchronously.

### Domain and application services

In DDD terminology, a service is a stateless object that implements logic that doesn't naturally belong to an entity or value object.

> [!TIP]
> The term *service* is overloaded in software development. The definition used here isn't directly related to microservices.

- **Domain services** encapsulate business rules that span multiple entities or aggregates. In the Shipping bounded context, the Scheduler is a domain service — scheduling logic involves business rules about drone availability, delivery windows, and route optimization that don't belong to any single entity.

- **Application services** orchestrate use cases. They coordinate calls to domain services and repositories, manage transactions, and handle concerns such as user authentication or sending an SMS notification. They contain no business logic themselves. An API endpoint that receives a delivery request, calls the Scheduler, and returns the result is an application service.

### Domain events

Domain events represent something meaningful that happened within the domain. For example, "a record was inserted into a table" isn't a domain event. "A delivery was canceled" is. Domain events are raised by aggregates after a state change and are the primary mechanism for coordinating work across aggregate boundaries.

In a microservices architecture, domain events sometimes need to cross microservice boundaries. Unlike internal domain events, integration events are published asynchronously through a message broker after the originating transaction commits. For example, when the Shipping bounded context completes a delivery, it publishes a `DeliveryCompleted` integration event that the Accounts bounded context consumes to trigger invoicing. For more information about asynchronous messaging, see [Interservice communication](../design/interservice-communication.yml).

### Additional patterns

There are a few other DDD patterns not covered here, including factories, repositories, and modules. These patterns can be helpful when you implement a microservice, but they're less relevant when you design the boundaries between microservices.

## Drone delivery: Applying the patterns

We start with the scenarios that the Shipping bounded context must handle.

- A customer can request a drone to pick up goods from a business that is registered with the drone delivery service.
- The sender generates a tag (barcode or RFID) to put on the package.
- A drone will pick up and deliver a package from the source location to the destination location.
- When a customer schedules a delivery, the system provides an ETA based on route information, weather conditions, and historical data.
- When the drone is in flight, a user can track the current location and the latest ETA.
- Until a drone has picked up the package, the customer can cancel a delivery.
- The customer is notified when the delivery is completed.
- The sender can request delivery confirmation from the customer, in the form of a signature or finger print.
- Users can look up the history of a completed delivery.

From these scenarios, the development team identified the following **entities**.

- Delivery
- Package
- Drone
- Account
- Confirmation
- Notification
- Tag

The first four, Delivery, Package, Drone, and Account, are all **aggregates** that represent transactional consistency boundaries. Each has an independent lifecycle and is managed by a different part of the system. For example, a delivery can be created, updated, and completed without requiring a transaction that locks the drone or account. Confirmations and Notifications are child entities of Deliveries because they don't exist independently. Tags are child entities of Packages for the same reason.

The **value objects** in this design include Location, ETA, PackageWeight, and PackageSize. These have no identity of their own and are not tracked over time.

To illustrate, here is a UML diagram of the Delivery aggregate. Notice that it holds references to other aggregates, including Account, Package, and Drone.

:::image type="complex" border="false" source="../images/delivery-entity.png" alt-text="UML diagram of the Delivery aggregate." lightbox="../images/delivery-entity.png":::
   The image contains a Delivery header. Below the header are the following terms: Id string, OwnerID: REF, Pickup: Location, Drop-off: Location, Packages: REF, Expedited: BOOLEAN, Confirmation: Confirmation, and DroneId: REF. Three lines connect this section to the terms Account, Package, and Drone.
:::image-end:::

There are two domain events:

- While a drone is in flight, the Drone entity sends DroneStatus events that describe the drone's location and status (in-flight, landed).

- The Delivery entity sends DeliveryTracking events whenever the stage of a delivery changes. The DeliveryTracking events include DeliveryCreated, DeliveryRescheduled, DeliveryHeadedToDropoff, and DeliveryCompleted.

Notice that these events describe things that are meaningful within the domain model. They describe something about the domain, and aren't tied to a particular programming language construct.

The development team identified one more area of functionality, which doesn't fit neatly into any of the entities described so far. Some part of the system must coordinate all of the steps involved in scheduling or updating a delivery. Therefore, the development team added two **domain services** to the design: a *Scheduler* that coordinates the steps, and a *Supervisor* that monitors the status of each step, in order to detect whether any steps failed or timed out. This approach is a variation of the [Scheduler Agent Supervisor pattern](../../patterns/scheduler-agent-supervisor.yml).

:::image type="complex" border="false" source="../images/drone-ddd.png" alt-text="Diagram of the revised domain model." lightbox="../images/drone-ddd.png":::
   The image contains 11 key sections. An arrow labeled Observes points from Supervisor to Scheduler. An arrow points from Scheduler to Drone. A double-sided arrow labeled Coordinates points from Account to Delivery. An arrow points from Coordinates to Package. A smaller arrow points from Package to Tag. A dotted arrow labeled Drone status points from Drone to Delivery. Two smaller arrows point from Delivery to Confirmation and Notification. A dotted line connects Delivery and Delivery status.
:::image-end:::

## Next steps

The next step is to define the boundaries for each microservice.

> [!div class="nextstepaction"]
> [Identify microservice boundaries](./microservice-boundaries.yml)

## Related resources

- [Microservices architecture design](../../guide/architecture-styles/microservices.md)
- [Design a microservices architecture](../../microservices/design/index.md)
- [Using domain analysis to model microservices](domain-analysis.md)
- [Choose an Azure compute option for microservices](../../microservices/design/compute-options.md)
