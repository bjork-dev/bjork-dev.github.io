---
author: Liam Björkman
tags: azure network PaaS
---

The following scenario is based on an application that we want to migrate to the cloud, but the application handles sensitive information and is integrated with internal servers and resources. Because of this our CTO does not agree with the PaaS solution since it would be sending the data through the Internet.  As a part of this modernization we also want to be able to implement Azure Service Bus as a message broker for our business data, again the CTO does not agree since it would expose messages to the Internet.

To overcome this we will need to convince our CTO that by using a Virtual Private Cloud architecture and Azure Private Link for connectivity.



## What is Virtual Private Cloud?

A VPC is a public cloud service that lets  you establish your own private cloud-like computing environment on shared public cloud infrastructure. A VPC gives you the ability to define and control a virtual network that is logically isolated from all other public cloud tenants, creating a private, secure place on the public cloud.

A VPC’s logical isolation is implemented using virtual network functions and security features that give a customer complete control over which IP addresses or applications can access particular resources.



### Features of VPC

- **Control:** Control the size of your virtual network and deploy cloud resources whenever your business needs them. You can scale these resources just like on a public cloud.
- **Availability:** Redundant resources and fault-tolerant availability zone architectures mean your applications and workloads are available.
- **Security:** Because the VPC is a logically isolated network, your data and applications won’t share space or mix with those of the cloud provider’s other customers. You have full control over how resources and workloads are accessed, and by whom.
- **Affordability:** VPC customers can take advantage of the public cloud’s cost-effectiveness, such as saving on hardware costs, low latency, and other resources.



Example of how a VPC can integrate with a on-premise network to extend the company infrastructure.

<img src="/img/vpc1.png">





## What is Azure Private Link?

Azure Private Link enables you to access PaaS Services (for example, Azure Storage and SQL Database) and Azure hosted services over a private endpoint in your virtual network (Azure's version of VPC).

**Private Endpoint** - A private IP from the VPC that connects privately and securely to a service such as a VM or Database through Azure Private Link.

Traffic between your virtual network and the service travels the Microsoft backbone network (Microsoft Global Network), a secure network that is not exposed to the Internet. It operates within the Azure data centers. The same network is used for Microsoft services such as Microsoft 365 and Azure Services, offering lower latency and faster handling than traversing across the Internet. 

### Features

Azure Private Link provides the following benefits:

- **Privately access services on the Azure platform**: Connect your VPC to services in Azure without a public IP address at the source or destination. Service providers can render their services in their own virtual network and consumers can access those services in their local virtual network. The Private Link platform will handle the connectivity between the consumer and services over the Microsoft Global Network.
- **On-premises and peered networks**: Access services running in Azure from on-premises over VPN tunnels using private endpoints. Private Link provides a secure way to reach private endpoints without connecting over the Internet.
- **Protection against data leakage**: A private endpoint is mapped to an instance of a PaaS resource instead of the entire service. Consumers can only connect to the specific resource. Access to any other resource in the service is blocked. This mechanism provides protection against data leakage risks.
- **Global reach**: Connect privately to services running in other regions. The consumer's virtual network could be in region A and it can connect to services behind Private Link in region B.

Example of how a consumer can connect to a VPC through a VPN and then use that to connect to another VPC hosting sensitive resources through Azure Private Link without traversing the Internet. 

**Azure EXPRESSROUTE** - VPN that connects directly via Microsoft Global Network to Azure- or Microsoft 365-services.

<img src="/img/vpc2.png">



## Summary

By implementing these strategies we do not expose any of our resources to the Internet just like if we were hosting everything ourselves on-premise. By implementing a cloud solution, we can significantly reduce costs while improving general performance and opening a door for seamless expansion and new implementations. 

One such implementation could be Azure Service Bus, that can help us improve performance and availability of our application by using technologies such as Message Queueing with load balancing and atomic transactions. We can use our private network to configure our Service Bus to either work only between certain privateIP-addresses, private endpoints or between Virtual Networks. No traffic is sent through the Internet.



##### Sources

##### [Service Bus Messaging](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview)

##### [Azure ExpressRoute](https://docs.microsoft.com/en-ca/azure/expressroute/expressroute-introduction)

##### [Virtual Private Cloud](https://www.cloudflare.com/learning/cloud/what-is-a-virtual-private-cloud/)

##### [Azure Private Link](https://azure.microsoft.com/sv-se/services/private-link/#how-it-works)

##### [Service Bus Endpoints](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-service-endpoints)



