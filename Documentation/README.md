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

- Be scalable and flexible.
- Be futureproof and expandable with new WordPress sites with minimal effort.

#### 1.2.1. Part 1 - Transformation and Migration to the Public Cloud
- Use control version system to IaC templates (*Azure Resource Manager template was used here*).
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

## 2. Assumptions and Deployment Architecture decisions made
According to the customer requirements previously mentioned, is necessary to assure and improve the availability in the approach solution.


There is  here an opportunity to take advance of the [Azure Service-Level Agreements](https://docs.microsoft.com/en-us/learn/modules/explore-azure-infrastructure/6-service-level-agreements) under the perspective of the resources that this approach solution will be use. (*This will be addressed later*)

So I decided the following considerations for the architecture.

### 2.1. MySQL database Wordpress sites.

- **Azure Database for MySQL was used.**
The wordpress service was deployed as [stateless application](https://cloud.google.com/kubernetes-engine/docs/how-to/stateless-apps), it means *"do not store data or application state to the cluster or to persistent storage"*. That's why is used MySQL as platform as service. 
Of this way, the wordpress sites will be easily scalable.

In addition, use MySQL as an external service have the following benefits: 

- Easy backups
- Easy maintainability
- Compliance
- High Availability
  - Availability Zones
  - The database will not be a single point of failure
- Create "databases on the fly".

The Azure database for MySQL manage service was deployed in a HA zone redundant approach.
According [to this article](https://azure.microsoft.com/en-us/blog/azure-sql-database-now-offers-zone-redundant-premium-databases-and-elastic-pools/), if the MySQL is deployed as a premium database tier, it will support multiple redundant replicas for each database that are automatically 
provisioned in the same datacenter within a region 

I've choose the following specifications: 
- [General Purpose](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L165) sku pricing tier was considered as starting point. There is the possibility to migrate to the **Memory optimized** pricing tier, just in case that memory performance, and transaction processing activities require more compute power.
- Compute Gen 5 [as an sku family](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L174)
- I decided to use local redundant backup and [not geo redundant backup](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L201) due to geo redundant backup store the backups not only within the region in which the MySQL server is deployed, azure also replicate the backup to a paired datacenter in a different region. 
  - At the moment I considered local redundant backup with only West Europe region scope as a good starting point with relation to MySQL high availability deployment. If we want to to enable a geo-redundant backup is only [change the default value to Enabled in the ARM template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L203) 
- The `storageAutoGrow` [also was disabled](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L192) due to in he General Purpose pricing tier I have available a range [between 5GB and 4TB of storage size](https://docs.microsoft.com/en-us/azure/mysql/concepts-pricing-tiers#storage) 
- The `"sslEnforcement": "Disabled"` also was selected in order to allow connections to MySQL server without need certificates. The inbound connections to MySQL are secure, because of network service endpoints.
You can see later the **About Virtual Network services endpoints** section to understand how works the services endpoints over subnets in a security context

Always will be a good idea [to stay monitoring MySQL](https://docs.microsoft.com/en-us/azure/mysql/concepts-monitoring) in order to start to re-consider the server configuration parameters


### 2.2. Hosting Wordpress instances sites

The Wordpress instances sites will be hosted and manage them by **Azure Kubernetes Service** to:

#### 2.2.1 **AKS cluster with Availability zones** 
[Currently available for West Europe](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=kubernetes-service). In AKS service, the availability of the agent nodes in the cluster is covered by the [Virtual Machines service level Agreement](https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/)

This will allow [distribute the AKS nodes in multiple zones](https://docs.microsoft.com/en-us/azure/aks/availability-zones) in order to have failure tolerance in one of those zones.
An Azure Standard Load Balancer is enabled for managing and traffic distribution across zones.

In this approach solution, AKS HA was used across two availability zones, due to the customer has two servers (hosting application environment) and two MySQL servers to achieve HA in their existing approach.


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

Use of Kubernetes environments require to beard in mind some security concerns like internamespace and intercontainer communications inside the cluster. 
I am mentioning here how to mitigate that security concerns, but due to time reasons, these thre weren't implemented
into the kubernetes environment

**Applying namespace scope per customer**
  - [Restrict the communication between namespaces](https://github.com/ahmetb/kubernetes-network-policy-recipes/blob/master/04-deny-traffic-from-other-namespaces.md) 
  - MySQL secrets by customers inside its own namespace.

- **Manage k8s networks [policies between pods](https://kubernetes.io/docs/concepts/services-networking/network-policies/#the-networkpolicy-resource)**
  - **Define rules, which specific traffic is allowed to the selected pods**
    - Containers can talk to any other container in the network,
    - Nodes in the cluster can talk to any other container in the network and vice-versa  

> By default, pods are non-isolated; they accept traffic from any source
> Each node in the cluster has a component called `kube-proxy` that is in charge of routing the traffic from anywhere in the cluster to the target Pod.
Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. 

- **Monitoring usage data resources**

Ok, we have an AKS HA cluster, with two availability zones, with Wordpress deployed as stateless application using
a MySQL managed database in a HA deployment as well. Those are high availability features under a higher cloud infrastructure perspective. Good for us.

**But what about container process scaling?** 
I mean, **what about Wordpress kubernetes behavior application?**

In scalability terms, also is important examine the application performance, monitoring the resources usage
at containers and pods levels in order to evaluate the application performance and find and remove bottlenecks to improve the overall performance.

Kubernetes allow to [monitor resources according to metrics](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/#resource-metrics-pipeline) like **CPU** and **memory usage.**

**What if some container or pod is overloaded with traffic?** (despite that we already have a Load Balancer of course ...)
Maybe the pods could be going down or start to restart itself, or even become to does not accept traffic in some point (according to `readiness` and `liveness` probe parameters in the deployment)

If we are using kubernetes, we can evaluate these kind of situations through monitoring tools like [metrics-server](https://github.com/helm/charts/tree/master/stable/metrics-server) inside our entire cluster.

I am encouraging myself to explore [Horizontal Pod AutoScaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) to automatically scale the number of pods in our Wordpress deployment based on observed CPU utilization as a defined metric.

I started (at least) to define the `HorizontalPodAutoScaler` [resource here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/WordpressHPA.yaml) which make use of a `ReplicaSet` resource [defined here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/WordpressReplicaSet.yaml)

The Kubernetes documentation give [a good walkthrough sample](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) using Apache as a web server and using a PHP script which performs some CPU intensive computations. 

At the moment this approach to monitor and scale at the container and pods level is considered
as a improve in the current architecture approach, due to time reasons to implement it because it.


**IMPORTANT NOTE**
The described above features are several benefits provide by Kubernetes to manage our Wordpress deployments 
Due to time reasons, in this approach solution only will be addressed the namespace scope per customer deployment, which opens the door to all other Network policies recommendations/benefits.

Also independent if an Horizontal Pod Auto Scaler is implemented or not, always will be a good idea involve
monitoring tools at different levels behavior like usage data resources or webserver, MySQL and load balancer logs. 

In this solution, I am not implementing any monitoring tool in particular, I just using `kubectl logs` as a mechanism to check the Wordpress, NGINX and cert-manager behavior. 
Despite everuything I wanted to try to explain and give ideas and some kind of roadmap in order to start to involve
monitoring activities in this solution approach.

A separate section in this documentation will be included, gathering all the future recommendations in order to improve the behavior of my architecture approach.

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

I would like to detail here the Virtual Network configuration section and why I made those configuration decisions about it.

- [AssessmentVnetTesting](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L273) 
  - [aks-subnet](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L279): Here, the Kubernetes cluster, aks nodes and pod ip address will be located.
    - aks-subnet has a `Microsoft.SQL` virtual network endpoint 
  - [persistence-subnet](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L291): For Azure database for MySQL. It has the following virtual networks endpoints
    - `Microsoft.SQL` 
    - `Microsoft.Storage`

The virtual networks services endpoints will be discussed later. 
The Vnet and subnets have the following configuration:
- `AssessmentVnetTesting` has [an address space of 10.0.0.0/8](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L21)
  - It is an A class private address whith range between `10.0.0.0 - 10.255.255.255`   and netmask `255.0.0.0`, that's why `/8`
- `aks-subnet` has [an address space of 10.240.0.0/16](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L28)
  - It is an address range supported within Vnet address space, by consequence also within A class and private address range as well with range between `10.240.0.0 - 10.240.255.255`.
  - It netmask is `255.255.0.0` that's why `/16` with a capacity of 65.536 adresses to grow. 
  Actually, with the AKS deployment I have around of 65.198 addresses available.

  ![alt text](https://cldup.com/gRNyd3IIFR.png "Aks Subnet configuration")
   
  It means that 338 addresses has been assigned to the `aks-defaultpool` which contain 3 nodes into kubernetes environment.
  
  
  So if we go to the Vnet configuration, we can see in **Connected devices** option all these addresses assigned
  and distributed across different nodes in the pool (`aks-defaultpool-34253081-vmss (instance 0)`, `aks-defaultpool-34253081-vmss (instance 1)` and `aks-defaultpool-34253081-vmss (instance 2)`)

  ![alt text](https://cldup.com/cQYKVLk-vK.png "Aks Subnet ip addresses connected/reserved")
  ![alt text](https://cldup.com/dZvVK6nCP6.png  "Aks Subnet ip addresses connected/reserved")
  ![alt text](https://cldup.com/8dTXt52J5k.png  "Aks Subnet ip addresses connected/reserved")
  
  But ... **why the addresses already were assigned or taken by Kubernetes environment through it nodes?**
  
  Well, let's talk a bit about **Azure Kubernetes** networking modes behavior:
  Azure Kubernetes in general has two kinds of networks types:
  - **Basic Networking**: 
    - AKS manage it's own virtual network and expose only public IPs
    - Pods IPs are accessible from within the cluster.
    - Only publicly exposed services are accessible outside the cluster.
  - **Advanced Networking**:
    - AKS’s nodes are deployed within a specified Azure Virtual Network we control.
    - Is possible deploy AKS inside an existing Vnet and exposes private IP
      - This allows us to keep the services running on AKS on private IPs.
        - **It's exactly I am doing with the IP addressess inside aks-subnet above in the pictures**  
      - It also allows us to make them accessible from on premise network.
        - **Just in case**

    **So, use advancing networking mode will allow us to integrate a new aks-nodepool (more nodes inside existing AKS cluster) or even a new AKS cluster into `AssessmentVnetTesting` in a short future.**

**IMPORTANT - PROS AND CONS CNI**
- Also important to keep in mind here is that Kubernetes as a general service implement two kind of network plugins  
[kubenet](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#kubenet) and [CNI](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#cni) (which is a [container network specification based](https://github.com/containernetworking/cni/blob/master/SPEC.md#network-configuration)).

- So if we want to work with Advanced networking mode, we have to use Container network specification as a plugin.
In the Azure cloud case, it has the [Azure VNET CNI Plugin](https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md), implementing the same Container Networking specification.

  
I am using CNI network profile in the [arm template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L247) (it is given by `azure` value)

- With Azure CNI, Basically, both pods and services get a private IP, they belong to the `AssessmentVnetTesting` virtual network. 
- Services also get a cluster-ip (accessible only from within the cluster).
- every pod gets an IP address from the subnet and can be accessed directly

If we use Azure CNI, a common issues are:

>- With Azure CNI, a common issue is the assigned IP address range is too small to then add additional nodes when you scale or upgrade a cluster. 
>- The network team may also not be able to issue a large enough IP address range to support your expected application demands.
>- Azure CNI could only support a maximum of 8 nodes in the cluster 

**How can I decide to use Azure CNI or Kubenet network profiles?**

- **KUBENET**
  - I have limited ip address space
    - Use NAT to route traffic between pods and nodes, because Nodes and pods are placed on different logical address space. 
      - Nodes receive an ip address from azure virtual network and pods use CIDR - POD. So NAT is configured to the pods can reach resources on the Azure Virtual Network 
        -  This additional routing may reduce network performance
    - I will not need a huge address space. 
      - **I have to say that this approach solution, does not require a huge address space.**
  - Most of the pod communication is within the cluster (**My pods needs to communicate with MySQL managed database service**).
  - When advanced AKS features such as virtual nodes or Azure Network Policy aren't needed.
    - **In this Wordpress deployments, I think [I don't need virtual nodes](https://docs.microsoft.com/en-us/azure/aks/virtual-nodes-portal), but I consider important
      get into Network policies to securize container or pods traffic, that's why CNI was considered.** 

---
- **AZURE CNI**
  - I have available IP address. 
    - **In this deployment, the subnetting scheme was choosen as a testing purposes, but also considering
     the future growing or more Wordpress sites in new AKS nodes or even new clusters**
  - In CNI is necessary  to design our Network address space to accommodate future scale outs. 
    - it has the advantage of not doing NAT and DMASK processes at node level, giving  more performance and it will be faster routing packets.
  - Most of the pod communication is to resources outside of the cluster.
      - **The pods are communicating with Azure database for MySQL instance in the `persistence-subnet`**
  - Use advanced features like [Network policies](https://docs.microsoft.com/en-us/azure/aks/use-network-policies)
    - **This option was purposed from the beggining of this document at 2.2.1 Managing Deployments on Kubernetes  section**
      - I consider important to restrict the traffic between namespaces and pods, because despite I am separating wordpress deployments by namespace, by default there is an intercontainer and internamespace communication, so will be necessary restrict the communication between different wordpress deployments to: 
        - Each Wordpress deployment would have its own apache web server (its own configuration)
        - Each Wordpress deployment would have its own plugins at Wordpress level. 
        - Each Wordpress deployment have its own secrets pointing to an specific MySQL database. 

>Also there is some scenarios where will require only CNI. Like setting up Ingress Controller Application Gateway where the backend pool will be the POD ID’s instead of the nodes. And only works if POD IP’s are visible from subnet level (CNI).

**I have a Load Balancer and an Ingress controller, but the backend pools are the aks nodes not the POD IDs.**
The Load Balancer topic belong to Kubernetes environment despite it is a network topic. 
I will discuss it later.

Is opportune to highlight despite I am using Azure CNI as a plugin Network, the Network policies process is not being implemented due to time reasons. Network policies will be considered as a future work to improve the solution. 

This [google document](https://docs.google.com/document/d/1hN2XXWuxXvO75Ri-IONWnrRDTGW8iaFTNgXRru92I9o/edit?usp=sharing) reference several article sources, where I took most of the pros and cons discussed above.

---
#### About Virtual Network services endpoints.

Those [endpoints](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview) allow a direct relation between a virtual network and the Azure services, making use of the network private address space, using an internal private ip address. 
>This allow to secure your critical Azure service resources to only your virtual networks

The `aks-subnet` and `persistence-subnet` has two endpoints
`Microsoft.SQL` endpoint for both subnets
`Microsoft.Storage` for `persistence-subnet`

Of this way, is warranted direct communication toward Azure MySQL and storage accounts between and from those two subnets and only in the `AssessmentVnetTesting`.

![alt text](https://cldup.com/jxQHbvtg8i.jpg "NetworkServiceEndpoints")

In addition, a virtual network rule in Azure database for MySQL instance was defined [in order to only accept traffic from `aks-subnet`](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L405) and [a rule to allow traffic from my home ip address](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L418).

![alt text](https://cldup.com/NA0pDsY_qA.jpg "VirtualNetworkRule")


#### 3.1.3. Infrastructure Services

**Azure Kubernetes Service**

Kubernetes as an orchestration container services is used to manage the Wordpress application and get the scalability and flexibility requirements.

Kubernetes is being created and deployed [across two availability zones](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L333) allowing high availability configuration, which require the Azure standard load balancer inclusion in order to  traffic routing between zones and aks nodes where the the wordpress services are (pods). 

**First of all let's talk a bit about Kubernetes configuration used:**

- **Node size**: The size of the virtual machines that will form the nodes in the cluster.
  - I am using `Standard_D2_v2` for [agent pool profile](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L85) and [master profile](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L332)

  The `Standard_D2_v2` give me **2 vcpus and 7 GiB memory** per node. I have 4 nodes so I have 8 cores and 28 gb of memory.

---
**I consider to highlight here the master kubernetes and non master nodes (placed at agent pool profile)**

We got the [k8s control plane](https://kubernetes.io/docs/concepts/#kubernetes-control-plane) which govern how kubernetes communications is performed, maintaining a record of all k8s objects in the system and manage them via control loops in order to respond to changes in the cluster.
This control plane contain the **kubernetes master process**, which refers to a collection of processes managing the cluster state to make global decisions about scheduling, starting pods, etc.

**[Master components](https://kubernetes.io/docs/concepts/overview/components/#master-components)** can be run on any machine in the cluster and it has inside:
- kube-apiserver, to expose the kubernetes api which is the frontend for the k8s control plane. That's why when we use `kubectl` we are communicating with the control plane.
- etcd is a key value store that K8s use as a backing store for all cluster data.
- kube-scheduler, is a kind of watching process looking for new pods created without node assigned in order to 
  select a node for them
- Kube-controller-manager, runs controllers (components that watch the state of the cluster and make or request changes when are needed.) 
  - Node controller: Responsible for noticing and responding when nodes go down.
  - Replication controller:  Responsible for maintaining the correct number of pods for every replication controller object in the system. 
  - Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
  - Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.
- Cloud controller manager,  runs controllers that interact with the underlying cloud providers via some dependencies.

**[Node components](https://kubernetes.io/docs/concepts/overview/components/#node-components)** are inside the agentpool profile, that's why run on every node maintaining running pods and providing the Kubernetes runtime environment. Are node components:

- kubelet: An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.
- kube-proxy: s a network proxy that runs on each node in the cluster. It maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
---
Well, all those master and node components described above, **should be distributed across availability zones that I defined.**

So, I decided:
- **Use 2 Availability Zones** for master profile ([ARM template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L334)) and **2 availability zones** for agentpool profile ([ARM template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L350))
In this approach solution, AKS HA was used across two availability zones, due to the customer had two servers (hosting application environment) and two MySQL servers to achieve HA in their existing approach.

- ##### Node count: Is the number of nodes in the node pool defined to k8s.
  >[To ensure HA](https://github.com/Azure/aks-engine/blob/master/examples/kubernetes-zones/README.md#availability-zones), each profile (master and node) must define at least two nodes per zone. For example, an agent pool profile with 2 zones must have at least 4 nodes total.

  The agentpool profile (node components) has 4 nodes in total ([ARM template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L343))

  The master profile (master component) has 5 nodes in total ([ARM template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L330))   
  
 
 - **Type of nodepool profile and masterprofile** 
   This is the type of virtual machine which support those nodepools defined here
   
   I decided to choose **[VirtualMachineScaleSets](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview)** to both aksdefault pool ([ARM Template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L347)) and master profile ([ARM Template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L347))
   
   This decision was made in order to allow [adjust the number of nodes inside an Azure Kubernetes cluster](https://docs.microsoft.com/en-us/azure/aks/cluster-autoscaler) and to keep up with Wordpress application demands 
   
   When I did this and the kubernetes cluster was deployed from ARM template, I could see in Azure portal the following features options

  ![alt text](https://cldup.com/w9WcRNkWdh.png "AKS with Autoscale nodes")
   

The following diagram try to detail the internal Kubernetes environment behavior when a Wordpress service is deployed into Kubernetes. 

I would like to emphasize here only in the Load Balancer behavior and kubernetes things. 


![alt text](https://cldup.com/ahhoyNGKEN.png "Kubernetes Behavior")


**You can see this kubernetes behavior picture in a bigger size [here](https://cldup.com/ahhoyNGKEN.png)**


**I consider necessary to highlight here the Azure Standard Load Balancer here.**

When we deploy a high availability Azure Kubernetes cluster, it require to have a Load Balancer to manage traffic and distribute it across availability zones across the groups of backend pool servers, in order to provide scalability, which is the primary goal of load balancing.

The idea on this approach solution is to have an optimal load distribution in order to reduce site inaccessibility caused by the failure of a single server or two of them (like the customer had deployed), assuring consistent performance for all users.

A Wordpress aplications like mentioned in this case of study, must support concurrent connections from clients
requesting text, images, video, or application data all in a fast and reliable manner, while scaling from hundreds of users to millions during peak times. Load balancers are a critical part of this scalability. 

When the AKS cluster was deployed from ARM template, I got in Azure the following Standard Load Balancer.

![alt text](https://cldup.com/7L5dQjhZi1.png "Azure LB")
**You can see this LB configuration picture in a bigger size [here](https://cldup.com/7L5dQjhZi1.png)**



This Public IP address created is known as **Frontend IP configuration**
![alt text](https://cldup.com/E4O0WEq5Kg.png "Azure LB")

A public Load Balancer like this maps the frontend IP address and port number of incoming traffic to the private IP address and port number of the virtual machine (in this case of the VM scale sets defined previously), and vice versa for the response traffic from the virtual machines. In my case from Kubernetes cluster.

Those virtual machines that we are talking here, are the nodes located in the nodepool created in my Kubernetes cluster. They are the backend pools, and those are the same nodes referenced [in the agentpool profile component in my ARM template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L76)

![alt text](https://cldup.com/8qex2CkIgM.png "Azure LB NODEPOOLS")

**You can see this LB nodepools picture in a bigger size [here](https://cldup.com/8qex2CkIgM.png)**


By applying load-balancing rules, is possible distribute specific types of traffic across multiple VMs or services. 
For example, is possible spread the load of web request traffic across multiple web servers.

>Resources within the Assessment virtual network are not directly reachable from the outside unless that someone takes specific steps to expose them through public endpoints or connects them to on-premises networks through a virtual private network (VPN) or Azure Express Route. 

**So, is here where the Ingress Controller process inside Kubernetes environment take place**

We are using NGINX Ingress controller deployment inside Kubernetes to expose the WordPress sites to Internet. 
When we create the Ingress controller process inside Kubernetes, the Ingress controller is deployed as a Load balancer service. Let's see.

From the release pipeline in Azure DevOps, this command is executed:

`helm3 install nginx-ingress stable/nginx-ingress --namespace nginx`

![alt text](https://cldup.com/velmfOr3lR.png "Creating NGINX Ingress controller deployment")


When this happens, NGINX is installed as an Load Balancer service


![alt text](https://cldup.com/iGfxZAYX-Z.png "Creating NGINX Ingress controller deployment")

The External IP that I got here, is attached to the Load Balancer created when the AKS high availability cluster was created. If I go to the Load Balancer options in Azure Portal, I can see it has now 2 public IP addresses

![alt text](https://cldup.com/8z6QpT6k8u.png "Azure Load Balancer with 2 frontend IPs")

**You can see this Azure Load Balancer with 2 frontend IPs picture in a bigger size [here](https://cldup.com/8z6QpT6k8u.png)**

Inside the red rectangle I can see the same external IP address NGINX Ingress Controller IP address.
The orange rectangle is the public IP address created when the AKS HA cluster was deployed.

![alt text](https://cldup.com/cACcIPIlAW.png "Azure Load Balancer with 2 frontend IPs")

This Public IP address has two rules allowint the 80 and 443 ports to http and https.

![alt text](https://cldup.com/knwcsvUjXA.png "Azure Load Balancer rules")

These rules given by NGINX public IP address now are part of the general AKS HA Load Balancing rules

![alt text](https://cldup.com/tfbD8Wmbk8.png "Azure Load Balancer rules")
**You can see this Azure Load Balancer rules picture in a bigger size [here](https://cldup.com/tfbD8Wmbk8.png)**

And if we detail each rule, we can see the mapping between the frontend IP adresss and port number with the backend port and backend pool 

![alt text](https://cldup.com/R-pggL--qa.png "Azure Load Balancer mapping frontend - backend pools")

For the 80 port is the same situation
![alt text](https://cldup.com/I7D1apu-Fc.png "Azure Load Balancer mapping frontend - backend pools")

These are the backend pools that I've mentioned at the beggining of the Load Balancer section above.


#### 3.1.4.Platform as a Service Components:

- Azure Database for MySQL server instance
- Azure Container Registry to store Wordpress Docker images
  I decided not enable [geo replication for container images](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-multi-region#enable-geo-replication-for-container-images) due to the current customer performance concerns are not related with speed to store and pull the container images. 
---

### 3.2. Highest Level Architecture Deployment

According to all previously mentioned this is the general communication workflow between the
components that are involved in the solution approach purposed. 
The CI/CD process is not mentioned in detail here, it will be explained later in a specific point.

![alt text](https://cldup.com/zT9vCqN6Py.jpg "Kubernetes High Level Behavior")

**You can see this architecture picture in a bigger size [here](https://cldup.com/zT9vCqN6Py.jpg)**

---

## 4. Procedures to maintain the system

Is necessary to keep in mind the different components involved in the architecture deployment which were
mentioned in the **3.1 How the system is organized** section

Having understood how the the Vnet, Subnets and their network service endpoints were deployed and how they are working with the AKS cluster and MySQL instance I can reference the following important facts at the moment to
make modifications to the deployment:

### 4.1 Network Connectivity configuration 
The Kubernetes cluster was deployed using Azure Container Network Interface plugin having the following address space:

- `AssessmentVnet` has an address space of 10.0.0.0/8. If you want to modify/re-design the network address space 
  is necessary modify it [here in the ARM template](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L21)
- `aks-subnet` has an address space of 10.240.0.0/16. If you want to modify/re-design the network address space, is necessary modify it [here in the ARM template](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L28)
- `persistence-subnet` has an address space of 10.241.0.0/27. If you want to modify/re-design the network address space, is necessary modify it [here in the ARM template](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L42)
  I decided to use `/27` as a netmask (`255.255.255.224 = 27`) in order to save optimize the address space in the persistence-subnet. `/27` as a netmask allow me to reserve until 32 private IP addressses. Here I only have deployed the MySQL server  
  
  ![alt text](https://cldup.com/vhZM4NaWVp.png "Persistence-subnet address range")
  27 adressses are available if we want to add more services in this subnet. 

**IMPORTANT**

- If you want to modify the subnetting schema for the Vnet and the subnets, please keep in mind the Networking facts and considerations referenced above **in the 3.1.2 section**: 
  - **A Virtual Network and two subnetworks** 
  - **Basic Networking** and **Advanced Networking** 
  - **IMPORTANT - PROS AND CONS CNI**
  - **How can I decide to use Azure CNI or Kubenet network profiles?**
  - And also read about [the virtual network service endpoints explanation](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#about-virtual-network-services-endpoints) at the subnets level

If you want to change the Network profile mode from Advanced (CNI) to Basic (Kubenet), is possible to do it 
and keep the services up and running. Just make sure that some route tables between subnets were created and the network endpoints also persist.

If you decide to change to the basic netwok profile, please keep in mind that the Network policies recommended to apply in the [Managing deployments on kubernetes section](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#managing-deployments-on-kubernetes), they will cannot be applied in the basic network profile.

### 4.2. Azure Kubernetes HA configuration.

You can modify the HA schema in the cluster. If you want to do it, you should go to the options in the [ARM template](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L322). Please check from the line 322 until 355 code section.

Please if you want to know about AKS HA configurations details read the **Azure Kubernetes Service** section located [here](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#313-infrastructure-services)
Going through these configuration details will allow you to be familiarized with important deployment features like:
- Availability zones (for master and agent pool profiles)
- Azure Standard Load Balancer (mandatory for HA schemas. Please refer to **"I consider necessary to highlight here the Azure Standard Load Balancer here."** section)
- Virtual machine Size and number of nodes and its distribution across zones.

These previous points are very important detail to keep in mind at the moment of deploy or modify an AKS HA service

### 4.3. Azure Database for MySQL HA configuration.

You can modify the HA schema for MySQL managed service. If you want to do it, you should go to the options in the [ARM template](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L374). Please check from the line 374 until 437 code section.
The MySQL configurations parameters are explained in the **[2.1. MySQL database Wordpress sites](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#21-mysql-database-wordpress-sites)**
You can explore the links information provided there in order to understand the changes to do.

#### 4.3.1.  Adding Azure Virtual Network Rules

In the [ARM template](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L248) some firewall rules were added in order to allow incoming traffic to MySQL (3306 port)
Currently, my home IP address is being allowed in order to connect to MySQL server from any database client like Workbench or MySQL cli.

If you want to allow additional locations to connecto to MySQL, you should add a rule here.

Also, [other Virtual Network rule name was defined](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L405) in order to allow communication from `aks-subnet` network to MySQL

If you add a new subnet, you should add it into a virtual rule name here.

### 4.4. Azure Container Registry.

An ACR repository was created in order to store the customize Wordpress images to be deployed.
The following information is useful to create a new repositories inside ACR and also if you want to modify the 
`Dockerfile`
This repository was created of the following way:

- You should go to the `sentia-assesment/WordpressSite` path. You will find there the `Dockerfile`.

- Building the image.

```
~/projects/sentia-assesment/WordpressSite · (master±)
⟩ docker build -t customize_wordpress:5.2.4 .
```
It creates the `customize_wordpress:5.2.4` image

- Creating tag: We need to create a tag in order to upload  our image to our ACR
We have to include in the tag our ACR host server, in my case `wordpresssentiaassessment.azurecr.io.azurecr.io` 
[Here](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli) for more information.
 
```
⟩ docker tag customize_wordpress:5.2.4 wordpresssentiaassessment.azurecr.io.azurecr.io/customize_wordpress:5.2.4
```

- Push the image: Please, we have to be sure of use our complete tag created in the immediately previous step, I mean `wordpresssentiaassessment.azurecr.io.azurecr.io.azurecr.io/customize_wordpress:5.2.4`

```
⟩ docker push wordpresssentiaassessment.azurecr.io/customize_wordpress:5.2.4
The push refers to repository [registryname.azurecr.io/customize_wordpress]
b63469233da6: Pushed 
b032b61b15b2: Pushed 
b67d19e65ef6: Pushed 
5.2.4: digest: sha256:dc62844f946a49f2e724fa38bad6e2cab73a4561b22b690876ab5534febd3569 size: 5128
```
This process will create the `customize_wordpress:5.2.4` image. 

![alt text](https://cldup.com/hGH6HnMsQT.png "ACR Wordpress repository")

Is from this repository that the Wordpress helm chart are going to pull the `customize_wordpress:5.2.4` image inside Kubernetes environment.

This process is an essential part of the Continous Integration steps and it will be automated in the Build pipeline.

### 4.5. Modifying the Wordpress Helm chart.

In the `Deployments/Kubernetes/HelmCharts/wordpress-mine` path there are the Wordpress helm chart structure directory. 
Please, keep in mind the following code sections in order to point to the Wordpress image:

- `Deployments/Kubernetes/HelmCharts/wordpress-mine/values.yaml` file: General values to be deployed. You should modify the following parameters:
  - `repository`: The URL repository and image tag to pull the image. [Check it here](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/Kubernetes/HelmCharts/wordpress-mine/values.yaml#L9)
  - `tag`: The tag defined when the Wordpress image was created. Check it here](https://github.com/bgarcial/sentia-assessment/blob/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/Kubernetes/HelmCharts/wordpress-mine/values.yaml#L10)

**IMPORTANT**

Please keep in mind that this is a basic Wordpress Helm chart created locally using the `helm create ...` command.
I encourage to myself to use the official [Wordpress Helm chart](https://github.com/helm/charts/tree/master/stable/wordpress), clone it and customize it, in order to take in advance of all the Wordpress configuration parameters.
(You can also add all those parameters to the local Helm chart created, but is better use the official Helm chart configuration)

I was [starting to customize this official Helm chart](https://github.com/bgarcial/sentia-assessment/tree/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/Kubernetes/HelmCharts/wordpress), but due to time reasons I had to continue using the [local basic Helm chart created](https://github.com/bgarcial/sentia-assessment/tree/e581ca122f4da16de90f5576234d2bc9fc70bc00/Deployments/Kubernetes/HelmCharts/wordpress-mine).


---

## 5. PART 2 - CICD 

This is the CI/CD approach purposed

![alt text](https://cldup.com/AzqVHuvSjZ.jpg "Kubernetes Behavior and CI/CD")

**You cans see this K8s CI/CD behavior as a bigger picture [here](https://cldup.com/AzqVHuvSjZ.jpg)**


Currently, I have a Build and Release Pipeline automated from Azure DevOps.

The Build pipeline contains the activities that impact on the general artifact
The Release pipeline contains the activities that impact on the environment (accp, testing, development, production)

I decided the following:

## 5.1 Continous Integration process

Here will be only activities related to the Wordpress application sites upgrade process, via Docker and finally
I expose the Wordpress Helm chart as an artifact created.

In the CI process I am getting the source code from [github repository](https://github.com/bgarcial/sentia-assessment/)
code project.

![alt text](https://cldup.com/qV0zlvXKsm.png "Kubernetes Behavior and CI/CD")

In the CI approach will be all the activities related with the creation and uploading  Wordpress Docker image to ACR
[These activities](https://github.com/bgarcial/sentia-assessment/blob/master/Documentation/README.md#44-azure-container-registry), in order to automatize the continuous integration every time the client updates any of their WordPress applications in GIT. 

So the CI build pipeline has enabled the continuous integration trigger:

![alt text](https://cldup.com/CkTlNzeAtl.png "CI Trigger in the build pipeline")

### 5.1.2 IMPORTANT NOTE:

The CI/CD workflow in this purpose is strongly related with Helm tool, which allow us to deploy applications inside 
Kubernetes.

I decide to use Helm3 due to [some security concerns](https://engineering.bitnami.com/articles/helm-security.html) about helm 2 version. 

Currently [Helm 3 is in Release Candidate 2 version](https://github.com/helm/helm/releases/tag/v3.0.0-rc.2)
At the moment of come across in this assignment the latest helm 3 available version was [Helm v3-beta5](https://github.com/helm/helm/releases/tag/v3.0.0-beta.5) released 15 days ago. 
This CI/CD approach will work with this version, despite there is a more recently.

Azure DevOps, the CI/CD platform used, does not have compatible tasks with Helm3, only with Helm 2.

According to this, I decided to install helm3 inside the Azure DevOps build agents via bash script files and use helm3 via bash tasks.
From bash tasks (in azure devops) we can navigate through the [Wordpress Helm chart structure directory defined](https://github.com/bgarcial/sentia-assessment/tree/master/Deployments/Kubernetes/HelmCharts/wordpress-mine) in the release pipeline. 


## 5.2 Continous Delivery process

Such as I mentioned, is a the continious delivery process where the environment related activities must be executed, that's why I decided to divided in two categories the release pipeline process
 
### 5.2.1 Infrastructure Deployment Release Pipeline

This CD section is given by the ARM template execution.
I am using a [resource group deployment scope](https://docs.microsoft.com/en-us/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-3.0.0).

ARM templates require we supply the name resource group where we want to deploy the resources, as a part of the 
deployment command. 

#### IMPORTANT
If we want to execute the ARM template from Power shell the resource group must to exist before running the deployment.
[Here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/README.md) some preparation Power shell execution of the ARM template used, before to execute it from the release pipeline.
If you want to try the local Power shell approach, [you must to Install the Azure Power Shell environment](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-2.8.0) 

Is not the objective of the CD process the local and manual execution of the ARM templates from Powershell

- I am using n Azure Resource Group deployment task to deploy from Azure DevOPs the ARM template.
  Let's have a look to the task.

#### Pre Requisites for ARM template execution

The [ARM template to execute](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json) use a service principal to connect with Azure cloud and create the Kubernetes cluster. Actually, this service principal data adquisition process is not being automated from the ARM template, so will be necessary create a service principal of a manual way and reference in the ARM template their `servicePrincipalClientId` and `servicePrincipalClientSecret` data.

This service principal creation process should be automated from the ARM template.

- Creating the `SentiaAssessment` Service Principal using azure cli command

```
az ad sp create-for-rbac --role contributor -n "SentiaAssessment"
Changing "SentiaAssessment" to a valid URI of "http://SentiaAssessment", which is the required format used for service principal names
{
  "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "displayName": "SentiaAssessment",
  "name": "http://SentiaAssessment",
  "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
} 
```
You have to keep in mind the `appId`, and `password`  data. Those data and `tenant` are sensitive data. 

I created the `SentiaAssessment` service principal with the `create-for-rbac` and `--role contributor` flag.
[`create-for-rbac`] create the SP with a default role assignment
`--role contributor`is a contributor assignment.
You can see [here more information](https://docs.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac)


- Take the `appId`, and `password`  data previously mencioned and enter them in the release pipeline as a **pipeline 
variables**.
![alt text](https://cldup.com/RvtgB2KGXf.png "Infrastructure Release Pipeline")

- Give a name to the task and select a subscription to use to deploy the resources.
  ![alt text](https://cldup.com/6_Ew8e4MRx.png "Infrastructure Release Pipeline")

  The **Azure Personal subscription** selected, is an Azure Resource Manager service connection created previously [of this way](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml)

![alt text](https://cldup.com/Oa3ZofKGs1.png "Infrastructure Release Pipeline")

It was created using the service principal created previously, but with specific secrets to this service connection.
You can try it out [checking this link](https://4bes.nl/2019/07/11/step-by-step-manually-create-an-azure-devops-service-connection-to-azure/)

- In **Action** section is necessary to choose  *Create or update resource group* option
  ![alt text](https://cldup.com/0bKdIY-rtF.png "Infrastructure Release Pipeline")
  This *Create or update resource group* will create the resource group if it does not exist, otherwise it will update
  the resources in.

  **You can see this Infrastructure Release Pipeline picture in a bigger size [here](https://cldup.com/0bKdIY-rtF.png)**


- Choose the **resource group name** and **location** where the resources will be deployed.
 ![alt text](https://cldup.com/qismuYmXRu.png "Infrastructure Release Pipeline")
  
  I am passing the resourceGroup pipeline variable to get the resource name to create here
  
  ![alt text](https://cldup.com/1rp1HFfQf7.png "Infrastructure Release Pipeline")

- In **Template Location** choose *Linked Artifact* and after that choose the ARM template PATH
  ![alt text](https://cldup.com/VX1TpV2Th6.png "Infrastructure Release Pipeline")
  The ARM template to execute is [this](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L356)
  `sentia-assessment/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json`
  
  ![alt text](https://cldup.com/2K-05DcHYZ.png  "ARM Template to execute")

   **You can see this Infrastructure Release Pipeline picture in a bigger size [here](https://cldup.com/2K-05DcHYZ.png)**

- In **Overrride template parameters** you should see empty `servicePrincipalClientId` and `servicePrincipalClientSecret` parameters.     
  ![alt text](https://cldup.com/56FbpTnSXg.png  "ARM Template parameters")

  Add the pipeline variables there of this way: `$(servicePrincipalClientId)` and `$(servicePrincipalClientSecret)`

  ![alt text](https://cldup.com/EW8otyypTk.png  "ARM Template parameters")

  Do the same with `$(administratorLogin)` and `$(administratorLoginPassword)` which will be a privilege user for
  MySQL database. A privilege user database and password will be created from the template. 
  
  ![alt text](https://cldup.com/JmfOf8-vO7.png  "ARM Template parameters")
  
  
  Of course you must create  `administratorLogin` and `administratorLoginPassword` as a pipeline variables previously
  
  ![alt text](https://cldup.com/EwWP5TKr83.png  "ARM Template parameters")

  And at the end enter *Testing* as a value for environmentName ARM template parameter and click OK
  
  ![alt text](https://cldup.com/8hX8lQQaeh.png  "ARM Template parameters")

Just for double check, the complete parameters flags or string placed finally here are these

```
-location "West Europe" -vnetName "AssessmentVNetTesting" -vnetAddressPrefix "10.0.0.0/8" 
-subnet1Prefix "10.240.0.0/16" -subnet1Name "aks-subnet" -subnet2Prefix "10.241.0.0/27" 
-subnet2Name "persistence-subnet" -k8s_cluster_name "WordpressSentiaAssessment" 
-kubernetesVersion "1.14.7" -nodeCount 4 -agentVMSize "Standard_D2_v2" 
-servicePrincipalClientId $(servicePrincipalClientId) 
-servicePrincipalClientSecret $(servicePrincipalClientSecret) -serviceCidr "100.0.0.0/16" 
-dnsServiceIP "100.0.0.10" -dockerBridgeCidr "172.17.0.1/16" 
-serverName "WordpressSentiaAssessment" -administratorLogin $(administratorLogin) 
-administratorLoginPassword $(administratorLoginPassword) -skuCapacity 2 -skuName "GP_Gen5_2" 
-SkuSizeMB 5120 -SkuTier "GeneralPurpose" -skuFamily "Gen5" -mysqlVersion "5.7" 
-storageAutoGrow "Disabled" -backupRetentionDays 7 -geoRedundantBackup "Disabled" 
-virtualNetworkRuleName "AllowAksSubnet" -registryName "WordpressSentiaAssessment" 
-registrySku "Standard" -adminUserEnabled true -environmentName Testing
```


- Choose *Complete* value in the **Deployment mode**
  
  ![alt text](https://cldup.com/HU0FH5VZcG.png  "ARM Template parameters")

  The Complete mode, deletes the existing resources which weren't defined in the ARM template, so if the resource group already do exist and has resources not defined in the ARM template, those resources will be deleted.
  This is a mechanism to keep the resources creating using the Infrastructure as Code approach.

  ![alt text](https://cldup.com/5h9uNOMJfK.png  "ARM Template parameters") 
  
- In **Deployment Name** I am using the `$(Release.ReleaseId)` pre-built variable.
  ![alt text](https://cldup.com/uIXOCxBRA0.png  "Deployment Name") 

  This will allow me to reference the Deployments on the resource group on Azure portal by its ID
  ![alt text](https://cldup.com/zW0ZVO7him.png  "Deployment Name")
  
  You can also use `$(Release.ReleaseName)` 
  
- Finally you should save the options selected and **Create** the release.
  
  You will see that the release pipeline start and the ARM template is executed

  
  ![alt text](https://cldup.com/r97BoTH-oF.png  "ARM Template Deployment")

  **You can see this ARM Template Deployment picture in a bigger size [here](https://cldup.com/r97BoTH-oF.png)**

  After 9 minutes and 2 seconds all the resources definde in the ARM Template were deployed.

  ![alt text](https://cldup.com/Dkizc-EzW4.png  "ARM Template Deployment")

  **You can see this ARM Template Deployment picture in a bigger size [here](https://cldup.com/Dkizc-EzW4.png)**

  So you can see in Azure portal the `sentia-assessment` resource group was created and the AKS HA cluster, MySQL, ACR and Vnets and subnets as well.
  ![alt text](https://cldup.com/7E_7jeZwBU.png  "ARM Template Deployment")
  
  If you go to **Deployments** inside `sentia-assessment` resource group, you can see the Deployment ID. 
  
  ![alt text](https://cldup.com/-rcT_R7g_4.png  "ARM Template Deployment")


### 5.2.2 Wordpress Deployment Release Pipeline

The Wordpress aplication is deployed here, that's why all the activities performed are related with Kubernetes.

In order to getting it up and running, is necessary to deploy several additional services, referenced in the following activities/steps:

#### Creating Pipeline Variables.
The following pipeline Variables are created
- `helmVersion`, is the version of helm used. I am using the v3.0.0-beta.5. Please read [important considerations about helm3](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#512-important-note)
- `kubernetesCluster`, is the name of the Kubernetes cluster deployed from the ARM template [just right here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L324)
- `resourceGroup` and `resourceGroupDev` are the names of the resource group which were created in the Infrastructure Deployment. I will use just `resourceGroupDev` here. 
  - The other one (`resourceGroup`) is used when I want to replicate the deployment at different infrastructure deployment made.


![alt text](https://cldup.com/rotaWbiPFB.png  "ARM Template Deployment")

- #### Creating DevelopmentRelease Variable groups

Variable groups are very useful when we want to group information that is closely related with some subject
In this case, like in the release activities goes everything that impact to a specific environment, I am creating
a variable group for Wordpress Deployment Development Environment.
The following variables were defined:

- `acr_repository`, is the URL repository created previously in [Azure Container Registry activities](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#44-azure-container-registry)
- `azure_tenant`, is the directory ID of my Azure Active Directory. From it I can manage access to multiple subscriptions. This `azure_tenant` is necessary to get it to login from `az login` using a service principal. 
- `database_host`, is the MySQL database hostname given from the ARM template [here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L376)
- `database_name`, is the database which will be created in this release pipeline and it will store the Wordpress site tables.
- `database_user` and `database_password`, this is the password used to access to MySQL database server. It was given from the ARM template parameter [here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L387) and its value was assigned during the [Infrastructure release pipeline execution](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#pre-requisites-for-arm-template-execution) in the **Overrride template parameters** section.
Find out `administratorLogin`(administrator username) and `administratorLoginPassword`(administrator password) pipeline variables.
Those admin credentials are used to connect with MySQL host database from MySQL release pipeline tasks and create the wordpress database, its user and assign it permissions. 

- image_tag, is the tag given to the Wordpress image uploaded to the ACR in [Azure Container Registry activities](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#44-azure-container-registry)

- `new_db_user_to_be_created` and `new_db_passwd_user_to_be_created` are the new user and password created during this release pipeline execution. This credentials are used to connect with Wordpress database (`database_name` parameter above) when the Wordpress is installed. 
- 
- `service_principal_sentia_assessment_app_id` and `service_principal_sentia_assessment_password` are the service principal credentials, which were created in the [Pre Requisites for ARM template execution](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#pre-requisites-for-arm-template-execution).
Find out the **Creating the SentiaAssessment Service Principal** section  
- `wordpressUsername`, `wordpressPassword` and `wordpressEmail` are the data to access to the Wordpress instance installed.

#### Wordpress Deployment Release pipeline tasks

The steps addressed here will be explained:

- **Creating `wordpress-site-*`,`nginx` and `cert-manager` namespaces**

All of them are executed via Kubernetes task (interacting with `kubectl` command tool)


![alt text](https://cldup.com/bxU720YcLn.png "Kubernetes namespace")

You have to be sure to choose the following options highlighted in red rectangles: 
**create** option as a command
`namespace cert-manager` as an arguments
In **Advanced** section the `kubectl` tool is downloaded. Choose 1.14.7, which is the current kubernetes version given to the cluster from the [ARM template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L63)
In **Working Directory** enter  `$(System.DefaultWorkingDirectory)` prebuilt variable
In **Output format** choose YAML option. 

![alt text](https://cldup.com/E-6uP2Hj6k.png "Kubernetes namespace")

- **Login to ACR**
Is the access to the Azure Container Registry created from the [ARM template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L440)

![alt text](https://cldup.com/AQRx-0sW6b.png "Login to ACR")

It's a docker task using the `docker login` command and connecting to my ACR via **Azure Container Registry**
service connection that I created previously of this way:

![alt text](https://cldup.com/tBbtzPob_3.png "Login to ACR")


- Getting Ready Environment to execute Deployments
This is a bash script task file and I am doing the following here:
```

# Stop on error. Bash process will stop and highlight some error.
set -e

# ownloading and installing latest beta helm3 version on azure devops agent machine. 
wget https://get.helm.sh/helm-v3.0.0-beta.5-linux-amd64.tar.gz
pwd
# Unpacking latest beta helm3 version
tar -xzf helm-v3.0.0-beta.5-linux-amd64.tar.gz
./linux-amd64/helm version

# Login to azure from az cli tool using SentiaAssessment Service principal
az login --service-principal -u $(service_principal_sentia_assessment_app_id) -p $(service_principal_sentia_assessment_password) --tenant $(azure_tenant)

# Getting K8s azure cluster credentials
az aks get-credentials --resource-group $(resourceGroupDev) --name $(kubernetesCluster)

# Creating a symbolic link to use helm3 command in anywhere on agent machine
echo "Show ls command"
ls  /home/vsts/work/r1/a/linux-amd64/helm
#helm create release
cd /usr/bin
echo "Creating a symbolic link to make helm3 wide system scope"
sudo ln -s /home/vsts/work/r1/a/linux-amd64/helm helm3 
echo "Symbolic link created"
pwd
cd
echo "Now in root path directory"
helm3 version
# Initializing and update helm chart repository - https://v3.helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository
helm3 repo add stable https://kubernetes-charts.storage.googleapis.com/
helm3 repo update
```
- **Execute Azure MySQL : SqlTaskFile  - Creating Wordpress Database**
The database for the Wordpress site is created here

![alt text](https://cldup.com/9-BtBqIUhN.png "Creating Wordpress database")

Here we are using most of the variable created in the DevelopmentRelease variable group

![alt text](https://cldup.com/nIKuvgNiee.png "Variable group") 

- **Creating User inside Wordpress database and grant it permissions**

Here a wordpress site user database is created and I assigned permissions to the database
created in the previous step

![alt text](https://cldup.com/LtqpR5lhwM.png "Variable group")

- Deploying Wordpress Helm chart in `wordpress-site-*` namespace

Here my own Wordpress Helm chart is deployed via the following bash script:

```
# Testing helm3 command
echo 'Hello Helm'
helm3 version

# Knwing current path
echo 'Current directory:'
pwd

echo 'Content Current directory:'
echo $(System.ArtifactsDirectory)

# Entering to the Project root directory
echo "Entering to project"
cd ./GithubSentiaAssesment/
pwd
ls

# Deploying Wordpress Helm chart
helm3 install wordpress-site-8 ./Deployments/Kubernetes/HelmCharts/wordpress-mine \
      --set image.repository=$(acr_repository) \
      --set image.tag=$(image_tag) \
      --set image.pullPolicy=Always \
      --set wordpressUsername=$(wordpressUsername) \
      --set wordpressPassword=$(wordpressPassword) \
      --set wordpressEmail=$(wordpressEmail) \
      --set mariadb.enabled=false \
      --set externalDatabase.host=$(database_host) \
      --set externalDatabase.user=$(new_db_user_to_be_created) \
      --set externalDatabase.password=$(new_db_passwd_user_to_be_created) \
      --set externalDatabase.database=$(database_name) \
      --set externalDatabase.port=3306 \
      --set wordpressScheme=https \
      --set replicaCount=1 \
      --namespace site-8 

echo 'Wordpress Helm chart installed'

```

![alt text](https://cldup.com/JbMA-tU_PO.png "Wordpress Environments Release Pipeline")

**You can see this kubernetes behavior picture in a bigger size [here](https://cldup.com/JbMA-tU_PO.png)**

- **Installing NGINX Ingress controller helm chart in `nginx` namespace**

Here, nginx ingress controller helm chart is deployed via this bash script.
`helm3 install nginx-ingress stable/nginx-ingress --namespace nginx`

![alt text](https://cldup.com/qgNlGGEug3.png "NGINX from helm chart")


- **Cert-Manager Deployment**
  
  - **Installing Cert-manager custom resources definitions**
    cert-manager uses some custom resources in order to work with certificates, clusterissuers and orders inside kubernetes environment. 
    Basically this command is executed in this kubernetes task
    ```
    # Install the CustomResourceDefinition resources separately
    kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml
    ```

    ![alt text](https://cldup.com/2ihu_FzdGC.png "Executing Cert Managers CRDs")

  - **Installing Cert Manager helm chart in `cert-manager` namespace.**
    Here, cert-manager helm chart is deployed via this bash script.

    ```
    echo 'Deploying Cert Manager'
    # Adding cert-manager repository and updating helm3 index
    helm3 repo add jetstack https://charts.jetstack.io
    helm3 repo update

    # Installing cert-manager helm chart
    helm3 install cert-manager --namespace cert-manager --version v0.11.0 jetstack/cert-manager

    echo 'Cert Manager Controller installed'
    ```

    ![alt text](https://cldup.com/nbfDKwtYfb.png "NGINX from helm chart")

  - **Installing Staging and Production ACME Cluster Issuers to contact to Let'sEncrypt ACME CA**
    Cert-Manager setup contact with certificate authorities in order to get the TLS certificates to the
    https protocol. I choose to contact to Let'sEncrypt, which has Staging and Production servers to create
    certificates. 
    Cert-Manager contact to Let'sEncrypt via `ClusterIssues` resources, so here those are created.
    The YAML manifest to `StagingClusterIssuer` is:
```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: botibagl@gmail.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource used to store the account's private key.
      name: letsencrypt-staging
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: nginx
```
The YAML manifest to `ProductionClusterIssuer` is:
```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: botibagl@gmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource used to store the account's private key.
      name: letsencrypt-prod
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: nginx
```
Those YAML files are being executed in these tasks here

  ![alt text](https://cldup.com/xzo3RmVU_9.png "NGINX from helm chart")

- Creating Ingress resource to expose the Wordpress service and get https protocol to it.

And now, is necessary to create an Ingress resource to expose the Wordpress helm chart service deployed previously to Internet.

The `Site8Ingress.yaml` file was create here:

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    # Contacting directly to Let'sEncrypt production environment
    cert-manager.io/cluster-issuer: letsencrypt-prod
    # Contacting to Nginx Ingress controller
    kubernetes.io/ingress.class: nginx 
  name: wordpress-site-8-ingress
  namespace: site-8
spec:
  rules:
  - host: site8.bgarcial.me
    http:
      paths:
      - backend:
          serviceName: wordpress-site-8
          servicePort: 80
        path: /
  tls: # < placing a host in the TLS config will indicate a cert should be created
  - hosts:
    - site8.bgarcial.me
    secretName: letsencrypt-prod-site-8
```

This ingress resource is being executed here:

![alt text](https://cldup.com/e00yN8k3Fe.png "Exposing Ingress for site8.bgarcial.me")


All those activities are being deployed from the release pipeline with just pushing a button.

![alt text](https://cldup.com/lzujgCN9wI.png "Wordpress Environments Release Pipeline")


- So, when this Release pipeline is executed, we can see the following:

![alt text](https://cldup.com/zGwSE-0k_8.png "Wordpress Environments Release Pipeline")

And all the resources were deployed in the Kubernetes cluster


NGINX Ingress Controller
![alt text](https://cldup.com/7QuJsnpEvI.png "Ingress Controller")

Cert-Manager Namespace
![alt text](https://cldup.com/6ZQJlPMQGl.png "Cert-Manager Namespace")

Cert-Manager CRDs and clusterissuers 
![alt text](https://cldup.com/vtP4cfYQ_q.png "Cert-Manager CRDs and clusterissuers")

And here an interesting part, the Wordpress helm chart deployed in order to get the
[site8.bgarcial.me](https://site8.bgarcial.me/) site up and running

![alt text](https://cldup.com/alUcKGLcKX.png "Cert-Manager CRDs and clusterissuers")

I only have 1 pod, this is because in the Wordpress helm chart [by default the replicas is one](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress-mine/values.yaml#L5) 

You can modify there the number of `replicaCount` or you can also add the `--set replicaCount=1` parameter in the `helm install` command.

### What about the Wordpress site?

Ok, I am going to adress somethings here.

- If we go to [site8.bgarcial.me](https://site8.bgarcial.me/), yes we got https for the domain.
  
![alt text](https://cldup.com/TYtJ17Gize.png "HTTPS protocol")

But currently I have a problem with consume the static files in my Wordpress deployment. [In the Issues and Risk Mitigation section I will refer to this](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#6-issues-and-risk-mitigation)

So Lets continue with the wordpress installation


![alt text](https://cldup.com/1drqYBne4E.png "Wordpress installation")

Giving the database name given in the variable groups `database_name` which was created in the release pipeline
Giving the database password given in the variable groups `new_db_passwd_user_to_be_created` 
Giving the database HOST given in the variable groups `database_host`  

![alt text](https://cldup.com/YCeknXDTyy.png "Wordpress installation")

![alt text](https://cldup.com/0ZPufaviYM.png "Wordpress installation")

Click in Run installation and I will need to provide some site information.

`Username` is the `wordpressUsername` variable given in the variable groups
`Password` is the `wordpressPassword
` variable given in the variable groups
![alt text](https://cldup.com/k3Nx7OHv_6.png "Wordpress installation")

And the `wordpressEmail` (which is missing here)

When we press Install Wordpress button


![alt text](https://cldup.com/3gZKHIrrtQ.png "Wordpress installation")

![alt text](https://cldup.com/orTHBQq_Tz.png "Wordpress installation")

These latest steps about Wizard Web Interface Wordpress Installation could be automated.
The official Wordpress Helm chart [that I had started to customize](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress/values.yaml) has those attributes
in order to be include in the  `sentia-assessment/Deployments/Kubernetes/HelmCharts/wordpress/values.yaml`

- `wordpressFirstName`: [Here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress/values.yaml#L57)
- `wordpressLastName`: [Here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress/values.yaml#L62)
- `wordpressBlogName`: [Here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress/values.yaml#L67)
- `wordpressTablePrefix`: [Here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress/values.yaml#L72)
- And the data [to contact to the external database](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress/values.yaml#L148) 


---

## 6. Issues and Risk Mitigation

Here the technical issues encountered in this soluton approach are documented.
 
### 6.1  Wordpress sites are not able to serve stylesheets and other assets files.

- **Name**: Deployed Wordpress sites are not able to serve static files
- **Id**: 0001
- **Description**: 

I cannot see the stylesheets and other assets files from [site8.bgarcial.me](https://site8.bgarcial.me/).

When I use Inspect element in the browser I got a kind of mixed content problem:


![alt text](https://cldup.com/J6264-J20F.png "Wordpress installation")

Let's see the errors here:

```
(index):13 Mixed Content: The page at 'https://site8.bgarcial.me/' was loaded over HTTPS, 
but requested an insecure script 'http://site8.bgarcial.me/wp-includes/js/wp-emoji-release.min.js?ver=5.2.4'. 
This request has been blocked; the content must be served over HTTPS.
f @ (index):13
```
![alt text](https://cldup.com/pHiQcwQae7.png "Wordpress installation")

According to this log console looks like the Wordpress normally get the style sheets and assets files via http and not by https.

But If I take the scripts URL that Wordpress site has requesting in the image above (all URLs squared in blue color ), 
I can achieve them, under https protocol, the `.js` and `.css` files.

https://site8.bgarcial.me/wp-includes/js/wp-emoji-release.min.js?ver=5.2.4
https://site8.bgarcial.me/wp-includes/css/dist/block-library/style.min.css?ver=5.2.4
https://site8.bgarcial.me/wp-includes/css/dist/block-library/theme.min.css?ver=5.2.4
https://site8.bgarcial.me/wp-content/themes/twentynineteen/style.css?ver=1.4
https://site8.bgarcial.me/wp-content/themes/twentynineteen/print.css?ver=1.4
https://site8.bgarcial.me/wp-includes/css/dist/block-library/style.min.css?ver=5.2.4
https://site8.bgarcial.me/wp-includes/css/dist/block-library/theme.min.css?ver=5.2.4
https://site8.bgarcial.me/wp-includes/js/wp-embed.min.js?ver=5.2.4


Maybe it could be because when I install my own Wordpress Helm chart, I included the `--set wordpressScheme=https`
and also from my Wordpress helm chart I have this `https` value [by default](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress-mine/values.yaml#L16)

But, also in our architecture we have Apache web server inside the docker wordpress image running in the
helm chart, and we have Nginx as an ingress controller.

Some analysis here is that nginx send the request using https, but looks like Apache are not able to understand https.

- **Criticallity**: It's a very critical issue.

Let's agree on the section name and subsections:
* Name: Risks mitigation;
   * Table of risks:
       Header: Id|Name|Description|Criticallity
   * Table of mitigations:
       Header: Id|Status<Open, 
       Closed>|Mitigation status description.

#### 6.1.1 Possible mitigations. 

- **Current Status**: Open.


- **Mitigation status description**:

Ok let's go back to:
Some analysis here is that nginx send the request using https, but looks like Apache are not able to understand https.   
I tried to enable a `RewriteEngine` rule  in Apache web server when I build the `Dockerfile` image [here](https://github.com/bgarcial/sentia-assessment/blob/master/WordpressSite/Dockerfile#L14)

As you can see, in the `Dockerfile` I am overriding the `/etc/apache2/apache2.conf` file adding the `RewriteEngine` [here](https://github.com/bgarcial/sentia-assessment/blob/master/WordpressSite/etc/apache2/apache2.conf#L181)

I taken this approach solution [from this link](https://aws.amazon.com/es/premiumsupport/knowledge-center/redirect-http-https-elb/)

But it does not work
There are other answers with different configurations on Apache [here](https://stackoverflow.com/questions/16200501/http-to-https-apache-redirection/16200545#16200545) which I tried several of them withot success.

But then, now I put my eye on the Load balancer that I have in the architecture.
And I found this article: [WordPress is_ssl() doesn’t work behind some load balancers](https://snippets.webaware.com.au/snippets/wordpress-is_ssl-doesnt-work-behind-some-load-balancers/)
They purpose a plugin to be installed via Wordpress admin, [this plugin](https://wordpress.org/plugins/ssl-insecure-content-fixer/).

I've installed the plugin, but the issue still persist. (I did it on site4.bgarcial.me domain in that time.)

![alt text](https://cldup.com/6VFBCsQ2DW.png "Wordpress installation")

Some forums say that it is a common problem when we have wordpress installation behind some load balancers, and I am in this side, and I think that I have to continue working in the `RewriteEngine` rule 

Maybe not in the `/etc/apache2/apache2.conf` but yes in `/etc/apache2/sites-available/000-default.conf`
Also maybe keep in mind the apache mod ssl actions ...

Also [this article](https://www.nginx.com/blog/9-tips-for-improving-wordpress-performance-with-nginx/) about how to improve performance to Wordpress applications with nginx looks interesting


Note: At the moment of I realize of this alternative actions that I describe to solve the problem, I had to 
stop to solve this issue in order to advance in this documentation process.

---

### 6.2  ErrImagePull when the Kubernetes pod try to consume the Wordpress customize docker image created

- **Name**: Kubernetes pod is not able to consume the Wordpress customize docker image created 
- **Id**: 0002
- **Description**: 
If you got an `ErrImagePull` message when you deploy the Wordpress Helm chart


![alt text](https://cldup.com/q8LZgGoo9G.png "Wordpress installation")


And if you describe the pod which is not able to pull the image, and if you got this message:

```
Error response from daemon: pull access denied for customize_wordpress, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
  Warning  Failed     2m20s (x4 over 3m39s)  kubelet, aks-defaultpool-34253081-vmss000001  Error: ErrImagePull
```

```
Normal   Pulling    2m21s (x4 over 3m40s)  kubelet, aks-defaultpool-34253081-vmss000001  Pulling image "customize_wordpress:5.2.4"
  Warning  Failed     2m20s (x4 over 3m39s)  kubelet, aks-defaultpool-34253081-vmss000001  Failed to pull image "customize_wordpress:5.2.4": rpc error: code = Unknown desc = Error response from daemon: pull access denied for customize_wordpress, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
  Warning  Failed     2m20s (x4 over 3m39s)  kubelet, aks-defaultpool-34253081-vmss000001  Error: ErrImagePull
  Normal   BackOff    114s (x6 over 3m38s)   kubelet, aks-defaultpool-34253081-vmss000001  Back-off pulling image "customize_wordpress:5.2.4"
  Warning  Failed     103s (x7 over 3m38s)   kubelet, aks-defaultpool-34253081-vmss000001  Error: ImagePullBackOff
```
- **Criticallity**: It's a very critical issue, but it was a casual issue. I will explain why.
  

#### 6.2.1 Mitigation action. 

- **Current Status**: Closed.

- **Mitigation status description**:
It's happens because by default when the Azure Container Registry was deployed from the ARM template, the service principal used to create the AKS service does not had permissions to pull images over that Azure Container Registry.

I didn't realize that I had to associate the aks cluster with the Azure container registry.

Microsoft documentation say it how to do it [here](https://docs.microsoft.com/en-us/azure/aks/cluster-container-registry-integration#configure-acr-integration-for-existing-aks-clusters)  

And then before to deploy my own Wordpress helm chart application, in my case from `helm install` command (referenced before in the Wordpress Deployment release pipeline section), is necesary to integrate my existing ACR with my existing AKS cluster 


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

#########   PLEASE PAY ATTENTION TO THIS SECTION WHERE THE SP SentiaAssessment IS INVOLVED ###########################

  "nodeResourceGroup": "MC_sentia-assessment-_WordpressSentiaAssessment-aks_westeurope",
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


So, when I install my helm chart, the image is up and running

```Events:
  Type     Reason     Age                   From                                          Message
  ----     ------     ----                  ----                                          -------
  Normal   Scheduled  14m                   default-scheduler                             Successfully assigned default/wordpress-site-4-6565b8c64f-w7xvq to aks-defaultpool-34253081-vmss000000
  Normal   Pulling    14m                   kubelet, aks-defaultpool-34253081-vmss000000  Pulling image "registryname.azurecr.io/customize_wordpress:5.2.4"
  Normal   Pulled     14m                   kubelet, aks-defaultpool-34253081-vmss000000  Successfully pulled image "registryname.azurecr.io/customize_wordpress:5.2.4"
  Normal   Created    14m                   kubelet, aks-defaultpool-34253081-vmss000000  Created container wordpress

```
If we do that, we don't need to reference the docker registry secret and any `imagePullSecrets` attribute in our helm chart or yaml file, our image will be pulled without reference any credentials

**And now, some inshights (in addition to the output above from the `az aks update` command)**

I started to fins the way to setup the association between my AKS cluster and the ACR from the ARM template, but looks like [it isn't possible](https://github.com/MicrosoftDocs/azure-docs/issues/39508) 

Microsoft guys say:

>Cannot be done in an ARM template because this is a CLI client side change only. 
We enabled this in the CLI for customers who find it hard to understand ARM RBAC. 
The simple implementation behind the scenes is a specifically a simple role assignment allowing AKS to access 

So  that role assignment is performed over the service principal used to create the AKS cluster from the arm template
so indeed, and then when I going to my `WordpressSentiaAssessment` container registry options, I can see that
the `az aks update` command executed above, has created the `acrPull` role assignment to the Service Principal `SentiaAssessment` 



![alt text](https://cldup.com/j6pHyRreZm.png "acrPull role")


### ADDITIONAL NOTES:

Trying to solve [the style sheets and assets files over https issue](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#61--wordpress-sites-are-not-able-to-serve-stylesheets-and-other-assets-files), I was
getting into in the Wordpress helm chart parameters, and I realize that in my own Wordpress helm chart was creating 
 a service account in Kubernetes [here](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress-mine/values.yaml#L21) 
 
So, when I realize abut it [I put it to false](https://github.com/bgarcial/sentia-assessment/commit/41e58b7c2c4a7e73617e55cf4673cf9c45ab4eb0), because the the official Wordpress helm chart (I just clone it from Github's helm charts ) [they don't do that](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/Kubernetes/HelmCharts/wordpress/values.yaml)

That service account creation detail leads me to think that maybe it was interfering with the pull of my image. 

I cannot to be entirely sure of this, but I deployed a new infrastructure from scratch and I executed my own Helm chart and the wordpress image was pulled of a successfully way, without I had to setup the association between the als cluster and ACR documented here in this issue.

### SOME HINT TO TRY IN THE FUTURE 

These comments here could be related with the first and most critical issue [presented here](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#61--wordpress-sites-are-not-able-to-serve-stylesheets-and-other-assets-files)

I want to highlight that actually in the github repository I have two Wordpress Helm charts you can see them [here](https://github.com/bgarcial/sentia-assessment/tree/master/Deployments/Kubernetes/HelmCharts)



![alt text](https://cldup.com/lli67aA-y8.png "Wordpress Helm charts")

- `wordpress`: My own Wordpress helm chart. Is with this chart I have been working to write this documentation process
- `wordpress-mine`: The official Wordpress Helm chart, it was cloned from the [official github site](https://github.com/helm/charts/tree/master/stable/wordpress)


So I decided to clone the official Wordpress helm chart and customize it, and not create a new one via `helm create` command, which it was that I had.
This decision is reflected [in this commit](https://github.com/bgarcial/sentia-assessment/commit/be69f770ca57f9375efdfd5807972d4f27cc8fc2) and currently that I am thinking, if I move to use the official Wordpress helm chart, maybe its can help me to customize better the Wordpress deployment and maybe I couldn't have problems like the pull error image and the https problem about my static files.

I say this, because in my own Helm chart `wordpress-mine` (which is actually running for this assessment), I don't have there many parameters that the official `wordpress` helm chart has, and also the structure of my own created helm chart is different to the official helm chart

--


### 6.3 Client with IP address '13.79.132.109' (Azure DevOps agent) is not allowed to connect to this MySQL server. 

- **Name**: Azure DevOps agent machine does not have access to MySQL managed service.
- **Id**: 0003
- **Description**: 
During the [Wordpress release pipeline](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#wordpress-deployment-release-pipeline-tasks) when the MySQL database tasks were executed, I got this message the first time

![alt text](https://cldup.com/gZbDiCGC-b.png "Azure DevOps agent machine does not have access to MySQL managed service")

- **Criticallity**: Low 

#### 6.3.1 Mitigations. 

- **Current Status**: Closed.
- **Mitigation status description**:

The Azure DevOps agent machine does not have access to MySQL managed service, so it was necessary to figure out its IP
address and white-listing it in the security settings in MySQL server and also from the [ARM template](https://github.com/bgarcial/sentia-assessment/blob/master/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json#L248).

But something particular happens when I try to figure out the Azure DevOps agent machine IP address

Each release executed by me, in each release, the Azure DevOps agent machine IP address was different, that's why I had to figure out what's going on with it.

Microsoft says [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops#agent-ip-ranges):

>In some setups, you may need to know the range of IP addresses where agents are deployed. 
For instance, if you need to grant the hosted agents access through a firewall, you may wish to restrict that access by IP address. 
Because Azure DevOps uses the Azure global network, IP ranges vary over time. We publish a weekly JSON file listing IP ranges for Azure datacenters, broken out by region. 
This file is published every Wednesday (US Pacific time) with new planned IP ranges. The new IP ranges become effective the following Monday. 
We recommend that you check back frequently to ensure you keep an up-to-date list. If agent jobs begin to fail, a key first troubleshooting step 
is to make sure your configuration matches the latest list of IP addresses.

The [weekly JSON file](https://www.microsoft.com/en-us/download/details.aspx?id=56519) listing IP ranges link looks like it does not exist because it redirects to a Microsoft download center website

In this [post question-answers](https://developercommunity.visualstudio.com/idea/467755/static-ip-address-for-azure-devops.html) I manage to detail the correct link to download the weekly JSON file IP addresses range for Azure DevOps.

It says:
>avatar image
Luis Ayala [MSFT]
May 15 at 10:39 PM
Hey check link below, is about IP ranges.
https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops#agent-ip-ranges
Regions
https://azure.microsoft.com/en-us/global-infrastructure/geographies/
>#### IS THIS 
>IP Ranges Document.
https://www.microsoft.com/en-us/download/confirmation.aspx?id=41653
You only need to add that subnets to your white list depending of your geography.
Regards.

So the correct link that point to the list of IP addresses ranges is [this](https://www.microsoft.com/en-us/download/confirmation.aspx?id=41653)

But I did prefere Allow access to MySQL server from Azure Services via Azure portal of this way:

https://cldup.com/3MKFO23xdO.png

![alt text](https://cldup.com/3MKFO23xdO.png "Allow access to MySQL server from Azure Services")

I did prefer this, because according to the documentation refered those IP addresses ranges are changing every week, so it was better to work with my release pipeline workflow.

---

## 7. About High Availability in my solution 

HA means different things to different people. I prefer to approach it with math, defining it numerically and based on facts.


A good inspirational reading about System Availability and Reliability you can find here: https://www.eventhelix.com/RealtimeMantra/FaultHandling/system_reliability_availability.htm


### 7.1. Azure Kubernetes Service

From Novemer 4th, in the Microsoft Ignite was announced that [the Availability Zone support for AKS](https://azure.microsoft.com/en-us/updates/azure-kubernetes-service-aks-support-for-azure-availability-zones-is-now-available/?utm_source=samcogan) has passed from preview to be considered now as an available feature in Azure

Talk about HA IN AKS in the Azure context, is talk about the availability of the agent nodes in the cluster. It is covered by the Virtual Machines SLA according to [this link](https://azure.microsoft.com/en-au/support/legal/sla/kubernetes-service/v1_0/).

In my case of the Virtual Machines Scale sets which support the nodes in the AKS cluster

- If I go to the
![alt text](https://cldup.com/7CQ3kO9dwp.png "K8s nodepool")

- If I go to the node-0. it was located in the **West Europe Zone 1.**

![alt text](https://cldup.com/pV1hhS5E6g.png "K8s node-0")

- If I go to the node-1. it was located in the **West Europe Zone 2.**

![alt text](https://cldup.com/IIhaFEzFiP.png "K8s node-1")

- The node-2 is in **West Europe Zone 1.**

![alt text](https://cldup.com/-0OJJmILjO.png "K8s node-2")

- And the node-3 is in **West Europe Zone 2.**

![alt text](https://cldup.com/X5o_UmdpJJ.png "K8s node-2")

Ok, this is not new, due to [in the node count configuration I defined that](https://github.com/bgarcial/sentia-assessment/tree/master/Documentation#node-count-is-the-number-of-nodes-in-the-node-pool-defined-to-k8s)

Even we can verify this node distribution listing the agent nodes in the cluster, filtering on the `failure-domain.beta.kubernetes.io/zone` ':

![alt text](https://cldup.com/eZu3BuU7AL.png "Verifying nodes distribution")

As we add additional nodes to an agent pool, the Azure platform automatically distributes the underlying VMs across the specified availability zones.

#### 7.1.2.SLA for Virtual Machine Scale Sets

Microsoft [says](https://azure.microsoft.com/en-us/support/legal/sla/virtual-machine-scale-sets/v1_1/):

>Virtual Machine Scale Sets is a free service, therefore, it does not have a financially backed SLA itself. 
However, if the Virtual Machine Scale Sets includes Virtual Machines in at least 2 Fault Domains, 
the availability of the underlying Virtual Machines SLA for two or more instances applies. 
If the scale set contains a single Virtual Machine, the availability for a Single Instance Virtual Machine applies.
See the Virtual Machines SLA for more details.

Ok I have two Virtual Machine Scale sets in each availability zone, so the availability for them is applied.

- Then, they refer to me to the **[SLA for Virtual Machines](https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/)**, it cover the Virtual Machine Scale sets.

Let's keep in mind some facts about it.

>For all Virtual Machines that have two or more instances deployed across two or more Availability Zones 
in the same Azure region, we guarantee you will have Virtual Machine Connectivity to 
at least one instance at least 99.99% of the time.

This condition apply for me, so, at least I would have one virtual machine running in any of my two availability zones defined, in case of failure.
The nine terms here in this case are two nines, i.e. **99.99%**

>For all Virtual Machines that have two or more instances deployed in the same Availability Set, 
we guarantee you will have Virtual Machine Connectivity to at least one instance at least 99.95% of the time.

This condition also apply for me, so If one availability zone goes down, I would have one virtual machine running at least in the remaining zone. The nine terms here 1 nine, close to 2 nines.  **99.95%**

>For any Single Instance Virtual Machine using premium storage for all Operating System Disks and Data Disks, 
we guarantee you will have Virtual Machine Connectivity of at least 99.9%.

I don't understand very well this condition, due to my default pool nodes on Kubernetes use this storage configuration 
type:

![alt text](https://cldup.com/xlZzUze5aL.png "K8s storage type")



