---
author: Liam Björkman
tags: cloud example
---

The cloud has steadily evolved over the last year and will remain doing so for a long time to come. For a non-techy person the cloud is probably mostly storage, such as Dropbox or Onedrive. For people with more experience in the infrastructural or development part, the cloud offers solutions for any environment and/or business. 



A few key categories that can define the usage of cloud services:

### Infrastructure

* Host a server without having the hardware inhouse, instead pay for the resources you use.
* Seamlessly scale your resource(s) up or down depending on usage scenarios.
* Complete administrative controls from a single endpoint.



To summarize the infrastructural side:  

By using a cloud service you can seamlessly select hardware and software to work with each other, such as hosting a virtual machine or database, from the comfort of a single page. If demand for requests to the database goes up for example, adding more resources (scaling up) such as vCPUs and RAM is just a few clicks and a higher monthly invoice. Most cloud providers also provide a powerful portal where you can monitor everything from performance to cost.



### Development

* Use a virtual environment hosted in a cloud service to develop against the same metrics across the team.
* Host code repositories and deploy applications.
* Use cloud computing for resource-heavy projects.

To summarize the developmental side:

Developers can leverage cloud environments to make sure that everyone is developing against the same platform, an example of this is using Docker containers to ensure consistency across the development phase. Cloud environments can also be used to develop and/or build applications, for example when this can be used is for automatic deployment (CI/CD).



The cloud is basically an enormous ecosystem of servers that operate together and that can host most resources, but the cloud itself could be described as a service / toolbox for developers, DBAs, stakeholders, the list goes on...

​     

## Pros and Cons of The Cloud

#### Pros

* Build and scale systems easily without having to manually buy hardware / servers. (Platform as a Service)
* Deploy servers in seconds by leveraging powerful cloud systems (Infrastructure as a Service)
* Deploy software through cloud providers that costumers can access through a subscription based agreement. (Software as a Service)

#### Cons

* High-speed internet connection is needed, some applications such as PWAs might work with limited functionality offline, but all cloud-based software will need an active connection for optimal usage.

* Contracts and agreements must be followed as set by the cloud provider, if the customer does not agree with these, they will have to find another provider.
* Storaging sensitive information in the cloud could be a security risk due to malicious actors continuously targeting cloud providers due to their huge customer base.



## Price Conclusion

Different providers have different pricing on different resources. Because of this, a good practice is to compare several actors to find the optimal provider in price to performance.

####  Comparisons made with Virtual Machine template running Ubuntu

* Azure - Virtual Machine - 2vCPUs, 7GB RAM, 100GB SSD (931 KR / mån)

* Google Cloud - Compute Engine - 2vCPUs, 8GB RAM, 375GB SSD (Lowest) (748 KR / mån)

* IBM Cloud - Virtual Server - 2 vCPUs, 8GB RAM, 100GB SSD (Lowest) (659Kr / mån)

These prices are based solely on hardware and does not take into account bandwith speed and limits for example.
When comparing just the specs, IBM is the clear winner here when it comes to price to performance.


