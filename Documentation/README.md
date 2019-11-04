# Documentation


This section presents the documentation about the process solution approach to address the assigment [presented here](https://github.com/sentialabs/public-cloud-recruitment/blob/master/ASSIGNEMENT.md)

A public cloud (Azure) high availability environment will be presented in order to provide scalability and flexibility to Wordpress customer sites. 

## 1. Facts about current customer deployment infrastructure.

The customer is currently hosting 10 wordpress sites using wordpress multisite in a private datacenter.
- *10 wordpress instances*
- They achieve HA  by using 2 servers and having 2 copies of their multisite
- For the database they are using 2 MySQL servers behind HA proxy to achieve HA


### 1.1. Problem/opportunity

- The past few months, they have been having a lot of issues because some of their websites have increased in popularity, especially during certain timeframes.
- So, the customer has decided to move away from wordpress multisites and have independent Wordpress applications.
- They have also pointed out that they have 5 more sites in the making that will reach Production in the next 12 months.

The client is only interested in developing the WordPress sites from an application perspective. They work using GIT repositories, and they have agreed to provide access to the application source code in one or more repositories.

- Here, is the opportune time to highlight that the Wordpress sites do not exist, this a ficticional case 

### 1.2. Requirements in the design and implementation. 


You have undertaken the task to design the future state of this environment in the public cloud. The solution needs to:

be scalable and flexible.
be futureproof and expandable with new WordPress sites with minimal effort.

#### 1.2.1. Part 1 - Transformation and Migration to the Public Cloud
- Use control version system to IaC templates (*Azure Resource Manager templates will be used here*).
- Provide a design for the designated Azure architecture.
- Provide ARM templates.
- Everything else that you need to accompany your solution with based on your approach.

#### 1.2.2. Part 2 - CI/CD
- Provide a design for the CI/CD pipeline that you will use to deliver the changes to the environment,every time the client updates any of their WordPress applications in GIT

### 1.3. Deliverables
- For Part 1, an architectural design and IaC templates for deploying the components.
- For Part 2, a complete architectural design of the CI/CD process.
- Please include a simple time log of the activities you have performed.
- Please document any assumptions and decisions you have made.
- Please include a presentation of the results within slides, ready to be presented to our client.

---

## 2. Assumptions
According to the customer requirements previously mentioned, is necessary to assure and improve the availability in the approach solution.

There is  here an opportunity to take advance of the [Azure Service-Level Agreements](https://docs.microsoft.com/en-us/learn/modules/explore-azure-infrastructure/6-service-level-agreements) under the perspective of the resources that this approach solution will be use. (*This will be addressed later*)

So I decided these initial considerations for the architecture.

### 2.1. MySQL database Wordpress sites.

- **Azure Database for MySQL will be used.**
The wordpress service will be deployed as stateless application, it means *"do not store data or application state to the cluster or to persistent storage"*. That's why is used MySQL as platform as service. 
Of this way, the wordpress sites will be easily scalable.

In addition, use MySQL as an external service have the following benefits: 

- Easy backups
- Easy maintainability
- Compliance
- High Availability
  - Availability Zones
  - The database will not be a single point of failure
- Create "databases on the fly" 

### 2.2. Hosting Wordpress instances sites

The Wordpress instances sites will be hosted and manage them by **Azure Kubernetes Service** to:

Note: Use kubernetes require to beard in mind some security concerns.

#### 2.2.1 **AKS cluster with Availability zones** (*Preview feature, excluded from the service level agreements*)

This will allow [distribute the AKS nodes in multiple zones](https://docs.microsoft.com/en-us/azure/aks/availability-zones) in order to have failure tolerance in one of those zones.
An Azure Standard Load Balancer is enabled for managing and traffic distribution across zones.

In this approach solution, AKS HA will be used across two availability zones.

##### Scalability

Several scalability benefits in AKS clusters.

- Kubernetes allow to control the number of instances of a specific service or pod
  - `replicas: x` on the `kind: Deployment` resource 
  - `kubectl scale` [command](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)

- The aks nodes could be scaled (*Multiple VM nodes with specific OS configuration*)
  - Using [cluster autoscaler](https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler) in AKS HA.
  - [Scale cluster](https://docs.microsoft.com/en-us/azure/aks/scale-cluster) in Azure Kubernetes Service without HA 
  - Adding [nodepools](https://docs.microsoft.com/en-us/azure/aks/use-multiple-node-pools) (nodes in the same configuration) in irder to use more nodes. 

##### Managing Deployments on Kubernetes

Applying namespace scope per customer
  - Restrict the communication between namespaces 
  - MySQL secrets by customers inside its own namespace.

- Manage k8s networks [policies between pods](https://kubernetes.io/docs/concepts/services-networking/network-policies/#the-networkpolicy-resource)
  - Define rules, which specific traffic is allowed to the selected pods
    - Containers can talk to any other container in the network,
    - Nodes in the cluster can talk to any other container in the network and vice-versa  

> By default, pods are non-isolated; they accept traffic from any source
> Each node in the cluster has a component called `kube-proxy` that is in charge of routing the traffic from anywhere in the cluster to the target Pod.
Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. 

The above features are several benefits provide by Kubernetes to manage deployments. 
Due to time reasons, in this approach solution only will be addressed the namespace scope per customer deployment, which opens the door to all other recommendations/benefits presented.

---

So, the points 2.1 and 2.2 are oriented to keep and improve the high availability required by the customers deployment.
According to it, this first high level perspective HA diagram is presented, keeping in mind the MySQL and AKS contexts deployments.

![alt text](https://cldup.com/-sNtukfxDS.jpg "HA approach")
In this approach solution two availability zones will be created. 
*Screenshot taken from Google maps*

---

## 3. Deployment Architecture

Some diagrams and short explanations are presented here. 

### 3.1. How the system is organized?

This is the [ARM Template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json) with the following Architecture components:

#### 3.1.2. Core components

**A Virtual Network and two subnetworks** were defined with availability zones scope:

- [AssessmentVnetTesting](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L273) 
  - [aks-subnet](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L279): Here, the Kubernetes cluster, aks nodes and pod ip address will be located
    - Microsoft.SQL endpoint 
  - [persistence-subnet](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L291): For Azure database for MySQL
    - Microsoft.SQL endpoint
    - Microsoft.Storage endpoint

##### About Virtual Network services endpoints.

Those [endpoints](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview) allow a direct relation between a virtual network and the Azure services, making use of the network private address space, using an internal private ip address. 
>This allow to secure your critical Azure service resources to only your virtual networks

The `aks-subnet` and `persistence-subnet` has two endpoints
`Microsoft.SQL` endpoint for both subnets
`Microsoft.Storage` for `persistence-subnet`

Of this way, is warranted direct communication toward Azure MySQL and storage accounts between and from those two subnets and only in the `AssessmentVnetTesting`.

![alt text](https://cldup.com/jxQHbvtg8i.jpg "NetworkServiceEndpoints")

In addition, a virtual network rule in Azure database for MySQL instance was defined [in order to only accept
traffic from `aks-subnet`](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L405) and [a rule to allow traffic from my home ip address](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L418).

![alt text](https://cldup.com/NA0pDsY_qA.jpg "VirtualNetworkRule")


#### 3.1.3. Infrastructure Services:

**Azure Kubernetes Service**

Kubernetes as an orchestration container services is used to manage the Wordpress application and get
the scalability and flexibility requirements
Kubernetes is being created and deployed [across two availability zones](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L333) allowing high availability configuration, which require the Azure standard load balancer inclusion in order to  traffic routing between zones and aks nodes where the the wordpress services are (pods)  

![alt text](https://cldup.com/MWwS6kBZWc.jpg "Kubernetes Behavior")



TALK ABOUT DETAILED K8S DIAGRAM


We will use our own Ingress controller deployment inside Kubernetes to expose the WordPress sites to Internet. 
When we create the Ingress process inside Kubernetes, the Ingress controller is in able to manage to become to be a Load Balancer
It'll act as an external Load Balancer, that's why the Kubernetes cluster does not include an LB, at least at the moment.

#### 3.1.4.Platform as a Service Components:
Azure Database for MySQL server instance
Rule name to allow access from my home IP address.
It could be necessary add later the Sentia HQ IP address. Just in case.

Azure Container Registry to store Wordpress Docker images
Like I mentioned yesterday the ResourceGroup and the ServicePrincipal (and its secrets used) are being created manually previously to the execution of the deployment.
This is something to include inside the template.
 
The deployment of these infrastructure resources are being executed from Powershell at Az-ResourceGroup level



Architecture Complete Landscape High level

![alt text](https://cldup.com/KfhPsd0zYk.jpg "Kubernetes Behavior")


 
---



## Issues and Risk Mitigation


Please document all the technical issues you have encountered so far in the final documentation.
 
Let's agree on the section name and subsections:
* Name: Risks mitigation;
   * Table of risks:
       Header: Id|Name|Description|Criticallity
   * Table of mitigations:
       Header: Id|Status<Open, 
       Closed>|Mitigation status description.


# PROBLEM 1

I cannot see the static files from my wordpress deployment,
Wordpress installation does not work well behind some load balancers

https://snippets.webaware.com.au/snippets/wordpress-is_ssl-doesnt-work-behind-some-load-balancers/

There is this plugin to install on the Wordpress instance deployed, but by itself does not worl
https://wordpress.org/plugins/ssl-insecure-content-fixer/


Looks like we would need to create a redirect rule in apache to allow https content
https://aws.amazon.com/es/premiumsupport/knowledge-center/redirect-http-https-elb/
https://cwiki.apache.org/confluence/display/HTTPD/RedirectSSL


Also the Wordpress helm chart has the `wordpressSchema=https` parameter, which force to any
client to use https protocol when we type the address with `http`
https://github.com/helm/charts/tree/master/stable/wordpress#parameters

---

# PROBLEM 2  --- ErrImagePull

When we install wordpress private image from helm chart and we got ErrImagePull
is because we need to associate the AKS with the ACR of this way

```
⟩ az aks update -n WordpressSentiaAssessment-aks -g sentia-assessment --attach-acr WordpressSentiaAssessment
{
  "aadProfile": null,
  "addonProfiles": null,
  "agentPoolProfiles": [
    {
      "availabilityZones": [
        "1",
        "2"
      ],
      "count": 3,
      "enableAutoScaling": null,
      "enableNodePublicIp": null,
      "maxCount": null,
      "maxPods": 110,
      "minCount": null,
      "name": "defaultpool",
      "nodeTaints": null,
      "orchestratorVersion": "1.14.7",
      "osDiskSizeGb": 1023,
      "osType": "Linux",
      "provisioningState": "Succeeded",
      "scaleSetEvictionPolicy": null,
      "scaleSetPriority": null,
      "type": "VirtualMachineScaleSets",
      "vmSize": "Standard_D2_v2",
      "vnetSubnetId": "/subscriptions/xxxxxx/resourceGroups/sentia-assessment/providers/Microsoft.Network/virtualNetworks/AssessmentVNetTesting/subnets/aks-subnet"
    }
  ],
  "apiServerAccessProfile": null,
  "dnsPrefix": "WordpressSentiaAssessment-dns",
  "enablePodSecurityPolicy": null,
  "enableRbac": true,
  "fqdn": "wordpresssentiaassessment-dns-a97c0ab0.hcp.westeurope.azmk8s.io",
  "id": "/subscriptions/xxxxxx/resourcegroups/sentia-assessment/providers/Microsoft.ContainerService/managedClusters/WordpressSentiaAssessment-aks",
  "identity": null,
  "kubernetesVersion": "1.14.7",
  "linuxProfile": null,
  "location": "westeurope",
  "maxAgentPools": 8,
  "name": "WordpressSentiaAssessment-aks",
  "networkProfile": {
    "dnsServiceIp": "100.0.0.10",
    "dockerBridgeCidr": "172.17.0.1/16",
    "loadBalancerProfile": {
      "effectiveOutboundIps": [
        {
          "id": "/subscriptions/xxxxxx/resourceGroups/MC_sentia-assessment_WordpressSentiaAssessment-aks_westeurope/providers/Microsoft.Network/publicIPAddresses/2e9b295c-32f3-4ce7-b265-f9deae9ec934",
          "resourceGroup": "MC_sentia-assessment_WordpressSentiaAssessment-aks_westeurope"
        }
      ],
      "managedOutboundIps": {
        "count": 1
      },
      "outboundIpPrefixes": null,
      "outboundIps": null
    },
    "loadBalancerSku": "Standard",
    "networkPlugin": "azure",
    "networkPolicy": null,
    "podCidr": null,
    "serviceCidr": "100.0.0.0/16"
  },
  "nodeResourceGroup": "MC_sentia-assessment_WordpressSentiaAssessment-aks_westeurope",
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "sentia-assessment",
  "servicePrincipalProfile": {
    "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", # APPLICATION CLIENT ID SentiaAssessment SP
    "secret": null
  },
  "tags": {
    "Environment": "testing"
  },
  "type": "Microsoft.ContainerService/ManagedClusters",
  "windowsProfile": null
}
[I] 
~/projects/sentia-assesment · (master±)
```

Looks like that association between aks and acr is not possible from arm template
https://github.com/MicrosoftDocs/azure-docs/issues/39508

>Cannot be done in an ARM template because this is a CLI client side change only. We enabled this in the CLI for customers who find it hard to understand ARM RBAC. The simple implementation behind the scenes is a specifically a simple role assignment allowing AKS to access 

So  that role assignment is performed over the service principal used to create the AKS cluster from the arm template
so indeed, I will need to look at the permissions of the service principal

So I am using the [SentiaAssessment](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationMenuBlade/Overview/appId/b496b133-0dbc-4a2b-8e6f-622def202432/isMSAApp/) SP to my AKS clueste
Let's check the permissions over ACR ...


---