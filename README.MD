## Azure Private Link Hybrid Design Architecture

## Purpose
The purpose of this document is to share my experience with the Azure Private Link service and the resulting design outcome that blends itself with a common Virtual Network design pattern known as hub-and-spoke.

If you are not familiar the hub-and-spoke design pattern there is great information on this and other design patterns at the URL below.

- https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke

Note that the concepts in this article will apply to many other virtual network design patterns, hub-and-spoke is used in the example due to its commonality.

## What is Private Link and why use it?
Essentially, you use Private Link to access applicable Azure Platform services via a private endpoint from within Azure, or on premesis using (and this is important) the SAME hostname as you would if accessing the service via the public Internet. Conceptually its much like a DNS Split-Brain model.

Please see official Microsoft documentation for a formal overview on the Private Link Services and the benefits it offers.

- https://docs.microsoft.com/en-us/azure/private-link/private-link-overview 

## How to enable Private Link

## The Design Architecture

![GitHub Logo](blob/master/privateLInk.jpg)