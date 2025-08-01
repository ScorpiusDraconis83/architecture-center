---
title: Considerations for Updating a Multitenant Solution
description: Learn about update strategies for multitenant solutions, including deployment approaches, tenant requirements, and maintenance windows.
author: johndowns
ms.author: pnp
ms.date: 06/25/2025
ms.topic: conceptual
ms.subservice: architecture-guide
ms.custom: arb-saas
---

# Considerations for updating a multitenant solution

One of the benefits of cloud technology is continuous improvement and evolution. As a service provider, you need to apply updates to your solution. You might need to make changes to your application code, your Azure infrastructure, your database schemas, or any other component. It's important to plan how you update your environments. In a multitenant solution, it's especially important to be clear about your update policy because some of your tenants might be reluctant to allow changes to their environments, or they might have requirements that limit the conditions under which you can update their service.

When planning a strategy to update your solution, you need to:

- Identify your tenants' requirements.
- Clarify your own requirements to operate your service.
- Find a balance that both you and your tenants can accept.
- Communicate your strategy clearly to your tenants and other stakeholders.

This article provides guidance for technical decision-makers about approaches to update tenant software and the trade-offs involved.

## Your customers' requirements

Customers often have explicit or implicit requirements that can affect how your system is updated. Consider the following aspects to build up a picture of any points of concern that customers might raise:

- **Expectations and requirements:** Uncover any expectations or requirements that customers might have about when their solution can be updated. These expectations or requirements might be formally communicated to you in contracts or service-level agreements, or they might be informal.

- **Maintenance windows:** Understand whether your customers expect service-defined or self-defined maintenance windows. They might need to communicate to their users about potential outages. They might also expect to test important aspects of your service after the update is complete.

- **Regulations:** Clarify whether customers have any regulatory concerns that require extra approval before updates can be applied. For example, if you provide a health solution that includes IoT components, you might need to get approval from the United States Food and Drug Administration (FDA) before applying an update.

- **Sensitivity:** Understand whether any of your customers are particularly sensitive or resistant to having updates applied. If they are, try to understand why. For example, if they run a physical store or a retail website, they might want to avoid updates around Black Friday, because the risks are higher than potential benefits.

- **History:** Review your own track record of successfully completing updates without any impact to your customers. You should follow good DevOps, testing, deployment, and monitoring practices to reduce the likelihood of outages, and to ensure that you quickly identify any problems that updates introduce. If your customers know that you're able to update their environments smoothly, they're less likely to object.

- **Rollback:** Consider whether customers want to roll back updates if there's a breaking change and who triggers such a rollback request.

## Your requirements

You also need to consider the following questions from your own perspective:

- **Control you're willing to provide:** Is it reasonable for your customers to have control over when updates are applied? If you're building a solution used by large enterprise customers, the answer might be yes. However, if you're building a consumer-focused solution, it's unlikely that you give any control over how you upgrade or operate your solution.

- **Versions:** How many versions of your solution can you reasonably maintain at one time? If you find a bug or security vulnerability and need to apply a hotfix, you might need to apply it to all versions currently in use.

- **Consequences of old versions:** What's the impact of letting customers fall too far behind the current version? If you release new features regularly, do old versions become obsolete quickly? Also, depending on your upgrade strategy and the types of changes, you might need to maintain separate infrastructures for each version of your solution. So, there might be both operational and financial costs, as you maintain support for older versions.

- **Rollback:** Can your deployment strategy support rollbacks to previous versions? Do you want to enable this capability?

> [!NOTE]
> Consider whether you need to take your solution offline for updates or maintenance. Outage windows are generally considered an outdated practice for software as a service (SaaS). Modern DevOps practices and cloud technologies enable you to avoid downtime during updates and maintenance. However, you need to design for zero-downtime deployments. It's important to consider your update process when you plan your solution architecture.
>
> Even if you don't plan for outages during your update process, you might still define a regular maintenance window. This approach helps communicate to your customers that changes occur during specific times.
>
> For more information about how to achieve zero-downtime deployments, see [Eliminate downtime through versioned service updates](/devops/operate/achieving-no-downtime-versioned-service-updates).

## Find a balance

If you leave cadence of your service updates entirely to your tenants' discretion, they might choose to never update. It's important to allow yourself to update your solution, while factoring in any reasonable concerns or constraints that your customers might have. For example, if a customer is particularly sensitive to updates on a Friday because that's their busiest day of the week, consider whether you can just as easily defer updates to Monday without affecting your solution.

