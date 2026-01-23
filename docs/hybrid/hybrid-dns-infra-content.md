This reference architecture illustrates how to design a hybrid Domain Name System (DNS) solution to resolve names for workloads that are hosted on-premises and in Microsoft Azure.

## Architecture

:::image type="content" source="./images/hybrid-dns-infra.png" alt-text="Diagram showing a Hybrid Domain Name System (DNS)." border="false" lightbox="./images/hybrid-dns-infra.svg" :::

### Workflow

The architecture consists of the following components:

- **On-premises network**. The on-premises network represents a single datacenter that's connected to Azure over an [Azure ExpressRoute][expressroute/overview] or [virtual private network (VPN)][vpn/design] connection. In this scenario, the following components make up the on-premises network:
  - **DNS servers**. These servers represent one or more servers with DNS a service installed that are acting as name resolver for on-premises systems. These servers must be configured with conditional forwarding rules to send DNS requests for Azure systems or private endpoints to Azure DNS Private Resolver's inbound endpoint. The IP addresses of the on-premises DNS servers will be referenced in Azure DNS forwarding rulesets for domains hosted on-premises.
  - **On-premises gateway**. The on-premises gateway represents either a VPN termination device or a router connected to Azure ExpressRoute that provides private connectivity to the Azure environment.
- **Connectivity subscription**. The connectivity subscription represents an Azure subscription that's used for resources that provide connectivity to Azure-hosted workloads.
> [!NOTE]
> If you use Active Directory Domain Controllers (ADDC) as DNS servers, these resources are considered to be more related to identity than to connectivity, and will therefore be usually deployed in the identity subscription as described in [Landing zone identity and access management][alz/identity].
  - **Hub virtual network**. The hub [virtual network][vnet/overview] contains connectivity resources such as Azure Virtual Network Gateways, Azure Firewall, Software-Defined WAN Network Virtual Appliances (NVA) and firewall NVAs.
    - **Gateway subnet**. The gateway subnet hosts the Azure VPN or ExpressRoute gateway that's used to provide connectivity back to the on-premises datacenter. See [Gateway subnet] for recommendations about the size of the Gateway subnet.
    - **Azure Firewall subnet**. The Azure Firewall subnet hosts the Azure Firewall to filter traffic and provide DNS proxy services. Its network mask should be /26, for more details see [Why does Azure Firewall need a /26 subnet size?][azfw/subnet].
  > [!NOTE]
  > The hub virtual network can be substituted with a [virtual wide area network (VWAN)][vwan/overview] hub. In both cases, the recommended best practice is hosting Azure DNS Private Resolver or your DNS servers in a different virtual network (VNet).
  - **Shared services virtual network**. The shared services virtual network hosts resources that additional services to workload subscriptions, such as DNS servers. If using a self-managed hub-and-spoke architecture you can move shared services to the hub vitual network to reduce the number of virtual network peering hops between workloads and shared services, but extra care should be taken with user-defined routes in the spokes to avoid asymmetric routing when firewalls are involved (see [Hub virtual network workload][azfw/hubworkloads] for more details).
    - **Virtual network peering**. This component is a [peering][vnet/peering] connection back to the hub VNet. This connection allows connectivity to the rest of the environment. It needs to be configured to use the hub's virtual network gateway (VPN or ExpressRoute) so that the workload IP prefixes are advertised to on-premises via VPN or ExpressRoute.
    - **Inbound endpoint subnet**. DNS requests to Azure Private Resolver must be sent to the IP address of its inbound endpoint. This address will be configured in the forwarding rules of the on-premises DNS servers and as DNS server for Azure Firewall.
    - **Outbound endpoint subnet**. When Azure DNS Private Resolver needs to forward DNS requests to external servers based on forwarding rules, those requests will be sourced from this subnet. The minimum size for inbound and outbound endpoint subnets is /28, but in this architecture we configured /26 for further flexibility in case the limits change. You can find more information in [Subnet restrictions][dns/resolver/subnets].
    - **Forwarding rulesets**. Rules that define which domains will be forwarded to which external DNS servers. These rules should contain all the name domains hosted on-premises and the IP addresses of the on-premises DNS servers as forwarding targets. The diagram above reflects the [centralized DNS architecture][dns/resolver/architecture] for external name resolution with Azure DNS Private Resolver, which requires that the forwarding ruleset is linked to the virtual network where the private resolver is deployed (the shared services virtual network in this architecture).
