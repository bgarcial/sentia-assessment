# Sentia Wordpress Technical Assesment

This repository presents an approach and possibile alternatives to address the assigment [presented here](https://github.com/sentialabs/public-cloud-recruitment/blob/master/ASSIGNEMENT.md)

## Part 1 - Transformation and Migration to the Public Cloud

### Some facts

- The customer is currently hosting 10 wordpress sites using wordpress multisite in a private datacenter.
  - *(10 wordpress instances - 1 databases ?)*
- They achieve HA  by using 2 servers and having 2 copies of their multisite
- They are using 2 MySQL servers behind HA proxy to achieve HA

---
### Problem / Opportunity

Some of their websites have increased in popularity especially during certain timeframes

#### Requirements 
- So, the customer has decided to move away from wordpress multisites and have independent Wordpress applications
- They have 5 more sites in the making that will reach production in the next 12 months
  - *5 additional sites wordpress instances - 5 additional databases?*

First question that arise me: 
  *Would we need to support 15 db in the short term?*

---
### To Do.
#### Design considerations to keep in mind

You have undertaken the task to design the future state of this environment in the public cloud. The solution needs to:

- be scalable and flexible.
- be futureproof and expandable with new WordPress sites with minimal effort.

Please provide a design for the designated architecture in either AWS or Azure. For the same design, please provide CloudFormation or ARM templates, and everything else that you need to accompany your solution with based on your approach.

---

### Approaching the solution

#### Facts about solution
- I decided to address an Azure approach, so I would need to use ARM templates as a native tool for IaC
---

#### Facts about current customer deployment infrastructure.

- The customer already using 2 MySQL servers behind and HAProxy to achieve HA. 
  So I would need to assure at least the same availability 
  - There is  here an opportunity to take advance of the [Azure Service-Level Agreements](https://docs.microsoft.com/en-us/learn/modules/explore-azure-infrastructure/6-service-level-agreements) under the perspective of the resources that this approach solution will be use. 

---
#### Initial Assumptions

According to the customer requirements previously mentioned, the design considerations to keep in mind are:

>*"to be futureproof and expandable with new WordPress sites with minimal effort"* 

And the initial insights and question that arises to me; looks like I would need to create databases *"on the fly"* (I could be wrong). 

So I decided these initial considerations for the architecture.

**1. For MySQL databases Wordpress sites, I decided to use Azure SQL for MySQL as an external service to**:

- Make backups
- Easy maintainability
- Compliance
- Encryption
- High Availability
  - Availability Zones
  - The database will not be a single point of failure
- Create "databases on the fly" (Good question/concern to be addressesd in CI/CD second part?) 
---
**2. The Wordpress instances sites will be hosted and manage them by Azure Kubernetes Service to**:

Note: Use kubernetes require to beard in mind some security concerns

-  AKS cluster with Availability zones (*Preview feature, excluded from the service level agreements*)
   -  Distribute the AKS nodes in multiple zones 
      -  Aks cluster is able to tolerate a failure in one of those zones
   -  Nodes run across separate update and fault domains in a single Azure data center
>Clusters with availability zones enabled require use of Azure Standard Load Balancers for distribution across zones. 


- Scalability 
  - Number of instances of a specific service or pod
    - `replicas: x` on the `kind: Deployment` resource or `kubectl scale` command
  - The aks nodes could be scaled (*Multiple VM nodes with specific OS configuration*)
    - Manually using `az cli`
    - Using cluster autoscaler (*currently in preview and excluded from the service level agreements and limited warranty*)
- Applying namespace scope per customer
  - Restrict the communication between namespaces 
  - MySQL secrets by customers inside its own namespace.
- Manage k8s networks policies between pods
  - Define rules, which specific traffic is allowed to the selected pods
    - Containers can talk to any other container in the network,
    - Nodes in the cluster can talk to any other container in the network and vice-versa  

> By default, pods are non-isolated; they accept traffic from any source
> Each node in the cluster has a component called `kube-proxy` that is in charge of routing the traffic from anywhere in the cluster to the target Pod.
Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. 

So, the points 1 and 2 are oriented to the HA that the customers already have and need improve.
That's why, this first High level perspective approach HA diagram keeping in mind the MySQL and AKs contexts deployments


![alt text](https://cldup.com/H7LEvfWeKo.png "HA approach")
*Screenshot taken from Google maps*

---

**3. Vnet and two subnets**

- Customer VNet
  - MySQL subnet
  - AKS subnet
    - Keep in mind the communication with availability zones.
     
---


#### Questions

**1. Does WordPress instances (Wordpress pods inside AKS) need to communicate with the Internet?**
  - Maybe not
    - I can define a network policy in order to allow the pods to talk only to the MySQL subnet.
  - Maybe yes
    - Maybe I can define a rule that allows accessing to Internet only to install Wordpress plugins from admin dashboard. 

Note: If the admin dashboard should be exposed to Internet, keep in mind protect their access.
(Secure endpoint access with basic-auth (different to wordpress admin access application)
   

**2. In the definition of the problem/opportunity, the requirements says:**

>The client is only interested in developing the WordPress sites from an application perspective. 
They work using GIT repositories, and they have agreed to provide access to the application source code in one or more repositories.

I am thinking to have access to these customer wordpress repositories solutions, so my questions here are:

- **Do these repositories currently exist?**
  - If yes, Can I have access to them?

I ask that because maybe I could consider important to know the Wordpress code project structure, 
and maybe according to it I can consider to create its own docker images; which lead me to the idea of having container registries repositories on azure. 
**Yes one component more in the architecture**

- **What kind of beside services are they using or specific fuctionalities do they have in the development wordpress sites process?**
  - **Do we need to compose a set of those services?**
    - MySQL is discarded to be part from an eventual `docker-compose.yml` becuase we are using it as a platform service
    - We could use *docker compose* process to build and push all the website images 
  - ** What about `Dockerfile` activities?**
    - work dirs or fetch some other images
    - Copy some directory or important startup project files?
    - Launch some commands/entry points?
    - Run some build process (php, or related technologies that they are using)
 
Depending of the customer bussines logic and the functional requirements of the websites
would be possible consider the following benefits when we build our own application docker images:
- One container repository per wordpress website application portal
  - Better control of the different versions of the Wordpress sites
  - Improve the creation of artifacts and the release cadence in the CI/CD process
  - Upgrading Wordpress version building the latest version from Dockerfile.
    
- **There are not wordpress application github repositories?**
  - If no, I would need to assume that when we talk about Wordpress sites. I have to get them from somewhere


---
#### Getting into in the Deployment Architecture Desgin.

According to the previous facts, assumptions and questions,
I want to detail the design of the solution 
(Draft diagram ... of course. I have to create it from draw.io, it's just for illustrative purposes.)

![alt text](https://cldup.com/NBFVWcTdRd.jpg "HA second approach")


---
# REFERENCES

https://docs.microsoft.com/en-us/azure/aks/availability-zones
https://docs.microsoft.com/en-us/azure/aks/scale-cluster
https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler
https://kubernetes.io/docs/concepts/services-networking/network-policies/