One approach that can work well is to roll out updates on a tenant-by-tenant basis by using one of the [deployment strategies](#deployment-strategies-to-support-updates). Notify your customers about planned updates. Allow customers to temporarily but not permanently opt out. Put a reasonable limit on when you require the update to be applied.

Consider allowing yourself the ability to deploy security patches, or other critical hotfixes, with minimal or no advance notice. Ensure that tenants understand this practice and its importance in safeguarding their data.

Another approach can be to allow tenants to initiate their own updates during a time that they choose. Again, you should provide a deadline, at which point you apply the update on their behalf.

> [!WARNING]
> Be careful about enabling tenants to initiate their own updates. This process is complex to implement, and it requires significant development and testing effort to deliver and maintain.

Regardless of what you do, ensure that you have a process to monitor the health of your tenants, especially before and after updates are applied. Critical production incidents, also known as *live-site incidents*, often occur after changes to code or configuration. As a result, it's important to proactively monitor for and respond to any problems to retain customer confidence. For more information, see [Incident management for SaaS workloads on Azure](/azure/well-architected/saas/incident-management).

## Communicate with your customers

Clear communication is key to building your customers' confidence. It's important to explain the benefits of regular updates, including new features, bug fixes, resolving security vulnerabilities, and performance improvements. One of the benefits of a modern cloud-hosted solution is the ongoing delivery of features and updates.

Consider the following questions:

- Will you notify customers of upcoming updates?

- If you do, will you implicitly request permission by providing an opt-out process, and what are the limits on opting out?

- Do you have a scheduled maintenance window that you use when you apply updates?

- What happens if you have an emergency update, like a critical security patch? Can you force updates in those situations?

- If you can't proactively notify customer of upcoming updates, can you provide retrospective notifications? For example, can you update a page on your website with the list of updates that you've applied?

- How many separate versions of your system will you maintain in production?

## Communicate with your customer support team

It's important that your own support team has full visibility into updates that have been applied to each tenant's infrastructure. Customer support representatives should be able to easily answer the following questions:

- Have updates recently been applied to a tenant's infrastructure or to shared components?

- What was the nature of those updates?

- What was the previous version?

- How frequently are updates applied to this tenant?

If one of your customers has a problem because of an update, you need to ensure your customer support team has the information necessary to understand what's changed.

## Deployment strategies to support updates

Consider how you deploy updates to your infrastructure. Your update strategy depends heavily on the [tenancy model](tenancy-models.md) that you use. Three common approaches for deploying updates are deployment stamps, feature flags, and deployment rings. You can use these approaches independently, or you can combine them together to meet more complex requirements.

In all cases, ensure that you have sufficient reporting and visibility. You need to know what version of infrastructure, software, or feature each tenant uses, what they're eligible to migrate to, and any time-related data tied to those states. Tracking this information is often one of the responsibilities of a [control plane](./control-planes.md).

### Deployment Stamps pattern

Many multitenant applications are a good fit for the [Deployment Stamps pattern](../../../patterns/deployment-stamp.yml). In this pattern, you deploy multiple copies of your application and other components. Depending on your isolation requirements, you might deploy a stamp for each tenant or shared stamps that run multiple tenants' workloads.

Stamps are a great way to provide isolation between tenants. They also provide you with flexibility for your update process because you can roll out updates progressively across stamps without affecting others.

### Feature flags

[Feature flags](/devops/operate/progressive-experimentation-feature-flags) enable you to add functionality to your solution, while only exposing that functionality to a subset of your customers or tenants.

Consider using feature flags if either of these scenarios apply to you:

- You deploy updates regularly but want to avoid showing new functionality until it's fully implemented.

- You want to avoid applying changes in behavior until a customer opts in.

You can embed feature flag support into your application by writing code yourself or by using a service like [Azure App Configuration](/azure/azure-app-configuration/overview).

### Deployment rings

[Deployment rings](/devops/operate/safe-deployment-practices) enable you to progressively roll out updates across a set of tenants or deployment stamps. You can assign a subset of tenants to each ring in the rollout sequence.

You can determine how many rings to create and what each ring means for your own solution. Commonly, organizations use the following rings:

- **Canary:** A canary ring includes your own test tenants and customers who want to receive updates as soon as they're available. Anybody on the canary ring should understand that they might receive more frequent updates. These updates might not have gone through as comprehensive a validation process as updates in other rings.

- **Early adopter:** An early adopter ring contains tenants who are slightly more risk-averse but still prepared to receive regular updates.

- **Users:** Most of your tenants belong to the *users* ring, which receives less frequent and more highly tested updates.

### API versions

If your service exposes an external API, consider that any updates you apply might affect the way that customers or partners integrate with your platform. In particular, you need to be conscious of breaking changes to your APIs. Consider using [an API versioning strategy](../../../best-practices/api-design.md#implement-versioning) to mitigate the risk of updates to your API.

## Contributors

*Microsoft maintains this article. The following contributors wrote this article.*

Principal author:

- [John Downs](https://www.linkedin.com/in/john-downs/) | Principal Software Engineer, Azure Patterns & Practices

Other contributors:

- [Chad Kittel](https://www.linkedin.com/in/chadkittel/) | Principal Software Engineer, Azure Patterns & Practices
- [Daniel Scott-Raynsford](https://www.linkedin.com/in/dscottraynsford/) | Partner Technology Strategist
- [Arsen Vladimirskiy](https://www.linkedin.com/in/arsenv/) | Principal Customer Engineer, FastTrack for Azure

*To see nonpublic LinkedIn profiles, sign in to LinkedIn.*

## Related resource

- [Map requests to tenants in a multitenant solution](map-requests.yml)
