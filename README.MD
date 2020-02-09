## Azure Private Link Hybrid Design Architecture

### Purpose

The purpose of this document is to share my experience with the Azure Private Link service and the resulting design outcome that blends itself with a common Virtual Network design pattern known as hub-and-spoke.

If you are not familiar the hub-and-spoke design pattern there is great information on this and other design patterns at the URL below.

- https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke

Note that the concepts in this article will apply to many other virtual network design patterns, hub-and-spoke is used in the example due to its pervasiveness.

### What is Private Link, why use it and how does it work?

Private Link establishes a private endpoint to an applicable Platform Service (PaaS). For example, when you provision an Azure SQL Database, it has a globally unique, publically accessible, fully qualified domain name (FQDN) such as app1.database.windows.net. This FQDN resolves to a public IPv4 IP address that is maintained by Microsoft. With Private Link enabled the same Azure SQL Database will resolve to an IP address from an Address Space assigned to your VNet. 

For DNS nerds, think of Private Link as split-brain DNS.

You use Private Link to access applicable Azure PaaS services via a private endpoint from within Azure, or on premesis using, and this is important, the SAME hostname as you would if accessing the service via the public Internet. The latter is attractive if you want to access a PaaS service over a hybrid WAN connection such as Express Route or a site-to-site VPN.

It is IMPORTANT to understand that the enablement of Private Link on a PaaS service will establish three things;

1. A Private Endpoint (maps service to NIC, Unique ID, approval workflows, etc...)
2. A Network Interface Card (e.g. NIC)
3. A FLZ called privatelink + the service FQDN (e.g Azure Private DNS Zone). For example, privatelink.database.windows.net

Think about Resource Group placement and naming conventions of these services before provisioning the service.

Please see official Microsoft documentation for a formal overview on the Private Link Services and the benefits it offers at the URL below.

- https://docs.microsoft.com/en-us/azure/private-link/private-link-overview

### How to enable Private Link

You won't find any details here. For how-to please see our step by step documentation at the URL below.

- https://docs.microsoft.com/en-us/azure/private-link/create-private-endpoint-portal

### The Design Architecture

Here are the assumptions;

1. You need to access the PaaS service privately from your corporate network
2. Private Link is enabled on the applicable PaaS Service
3. You have Express route or a site-to-site VPN
4. You have a DNS Server running in Azure IaaS

### Architecture Diagram

![GitHub Logo](/PrivateLink.jpg)

Remember, the design objective for Private Link is to access a PaaS service privately using the SAME hostname. 

In order for Private Link to work properly, DNS queries must be answered by the Azure DNS Resolver, 168.63.129.16. This IP is used for some specific capabilities in Azure and understanding its purpose is important understand it's applicability here. More information at this URL;

- https://docs.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16

### Configuration

#### Corporate DNS Server

The DNS Server(s) in your corporate network must forward queries for the applicable PaaS Server FLZ (e.g. database.windows.net) to your Azure IaaS DNS Server.

There is no other way to achieve the objective other than configuring local host files on your hosts (good for testing, poor for production). 

In the scenario, you must configure your corporate DNS Server(s) to conditionally forward queries destined for database.windows.net (or applicable PaaS FLZ) to the DNS server that is housed in Azure, in this illustration thats 10.0.0.4.

#### DNS Server in Azure

Your DNS server in Azure IaaS must forward queries to the Azure DNS Resolver (168.63.129.16). 

You can chose to forward all queries to the Azure DNS Resolver, or conditionally forward queries (e.g. database.windows.net) to the Azure DNS Resolver. It's up to you, both will work. Ask your DNS nerd if you have any standards to adhere to. 

Important: The Azure DNS Resolver must have a reference to the Azure Private DNS Zone (next section) that will have the privatelink.database.windows.net FLZ mappings. 

In the scenario, you must configure your Azure IaaS DNS Server to conditionally forward (or forward all) queries destined for database.windows.net (or applicable PaaS FLZ) to the Azure DNS Resolver (168.63.129.16).

#### Azure Private DNS Zone

An Azure Private DNS Zone is created when you provision your first Private Link service. While the creation of this Private DNS Zone is optional, without it, you will need to use Host Files for accessing Private Link enabeld PaaS services. Host Files = good for testing, bad for production. 

The Azure Private DNS Zone may be linked to your Virtual Networks. The Azure Private DNS Zone will be linked to the VNets in which you placed a Private Endpoint (e.g. NIC). 

Important: The Azure Private DNS Zone must also be linked to the VNet that has your Azure IaaS DNS Server.\

In the scenario, you must link the Azure Private DNS Zone (privatelink.database.windows.net) to the VNet that has your Azure IaaS DNS Server.

