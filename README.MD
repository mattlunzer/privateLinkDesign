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

Because we all love pretty pictures, review the illustration below. Here are the assumptions;

1. You need to access the PaaS service privately from your corporate network
2. Private Link is enabled on the applicable PaaS Service
3. You have Express route or a site-to-site VPN
4. You have a DNS Server running in Azure IaaS

![GitHub Logo](/PrivateLink.jpg)

