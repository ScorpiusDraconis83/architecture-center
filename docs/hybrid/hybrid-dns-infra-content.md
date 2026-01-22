This reference architecture illustrates how to design a hybrid Domain Name System (DNS) solution to resolve names for workloads that are hosted on-premises and in Microsoft Azure. This architecture works for users and other systems that are connecting from on-premises and the public internet.

## Architecture

:::image type="content" source="./images/hybrid-dns-infra.png" alt-text="Diagram showing a Hybrid Domain Name System (DNS)." border="false" lightbox="./images/hybrid-dns-infra.svg" :::

### Workflow

The architecture consists of the following components:

- **On-premises network**. The on-premises network represents a single datacenter that's connected to Azure over an [Azure ExpressRoute][expressroute/overview] or [virtual private network (VPN)][vpn/design] connection. In this scenario, the following components make up the on-premises network:
  - **DNS** servers. These servers represent two servers with DNS service installed that are acting as resolver/forwarder. These DNS servers are used for all computers in the on-premises network as DNS servers. Records must be created on these servers for all endpoints in Azure and on-premises.
  - **On-premises gateway**. The on-premises gateway represents either a VPN termination device or a router connected to Azure ExpressRoute.
- **Platform subscription**. The hub subscription represents an Azure subscription that's used to identity, connectivity and management resources that are shared across multiple Azure-hosted workloads. These resources can be broken down into multiple subscriptions, as described in [Landing zone identity and access management][alz/identity].
  - **Hub virtual network**. The hub virtual network contains connectivity resources such as Azure Virtual Network Gateways, Azure Firewall, Software-Defined WAN Network Virtual Appliances (NVA) or firewall NVAs.
  > [!NOTE]
  > The hub virtual network can be substituted with a [virtual wide area network (WAN)][vwan/overview] hub. In both cases, the DNS servers should have to be hosted in a different [Azure virtual network (VNet)][vnet/overview]. In the Enterprise-scale architecture, that VNet can be maintained in its own subscription entitled the **Identity subscription** if the DNS servers are Windows Server Active Directory Domain Controllers, otherwise it would normally be in the Connectivity subscription.
    - **Gateway subnet**. The gateway subnet hosts the Azure VPN, or ExpressRoute, gateway that's used to provide connectivity back to the on-premises datacenter.
  - **Shared services virtual network**. The shared services virtual network hosts resources that additional services to workload subscriptions, such as DNS servers. If using a self-managed hub-and-spoke architecture you can move shared services to the hub vitual network to reduce the number of virtual network peering hops between workloads and shared services, but extra care should be taken with user-defined routes in the spokes to avoid asymmetric routing when firewalls are involved.
  > [!NOTE]
  > If the DNS servers are going to be Active Directory Domain Controllers, these would typically be deployed in an identity subscription. Consequently, they would be in their own virtual network, since virtual networks cannot span multiple subscriptions.
    - **DNS subnet**. The DNS servers or Azure Private DNS resolver will be located here. If using Azure Private DNS Resolver,
- **Workload subscription**. The connected subscription represents a collection of workloads that require a virtual network and connectivity back to the on-premises network.
  - **Virtual network peering**. This component is a [peering][vnet/peering] connection back to the hub VNet. This connection allows connectivity from the spoke virtual network to other resources reachable from the hub, such as the on-premises network or the DNS Private Resolver. 
  - **Workload subnet**. The default subnet contains a sample workload.
    - **Workload**. In this example a [virtual machine scale set][vmss/overview] or a collection of individual [virtual machines][vm/overview] hosts a workload in Azure that can be accessed from on-premises, Azure, and the public internet.

## Components

