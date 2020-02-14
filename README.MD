## Azure Private Link Hybrid Design Architecture

### Purpose

The purpose of this document is to share my experience with the Azure Private Link service and the resulting design outcome that blends itself with a common virtual network design pattern known as hub-and-spoke.

If you are not familiar the hub-and-spoke design pattern there is great information on this and other design patterns at the URL below.

- https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke

Note that the concepts in this article will apply to many other virtual network design patterns, hub-and-spoke is used in the example due to its pervasiness. 

### What is Private Link, why use it and how does it work?

Simply put, Private Link establishes a private endpoint on a supported Platform (Paas) Service. 

For example, when you provision an Azure SQL Database, it has a globally unique, publically accessible, fully qualified domain name (FQDN) such as app1.database.windows.net. This FQDN resolves to a public IPv4 IP address that is maintained by Microsoft. With Private Link enabled the same Azure SQL Database will resolve to a private IP address from an address space assigned to your VNet.

(For DNS nerds, think of Private Link as split-brain DNS. Private Link is all orchestrated through clever CNAME mechanisms Azure manages for you)

You use Private Link to access an Azure PaaS services via a private endpoint from within Azure, or your corporate networking using, and this is important, the SAME hostname as you are accessing the service via the Internet. Private Link also enables a PaaS service to be accessible over an Express Route (private peer) or site-to-site VPN.

Configuring Private Link on a PaaS service will establish three things;

1. A Private Endpoint (maps service to NIC, Unique ID, RBAC approval workflows, etc...)
2. A Network Interface Card (e.g. vNIC). The vNIC is placed in a VNet\Subnet
3. A Forward Lookup Zone (FLZ) called privatelink + the service FQDN (e.g Azure Private DNS Zone). For example, privatelink.database.windows.net

Consider the Resource Group placement, network and naming conventions of these services before provisioning Private Link. 

For elaboration please see official Microsoft documentation for a formal overview on the Private Link Services and the benefits it offers at the URL below.

- https://docs.microsoft.com/en-us/azure/private-link/private-link-overview

### How to enable Private Link

You won't find any step-by-step details here. For how-to please see our step by step documentation at the URL below. And yes, you can automate provisioning.

- https://docs.microsoft.com/en-us/azure/private-link/create-private-endpoint-portal

### The Design Architecture

Here are the assumptions;

1. You need to access the PaaS service privately from your corporate network
2. Private Link is enabled on the applicable PaaS Service
3. You have Express route or a site-to-site VPN
4. You have a DNS Server running in Azure IaaS

### Architecture Diagram

![GitHub Logo](/images/PrivateLink.jpg)

Remember, the design objective for Private Link is to access a PaaS service privately using the SAME hostname. 

In order for Private Link to work properly, DNS queries must be answered by the Azure DNS Resolver, 168.63.129.16. This IP is used for some specific capabilities in Azure and understanding its purpose is important understand it's applicability here. More information is available at the URL below.

- https://docs.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16

### Configuration

#### Corporate DNS Server

The DNS Server(s) in your corporate network must forward queries for the applicable FLZ (e.g. privatelink.database.windows.net) to your Azure IaaS DNS Server.

There is no other way to achieve the objective other than configuring local host files on your hosts (good for testing, poor for production). 

In the scenario, you must configure your corporate DNS Server(s) to conditionally forward queries destined for privatelink.database.windows.net (or applicable PaaS FLZ) to the DNS server that is housed in Azure, in this illustration thats 10.0.0.4.

#### DNS Server in Azure

Your DNS server in Azure IaaS must forward queries to the Azure DNS Resolver (168.63.129.16). 

You can chose to forward all queries to the Azure DNS Resolver, or conditionally forward queries (e.g. privatelink.database.windows.net) to the Azure DNS Resolver. It's up to you, both will work. Ask your DNS nerd if you have any standards to adhere to. 

Important: The Azure DNS Resolver must have a reference to the Azure Private DNS Zone (next section) that will have the privatelink.database.windows.net FLZ mappings. 

In the scenario, you must configure your Azure IaaS DNS Server to conditionally forward (or forward all) queries destined for privatelink.database.windows.net (or applicable PaaS FLZ) to the Azure DNS Resolver (168.63.129.16).

#### Speical note about Active Directory Integrated DNS Scenarios

If your DNS server in Azure is Active Directory Integrated, and you have configured conditional fowarding to also be AD integrated, you will not be able to create a conditional fowarder to the applicable FLZ on that server. instead you must either;

1. Configure the Azure IaaS DNS server to forward all queries to the Azure DNS Resolver or;

2. Build a stand-alone DNS Server that forwards queries to the Azure DNS Resolver. 
 
if you proceed with option 2, consider building two instances of your DNS Server in Azure for resliency using Availability Sets or Availability Zones, or simply deploy your Azure IaaS DNS (Stand-alone) server using modern DevOps and Infrastructure as Code practices and rapidly rebuild it upon failure. TTL and Caching should minimize a DNS server failure. Some automtaion to build a Windows DNS server is forthcoming.

Here is a basic template that deploys a Linux DNS server that fowards all traffic to the Azure DNS resolver. 

- https://github.com/Azure/azure-quickstart-templates/tree/master/301-dns-forwarder

#### Azure Private DNS Zone

An Azure Private DNS Zone is created when you provision your first Private Link service. While the creation of this Private DNS Zone is optional, without it, you will need to use Host Files for accessing Private Link enabeld PaaS services. Host Files = good for testing, bad for production. 

The Azure Private DNS Zone may be linked to your Virtual Networks. The Azure Private DNS Zone will be linked to the VNets in which you placed a Private Endpoint (e.g. NIC). 

Important: The Azure Private DNS Zone must also be linked to the VNet that has your Azure IaaS DNS Server.

In the scenario, you must link the Azure Private DNS Zone (privatelink.database.windows.net) to the VNet (10.0.0.0/24) that has your Azure IaaS DNS Server.

Here is the how-to for linking a private dns zone to your VNet (Don't need to enable auto-registration)

- https://docs.microsoft.com/en-us/azure/dns/private-dns-getstarted-portal#link-the-virtual-network

#### VNet Peering

Keep in mind that the VNet in which you place your vNIC must be accessible to your Express Route or site-to-site VPN otherwise the service will not be reachable.

There are numerous ways to interconnect VNets and that is not the purpose of this article. VNet peering is most common. For more information see the URL below.

- https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview

#### Testing

Once you have the above in place, Private Link should just work. If it's not working, you messed something up.

Good testing tools are things like;

- Host Files (good for testing, bad for production). Host files will verify you have proper connectivity and routing in place.

- NSLookup. NSLookup is handy for verifying that the PaaS FQDN resolves properly both internally and externally. Remember we want to use the SAME hostname for access

- DNS Cache. Be aware of DNS caching both locally on your device, and upstream DNS Servers. If you are getting improper results, flush your DNS cache where you can.

### Conclusion

I hope you find this article useful. 

Azure Private Link is a great new capability, but not without its complexities. Be sure you are using it to solve the right problem and do not use it for everything. Doing so will add cost, complexity and limit your agility. 