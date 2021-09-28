---
author: Liam Björkman
tags: azure network PaaS
---

The following scenario is based on an application that we want to migrate to the cloud, but the application handles sensitive information and is integrated with internal servers and resources. Because of this our CTO does not agree with the PaaS solution since it would be sending the data through the Internet.  As a part of this modernization we also want to be able to implement Azure Service Bus as a message broker for our business data, again the CTO does not agree since it would expose messages to the Internet.

To overcome this we will need to convince our CTO that by using a Virtual Private Cloud architecture and Azure Private Link for connectivity.



## What is Virtual Private Cloud?

A VPC is a public cloud offering that lets an enterprise establish its own private cloud-like computing environment on shared [public cloud](https://www.ibm.com/cloud/public) infrastructure. A VPC gives an enterprise the ability to define and control a virtual network that is logically isolated from all other public cloud tenants, creating a private, secure place on the public cloud.

A VPC’s logical isolation is implemented using virtual network functions and security features that give an enterprise customer granular control over which IP addresses or applications can access particular resources. It is analogous to the “friends-only” or “public/private” controls on social media accounts used to restrict who can or can’t see your otherwise public posts.



### Features

- **Agility:** Control the size of your virtual network and deploy cloud resources whenever your business needs them. You can scale these resources dynamically and in real-time.
- **Availability:** Redundant resources and highly fault-tolerant availability zone architectures mean your applications and workloads are highly available.
- **Security:** Because the VPC is a logically isolated network, your data and applications won’t share space or mix with those of the cloud provider’s other customers. You have full control over how resources and workloads are accessed, and by whom.
- **Affordability:** VPC customers can take advantage of the public cloud’s cost-effectiveness, such as saving on hardware costs, labor times, and other resources.



Example of how a VPC can integrate with a on-premise network to extend the company infrastructure.

<img src="/img/vpc1.png">





## What is Azure Private Link?

Azure Private Link enables you to access Azure PaaS Services (for example, Azure Storage and SQL Database) and Azure hosted customer-owned/partner services over a [private endpoint](https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview) in your virtual network.

**Private Endpoint** - A private IP from the VPC that connects privately and securely to a service such as a VM or Database through Azure Private Link.

Traffic between your virtual network and the service travels the Microsoft backbone network, a secure network that is not exposed to the Internet. It operates within the Azure data centers. The same network is used for Microsoft services such as Microsoft 365 and Azure Services.  

### Features

Azure Private Link provides the following benefits:

- **Privately access services on the Azure platform**: Connect your virtual network to services in Azure without a public IP address at the source or destination. Service providers can render their services in their own virtual network and consumers can access those services in their local virtual network. The Private Link platform will handle the connectivity between the consumer and services over the Azure backbone network.
- **On-premises and peered networks**: Access services running in Azure from on-premises over ExpressRoute private peering, VPN tunnels, and peered virtual networks using private endpoints. There's no need to configure ExpressRoute Microsoft peering or traverse the internet to reach the service. Private Link provides a secure way to migrate workloads to Azure.
- **Protection against data leakage**: A private endpoint is mapped to an instance of a PaaS resource instead of the entire service. Consumers can only connect to the specific resource. Access to any other resource in the service is blocked. This mechanism provides protection against data leakage risks.
- **Global reach**: Connect privately to services running in other regions. The consumer's virtual network could be in region A and it can connect to services behind Private Link in region B.
- **Extend to your own services**: Enable the same experience and functionality to render your service privately to consumers in Azure. By placing your service behind a standard Azure Load Balancer, you can enable it for Private Link. The consumer can then connect directly to your service using a private endpoint in their own virtual network. You can manage the connection requests using an approval call flow. Azure Private Link works for consumers and services belonging to different Azure Active Directory tenants.

<img src="/img/vpc2.png">