> [!NOTE]
> If the DNS servers are going to be Active Directory Domain Controllers, these would typically be deployed in an identity subscription. Consequently, they would be in their own virtual network, since virtual networks cannot span multiple subscriptions. This architecture guide focuses on using Azure DNS Private Resolver.
- **Workload subscription**. The connected subscription represents a collection of workloads that require a virtual network and connectivity back to the on-premises network.
  - **Virtual network peering**. This component is a [peering][vnet/peering] connection back to the hub VNet. This connection allows connectivity from the spoke virtual network to other resources reachable from the hub, such as the on-premises network or the DNS Private Resolver. It needs to be configured to use the hub's virtual network gateway (VPN or ExpressRoute) so that the workload IP prefixes are advertised to on-premises via VPN or ExpressRoute.
  - **Workload subnet**. The default subnet contains a sample workload.
    - **Workload**. In this example a [virtual machine scale set][vmss/overview] or a collection of individual [virtual machines][vm/overview] hosts a workload in Azure that can be accessed from on-premises, Azure, and the public internet.

## Components

- [Virtual Network][vnet/waf-sg]. Azure Virtual Networks (VNet) are the fundamental building blocks for your private network in Azure. VNets can provide connectivity to many types of Azure resources, such as Azure Virtual Machines (VM), to securely communicate with each other, the internet, and on-premises networks.
- [VPN Gateway][vpngw/about]. A VPN Gateway sends encrypted traffic between an Azure virtual network and an on-premises location over the public Internet. You can also use VPN Gateway to send encrypted traffic between Azure virtual networks over the Microsoft network. A VPN gateway is a specific type of virtual network gateway.
- [ExpressRoute Gateway][ergw/about]. An ExpressRoute Gateway connects a virtual network with an ExpressRoute circuit, which is a dedicated line with guaranteed bandwidth between Microsoft and your on-premises environment. An ExpressRoute gateway is a specific type of virtual network gateway.
- [Azure Firewall][azfw/overview]. An Azure Firewall inspects network traffic and only allows legitimate flows. Azure Firewall can be configured as DNS proxy, which enables Fully-Qualified Domain Name (FQDN) network rules and DNS logging. See [Azure Firewall DNS Proxy Details[azfw/dns] for more information about this functionality.
- [Azure Private DNS Zone][dns/overview]. Azure Private DNS Zone provide DNS resolution for Azure workloads. Virtual machines can be autorregistered and integration with private link endpoints can be automated. Azure workloads can use private DNS zones if they are located in a virtual network linked to a specific zone by means of a DNS virtual network link.
- [Azure DNS Private Resolver][dns/resolver/overview]. Azure DNS Private Resolver is a managed service that provides DNS resolution in Azure, including conditional forwarding of DNS requests to other DNS servers. Since it is a Microsoft-managed service, Azure administrators do not need to manage the operating system and can focus on the configuration of DNS functionality.
- [DNS forwarding ruleset][dns/resolver/rulesets]. DNS forwarding rulesets contain definitions of which name domains should be forwarded to which external DNS server. Forwarding rulesets can be linked to virtual networks to provide external DNS resolution.
> [!NOTE]
> If you already have DNS servers such as Windows Server virtual machines acting as Active Directory Domain Controllers (ADDC) you would not need Azure DNS Private Resolver, and your ADDCs would replace Azure DNS Private Resolver and the DNS forwarding rulesets in this architecture.

## Alternatives

The topology described in this article can be modified and be adapted to your specific requirements. This section illustrates some of the most common variations:

- Use [Virtual WAN][vwan/overview] instead of a self-managed hub-and-spoke solution. In this case the hub virtual network is replaced by a virtual hub. A virtual hub is a Microsoft-managed virtual network where only specific resource types are supported: firewalls (Azure Firewall and third-party), virtual network gateways for VPN and ExpressRoute, and SDWAN network virtual appliances (NVA), please see [Third-party integrations with Virtual WAN Hub][vwan/nva] for more details.
- Consolidate the shared resources and the hub virtual networks. You can deploy your Azure DNS Private Resolver, DNS servers or any other shared resources such as Azure Bastion or files shares inside of the hub virtual network. This approach only works if not using virtual WAN. If you also have an Azure Firewall in your hub virtual network you need to be careful with routing from the spoke virtual networks to the shared services subnet, and configure only the shared services subnet prefix (and not the whole hub virtual network range) in your user-defined routes (UDR). You can find more information in [Hub virtual network workloads][azfw/hubworkloads].
- You can use custom DNS server software such as [BIND][bind] running on virtual machines instead of Azure DNS Private Resolver. Although this option incurs in more management overhead, it offers more flexibility. For example, you can use Active Directory Domain Controllers as DNS servers extending your on-premises Active Directory domain to Azure, or if your organization is already familiar with open-source servers such as BIND you can leverage the same technology in Azure as well.

## Hybrid resolution flows

This section explains how hybrid resolution flows work. There are two main cases to be explored:
- An Azure workload tries to resolve a domain name for a system hosted on-premises.
- An on-premises workload tries to resolve a domain name for a system hosted in Azure.

### Azure to on-premises

Azure virtual machines might need to access on-premises systems such as database or monitoring applications and they would have to be able to resolve name domains for which the authoritative servers are the on-premises DNS servers. There are different scenarios for this flow:

1. The workload virtual networks are configured to use Azure Firewall as custom DNS server (see [Specify DNS Servers][vnet/customdns] for more information about custom DNS servers in Azure).
2. Azure Firewall is configured to used Azure DNS Private Resolver's inbound endpoint as DNS server
3. If Azure DNS Private Resolver finds a match in the rulesets associated to its outbound endpoints, it will forward the DNS request to the target specified in the rule, which should be the on-premises DNS servers.
4. One of the on-premises DNS servers resolves the DNS query.

The model described above of using the inbound endpoint's IP address as custom DNS server is referred to as "Centralized DNS architecture" in [Private Resolver architecture][dns/resolver/architecture]. Azure DNS Private Resolver also offers external DNS resolution by linking DNS forwarding rulesets to virtual networks, which is called "Distributed DNS architecture". If you link the forwarding ruleset to the hub virtual network, Azure Firewall's DNS server should be configured as Azure DNS, represented in Azure by the IP address 168.63.129.16.

### On-premises

On-premises systems might need name resolution for workloads deployed in Azure or for private endpoints of Azure PaaS services. This name resolution will follow this workflow:

1. The on-premises workload will send a DNS request to the on-premises DNS server.
2. The on-premises DNS server will look into its configured forwarding rules, which should have Azure Firewall configured as DNS destination.
3. Azure Firewall will forward the DNS request to its DNS server, the private resolver's inbound enpoint IP address.
4. Azure DNS Private Resolver will resolve the DNS query for any private DNS zone linked to the virtual network where it has been deployed.

## Recommendations

The following recommendations apply for most scenarios. Follow these recommendations unless you have a specific requirement that overrides them.

### Extend AD DS to Azure (optional)

If your organization uses Active Directory integrated DNS zones for name resolution, you should extend your Active Directory domain to Azure. This implies having an additional set of Active Directory Domain Controllers running in an Azure virtual network, ideally in the identity subscription as described above.

### Configure split-brain DNS

If you have applications that should be accessed internally and externally over the public Internet depending on where users are located, you can configure split-brain DNS so that users don't need to use different host names depending on their location. Split-brain consists in offering different name resolution for the same fully-qualified domain name (FQDN) to on-premises and Internet users. Internet users would resolve the application FQDN using publicly available Azure DNS zones, while DNS requests from internal users would be directed to Azure DNS Private Resolver using the mechanism described in this artcle.

### Integrate your private endpoints with private DNS zones

[Private Link][privatelink/overview] endpoints provide private connectivity to many Azure Platform-as-a-Service (PaaS) resources. Private link uses a particular implementation of the split-brain DNS implementation described in the previous section, with the difference that the public resolution is provided automatically by Microsoft. When a user resolves the FQDN of an Azure PaaS resource that has any private endpoint configured, Microsoft will redirect the name resolution to a new zone name. For example, storage accounts with private endpoints will be redirected from "blob.core.windows.net" to "privatelink.blob.core.windows.net". You can find the zone name for each service in [Azure Private Endpoint private DNS zone values][privatelink/dnszones].

This mechanism allows you to control name resolution by deploying an Azure private DNS zone for that "privatelink" domain. Users with access to domain resolution by the private zone will get the Azure PaaS FQDN resolved to a private IP address. Otherwise, Microsoft also provides public resolution for the "privatelink" domains, and the PaaS service's public IP address will be provided to any users without access to the private DNS zone.

The interaction between private link and DNS can be quite complex, you can find further details in [Azure Private Endpoint DNS integration Scenarios][privatelink/dnsintegration].

### Enable autoregistration

When you configure a virtual network link with a private DNS zone, you can optionally enable autoregistration [autoregistration for all the virtual machines][dns/autoregistration]. With DNS private zone autoregistration, new virtual machines will automatically create DNS records in the private DNS zone, so that other systems in Azure and on-premises can resolve their names.

Azure Private DNS zone autoregistration is especially useful for Linux virtual machines, since Linux doesn't provide out-of-the-box mechanisms for auto-registering their IP addresses in a DNS server. In on-premises environment Dynamic Host Configuration Protocol (DHCP) servers are often used to provide this automatic registration of all systems in DNS, but DHCP does not work in Azure virtual networks.

## Considerations

These considerations implement the pillars of the Azure Well-Architected Framework, which is a set of guiding tenets that can be used to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework][waf].