- [Virtual Network][vnet/waf-sg]. Azure Virtual Networks (VNet) are the fundamental building blocks for your private network in Azure. VNets can provide connectivity to many types of Azure resources, such as Azure Virtual Machines (VM), to securely communicate with each other, the internet, and on-premises networks.
- [VPN Gateway][vpngw/about]. A VPN Gateway sends encrypted traffic between an Azure virtual network and an on-premises location over the public Internet. You can also use VPN Gateway to send encrypted traffic between Azure virtual networks over the Microsoft network. A VPN gateway is a specific type of virtual network gateway.
- [ExpressRoute Gateway][ergw/about]. An ExpressRoute Gateway connects a virtual network with an ExpressRoute circuit, which is a dedicated line with guaranteed bandwidth between Microsoft and your on-premises environment. An ExpressRoute gateway is a specific type of virtual network gateway.
- [Azure Firewall][azfw/overview]. An Azure Firewall inspects network traffic and only allows legitimate flows. Azure Firewall can be configured as DNS proxy, which enables Fully-Qualified Domain Name (FQDN) network rules and DNS logging. See [Azure Firewall DNS Proxy Details[azfw/dns] for more information about this functionality.
- [Azure Private DNS Zone][dns/overview]. Azure Private DNS Zone provide DNS resolution for Azure workloads. Virtual machines can be autorregistered and integration with private link endpoints can be automated. Azure workloads can use private DNS zones if they are located in a virtual network linked to a specific zone by means of a DNS virtual network link.
- [Azure DNS Private Resolver][dns/resolver/overview]. Azure DNS Private Resolver is a managed service that provides DNS resolution in Azure, including conditional forwarding of DNS requests to other DNS servers. Since it is a Microsoft-managed service, Azure administrators do not need to manage the operating system and can focus on the configuration of DNS functionality.
> [!NOTE]
> If you already have DNS servers such as Windows Server virtual machines acting as Active Directory Domain Controllers (ADDC) you would not need Azure DNS Private Resolver, and your ADDCs would replace Azure DNS Private Resolver in this architecture.

## Hybrid resolution flows

This section explains how hybrid resolution flows work. There are two main cases to be explored:
- An Azure workload tries to resolve a domain name for a system hosted on-premises.
- An on-premises workload tries to resolve a domain name for a system hosted in Azure.

### Azure to on-premises

Azure virtual machines might need to access on-premises systems such as database or monitoring applications. There are different scenarios for this flow:

- If you are using Azure DNS Private Resolver, you can associate the DNS forwarding ruleset to specific virtual networks with ruleset virtual network links, so that DNS requests sent by virtual machines in those virtual networks are forwarded to the DNS servers defined in the ruleset.
- Alternatively, you can configure the inbound endpoint of the Azure DNS Private Resolver as custom DNS server for a virtual network or for a Network Interface Card (NIC). See [Specify DNS Servers][vnet/customdns] for more information. When the DNS requests arrive to the Azure DNS Private Resolver, it will forward it according to its forwarding rulesets.
- Finally, if you are using your own DNS servers running on Azure virtual machines instead of Azure DNS Private Resolver, you should configure the IP addresses of your DNS virtual machines as custom DNS servers on the virtual networks. You will need to configure DNS conditional forwarding yourself following the documentation of your DNS software.

### On-premises

On-premises systems might need name resolution for workloads deployed in Azure or for private endpoints of Azure PaaS services. The on-premises workload will send a DNS request to an on-premises DNS server, and this server will have a forwarding rule for the domain pointing to the inbound endpoint of Azure DNS Private Resolver.

Azure Private Resolver will use Azure DNS for name resolution, so it will be able to resolve any domain matching the private DNS zones linked to the virtual network where it has been deployed.

## Recommendations

The following recommendations apply for most scenarios. Follow these recommendations unless you have a specific requirement that overrides them.

### Extend AD DS to Azure (optional)

If your organization uses Active Directory integrated DNS zones for name resolution, you should extend your Active Directory domain to Azure. This implies having an additional set of Active Directory Domain Controllers running in an Azure virtual network, ideally in the identity subscription.

### Configure split-brain DNS

If you have applications that should be accessed internally and externally over the public Internet depending on where users are located, you can configure split-brain DNS. Split-brain consists in offering different name resolution for the same fully-qualified domain name (FQDN) to on-premises and Internet users. Internet users would resolve the application FQDN using publicly available Azure DNS zones, while internal users would be directed to private DNS zones, hosted either on your on-premises environment or in Azure with Azure Private DNS Zones.

### Use private DNS zones for a private link

[Private Link][privatelink/overview] endpoints provide private connectivity to many Azure Platform-as-a-Service (PaaS) resources and it makes heavy use of DNS. It is a particular implementation of the split-brain DNS implementation described in the previous section, with the difference that the public resolution is provided automatically by Microsoft. When a user resolves the FQDN of an Azure PaaS resource that has any private endpoint configured, Microsoft's resolution will redirect the name resolution to a new zone name. For example, storage accounts with private endpoints will be redirected from "blob.core.windows.net" to "privatelink.blob.core.windows.net". You can find the zone name for each service in [Azure Private Endpoint private DNS zone values][privatelink/dnszones].

This mechanism allows you to control name resolution by deploying an Azure private DNS zone for that "privatelink" domain. Users with access to domain resolution by the private zone will get the Azure PaaS FQDN resolved to a private IP address. Otherwise, Microsoft also provides public resolution for the "privatelink" domains, and the PaaS service's public IP address will be provided to any users without access to the private DNS zone.

The interaction between private link and DNS can be complex, you can find further details in [Azure Private Endpoint DNS integration Scenarios][privatelink/dnsintegration]

### Enable autoregistration

When you configure a virtual network link with a private DNS zone, you can optionally configure autoregistration [autoregistration for all the virtual machines][dns/autoregistration]. Virtual machines created in virtual networks linked to a private DNS zone with autorregistration will automatically create DNS A records in that zone, so that other systems in Azure and on-premises can resolve their names.

Azure Private DNS zone autoregistration is especially useful for Linux virtual machines, since Linux doesn't provide out-of-the-box mechanisms for auto-registering in a DNS server. However, if you have Windows virtual machines joined with Active Directory Domain Services, name resolution for those VMs will be automatically provided by the domain controllers.

## Considerations

These considerations implement the pillars of the Azure Well-Architected Framework, which is a set of guiding tenets that can be used to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework][waf].

