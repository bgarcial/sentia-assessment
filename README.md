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
  So I would need to assure at least the same availability l
  - There is  here an opportunity to take advance of the [Azure Service-Level Agreements](https://docs.microsoft.com/en-us/learn/modules/explore-azure-infrastructure/6-service-level-agreements) under the perspective of the resources that this approach solution will be use. 

---
#### Initial Assumptions

According to the customer requirements previously mentioned, the design considerations to keep in mind
in order that the solution *"to be futureproof and expandable with new WordPress sites with minimal effort"* and the initial insights and question that arises to me; looks like I would need to create databases "on the fly". So I decided these initial considerations for the architecture.

**1. For MySQL databases Wordpress sites, I decided to use Azure SQL for MySQL as an external service to**:

- Make backups
- Easy maintainability
- Compliance
- Encryption
- High Availability
  - Availability Zones
  - The database will not be a single point of failure
- Create "databases on the fly" (Good question about CI/CD second part) 

**2. The Wordpress instances sites will be hosted and manage them by Azure Kubernetes Service to**:

-  AKS cluster with Availability zones (*Preview feature, excluded from the service level agreements*)
   -  Distribute the AKS nodes in multiple zones 
      -  Aks cluster is able to tolerate a failure in one of those zones
   -  Nodes run across separate update and fault domains in a single Azure data center

>Clusters with availability zones enabled require use of Azure Standard Load Balancers for distribution across zones

![alt text](https://cldup.com/H7LEvfWeKo.png "HA approach")


---
# REFERENCES

https://docs.microsoft.com/en-us/azure/aks/availability-zones