### Scalability and availability

- If you use Azure DNS Private Resolver, verify that your architecture will stay within the [Azure DNS Private Resolver documented limits][dns/resolver/limits].
- Deploy all Azure resources across availability zones. If you use virtual machines as DNS servers instead of Azure DNS Private Resolver, consider using at least two DNS servers distributed across availability zones for better resiliency and scalability.
- If using virtual machines as DNS servers, you should deploy at least two. You can either use an [Azure Load Balancer][alb/overview] to provide a single IP address to your DNS clients, or configure all DNS server IP addresses in your DNS clients. An Azure Load Balancer provides simplified client configuration and easier migrations, but it can generate additional costs.
- DNS servers should be placed close to the users and systems that need access to them. You should consider providing DNS resolution in each Azure region.

### Operational excellence

- Use Azure DNS Private Resolver instead of custom DNS server software such as [BIND][bind] if possible to reduce management overhead.
- Use static IP address for Azure DNS Private Resolver's inbound and outbound endpoints, so that they are predictable and stay consistent across deployments.
- Implement integration between private endpoints and private DNS zones using Azure Policy. You can find more information about this in [Private Link and DNS integration at scale][dns/azpolicy].
- Configure DNS logging in the Azure Firewall diagnostic settings. See [AZFWDNSQuery][azfw/dnslogs] for details about the format of Azure Firewall DNS logs.
- Configure [Azure DNS Security Policies][dns/securitypolicy] with diagnostic settings for logging at the Azure DNS level.