### Scalability

- If you use Azure DNS Private Resolver, verify that your architecture will stay within the [Azure DNS Private Resolver documented limits][dns/resolver/limits].
- If you use virtual machines as DNS servers instead of Azure DNS Private Resolver, consider using at least two DNS servers distributed across availability zones for better resiliency and scalability.

### Availability

- Deploy all Azure resources across availability zones.
- If using virtual machines as DNS servers, you should have at least two DNS servers spread over availability zones. You can either use an [Azure Load Balancer][alb/overview] to provide a single IP address to your DNS clients, or configure all IP addresses in your DNS clients. An Azure Load Balancer provides simplified client configuration and easier migrations, but can generate additional costs.
- DNS servers should be placed close to the users and systems that need access to them. You should consider providing DNS resolution in each Azure region.

### Manageability

- Use Azure DNS Private Resolver instead of custom DNS server software such as [BIND][bind] to lower management overhead costs.
- Use static IP address for Azure DNS Private Resolver's inbound and outbound endpoints, so that they are predictable and stay constant across deployments.
- Implement integration between private endpoints and private DNS zones using Azure Policy. You can find more information about this in [Private Link and DNS integration at scale][dns/azpolicy].
- Configure DNS logging in the Azure Firewall diagnostic settings. See [AZFWDNSQuery][azfw/dnslogs] for details about the format of Azure Firewall DNS logs.
- Configure [Azure DNS Security Policies][dns/securitypolicy] with diagnostic settings for logging at the Azure DNS level.

### Security

- Consider [protecting private DNS zones and records][dns/protect]. You should restrict administrative access to DNS, since DNS disruptions caused by misconfigurations such as accidental deletion of private zones can potentially have a broad impact radius.
- If using public DNS, consider using [DNSSEC][dns/dnssec].
- Configure [Azure DNS Security Policies][dns/securitypolicy] for additional security in Azure DNS.
- See the [Azure security baseline for Azure DNS][dns/securitybaseline] for further details.

### DevOps