### Security

- Consider [protecting private DNS zones and records][dns/protect]. You should restrict administrative access to DNS, since DNS disruptions caused by misconfigurations such as accidental deletion of private zones can have a very broad impact radius.
- If using public DNS, consider using [DNSSEC][dns/dnssec].
- Configure [Azure DNS Security Policies][dns/securitypolicy] for additional security in Azure DNS.
- See the [Azure security baseline for Azure DNS][dns/securitybaseline] for further details.

### Cost efficiency

- If you configure your Azure DNS Private resolver or your DNS servers in your hub virtual network, DNS resolutions will traverse fewer VNet peerings and this traffic will be cheaper. However, consider that DNS requests are usually not bandwidth-intensive, so the cost savings you can achieve with this optimization are limited. Furthermore, this integration is not possible if you are using Virtual WAN or if your DNS servers should be located in a different subscription.
- Use the [Azure pricing calculator][calculator] to estimate costs. Pricing models for Azure DNS, Azure Private DNS and Azure DNS Private Resolver is documented in [Azure DNS Pricing][dns/pricing].


## Next steps

Learn more about the component technologies:

- [What is Azure DNS?][dns/overview]
- [What is Azure DNS Private Resolver?][dns/resolver/overview]
- [What is Azure Virtual Network?][vnet/overview]