- Automate configuration of this architecture by with Azure Resource Manager (ARM), bicep or Terraform templates for configuration of all the resources. If you prefer using scripts or Software Development Kits (SDKs), both private and public DNS zones support full management from Azure CLI, PowerShell, .NET, and REST API.
- If you're using a continuous integration and continuous development (CI/CD) pipeline to deploy and maintain workloads in Azure and on-premises, you can also configure autoregistration of DNS records.

### Cost Optimization

Cost Optimization is about looking at ways to reduce unnecessary expenses and improve operational efficiencies. For more information, see [Design review checklist for Cost Optimization][waf/cost].

- Azure DNS zone costs are based on the number of DNS zones hosted in Azure and the number of received DNS queries.
- The cost of the Azure DNS Private Resolver is based on the number of inbound and outbound endpoints and the forwarding rulesets.
- If you configure your Azure DNS Private resolver or your DNS servers in your hub virtual network, DNS resolutions will traverse fewer VNet peerings and this traffic will be cheaper. However, consider that DNS requests are usually not bandwidth-intensive, so the cost savings you can achieve with this optimization are limited. Besides, this integration is not possible if you are using Virtual WAN or if your DNS servers should be located in a different subscription.
- Use the [Azure pricing calculator][calculator] to estimate costs. Pricing models for Azure DNS, Azure Private DNS and Azure DNS Private Resolver is documented in [Azure DNS Pricing][dns/pricing].


## Next steps

Learn more about the component technologies:

- [What is Azure DNS?][dns/overview]
- [What is Azure DNS Private Resolver?][dns/resolver/overview]
- [What is Azure Virtual Network?][vnet/overview]

## Related resource

[Azure enterprise cloud file share](./azure-files-private.yml)

[architectural-diagram-visio-source]: https://arch-center.azureedge.net/hybrid-dns-infra.vsdx
[waf]: /azure/well-architected/
[waf/cost]: /azure/well-architected/cost-optimization/checklist
[alz/identity]: /azure/cloud-adoption-framework/ready/landing-zone/design-area/identity-access-landing-zones
[bind]: https://www.isc.org/bind/
[expressroute/overview]: /azure/expressroute/expressroute-introduction
[vpn/design]: /azure/vpn-gateway/design
[dns/resolver/overview]: /azure/dns/dns-private-resolver-overview
[dns/resolver/limits]: /azure/azure-resource-manager/management/azure-subscription-service-limits#dns-private-resolver1
[dns/overview]: /azure/dns/private-dns-overview
[dns/autoregistration]: /azure/dns/private-dns-autoregistration
[dns/pricing]: https://azure.microsoft.com/pricing/details/dns/
[dns/protect]: /azure/dns/dns-protect-private-zones-recordsets
[dns/dnssec]: /azure/dns/dnssec
[dns/securitybaseline]: /azure/baselines/azure-dns-security-baseline
[dns/securitypolicy]: azure/dns/dns-security-policy
[dns/azpolicy]: /azure/cloud-adoption-framework/ready/azure-best-practices/private-link-and-dns-integration-at-scale
[privatelink/dnsintegration]: /azure/private-link/private-endpoint-dns-integration
[privatelink/overview]: /azure/private-link/private-link-overview
[privatelink/dnszones]: /azure/private-link/private-endpoint-dns
[azfw/overview]: /azure/firewall/overview
[azfw/dns]: /azure/firewall/dns-details
[azfw/dnslogs]: /azure/azure-monitor/reference/tables/azfwdnsquery
[vnet/overview]: /azure/virtual-network/virtual-networks-overview
[vnet/waf-sg]: /azure/well-architected/service-guides/virtual-network
[vnet/customdns]: /azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances
[vnet/peering]: /azure/virtual-network/virtual-network-peering-overview
[alb/overview]: /azure/load-balancer/load-balancer-overview
[vwan/overview]: /azure/virtual-wan/virtual-wan-about
[vmss/overview]: /azure/virtual-machine-scale-sets/overview
[vm/overview]: /virtual-machines/overview
[vpngw/about]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[ergw/about]: /azure/expressroute/expressroute-about-virtual-network-gateways
[er/overview]: /azure/expressroute/expressroute-introduction
[calculator]: https://azure.microsoft.com/pricing/calculator/