[architectural-diagram-visio-source]: https://arch-center.azureedge.net/hybrid-dns-infra.vsdx
[waf]: /azure/well-architected/
[waf/cost]: /azure/well-architected/cost-optimization/checklist
[alz/identity]: /azure/cloud-adoption-framework/ready/landing-zone/design-area/identity-access-landing-zones
[bind]: https://www.isc.org/bind/
[expressroute/overview]: /azure/expressroute/expressroute-introduction
[vpn/design]: /azure/vpn-gateway/design
[dns/resolver/overview]: /azure/dns/dns-private-resolver-overview
[dns/resolver/limits]: /azure/azure-resource-manager/management/azure-subscription-service-limits#dns-private-resolver1
[dns/resolver/rulesets]: /azure/dns/private-resolver-endpoints-rulesets#dns-forwarding-rulesets
[dns/resolver/architecture]: /azure/dns/private-resolver-architecture
[dns/resolver/subnets]: /azure/dns/dns-private-resolver-overview#subnet-restrictions
[dns/overview]: /azure/dns/private-dns-overview
[dns/autoregistration]: /azure/dns/private-dns-autoregistration
[dns/pricing]: https://azure.microsoft.com/pricing/details/dns/
[dns/protect]: /azure/dns/dns-protect-private-zones-recordsets
[dns/dnssec]: /azure/dns/dnssec
[dns/securitybaseline]: /security/benchmark/azure/baselines/azure-dns-security-baseline
[dns/securitypolicy]: /azure/dns/dns-security-policy
[dns/azpolicy]: /azure/cloud-adoption-framework/ready/azure-best-practices/private-link-and-dns-integration-at-scale
[privatelink/dnsintegration]: /azure/private-link/private-endpoint-dns-integration
[privatelink/overview]: /azure/private-link/private-link-overview
[privatelink/dnszones]: /azure/private-link/private-endpoint-dns
[azfw/overview]: /azure/firewall/overview
[azfw/dns]: /azure/firewall/dns-details
[azfw/dnslogs]: /azure/azure-monitor/reference/tables/azfwdnsquery
[azfw/hubworkloads]: /azure/firewall/firewall-multi-hub-spoke#hub-virtual-network-workloads
[azfw/subnet]: /azure/firewall/firewall-faq#why-does-azure-firewall-need-a--26-subnet-size
[vnet/overview]: /azure/virtual-network/virtual-networks-overview
[vnet/waf-sg]: /azure/well-architected/service-guides/virtual-network
[vnet/customdns]: /azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances
[vnet/peering]: /azure/virtual-network/virtual-network-peering-overview
[alb/overview]: /azure/load-balancer/load-balancer-overview
[vwan/overview]: /azure/virtual-wan/virtual-wan-about
[vwan/nva]: /azure/virtual-wan/third-party-integrations
[vmss/overview]: /azure/virtual-machine-scale-sets/overview
[vm/overview]: /azure/virtual-machines/overview
[vpngw/about]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[vpngw/subnet]: /azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#gwsub
[ergw/about]: /azure/expressroute/expressroute-about-virtual-network-gateways
[er/overview]: /azure/expressroute/expressroute-introduction
[calculator]: https://azure.microsoft.com/pricing/calculator/
