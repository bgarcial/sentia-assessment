I have an existing Virtual Network created with two subnets: `aks-subnet` and `persistence-subnet`.

My goal is to create an Azure Kubernetes Cluster inside the `aks-subnet`

I am creating resource groups and resources [at the subscription level][1], using the `New-AzDeployment` command from PowerShell core.

Like my idea is create a resource group and deploy resources to it, I have a nested template defining the resources to create in the resource group. 

So I have the resource group created from the ARM template

- **please find** this json definition in the entire template showed below at the end of this thread
```
"type": "Microsoft.Resources/resourceGroups",
```

And I am using a Deployment resource in order to nest the template that contains the resources that I want to create inside the resource group.

- **please find inmediately after:**
```
"type": "Microsoft.Resources/deployments"
```

So inside, this `Microsoft.Resources/deployments` I am creating a Vnet with the two subnets previously mentioned, 

**and It works!**, the Vnet and the subnet is created inside the resource group created in the same template.

- **Please find** in the entire template showed below at the end of this thread
```
"type": "Microsoft.Network/virtualNetworks",
``` 

Now I want to add an Azure Kubernetes cluster inside the `aks-vnet`.

- **Please find** in the entire template: 
```
"dependsOn": [
    "Microsoft.Network/virtualNetworks/AssessmentVNet"
],
"type": "Microsoft.ContainerService/managedClusters",
```

And then when I am associating the defaultpool (or virtual machine) to the `aks-subnet`, 

- **Please find** `vnetSubnetID` attribute of this way in the entire template showed below at the end of this thread:

```
"vnetSubnetID": "[resourceId(parameters('resourceGroupName'),'Microsoft.Network/virtualNetworks/subnets',parameters('vnetName'),parameters('subnet1Name'))]",
```
I am trying to access to the VnetSubnetID of this way, according to this [AKS advanced networking official link][2] suggest:

According to the immediately above I am doing here the following:

- I got the `resourceId` from the `resourceGroupName` where is located the Vnet.
- I am indicating the type of resource subnet `Microsoft.Network/virtualNetworks/subnets`
- And I am passing like parameters the name of the Vnet which have the subnet and the name of the subnet as well `parameters('vnetName'),parameters('subnet1Name'))`

But when I execute the template from Power shell I got the following error:

```
PS /home/bgarcial/projects/my-project/Deployments/ARMTemplates/ResourceGroup> New-AzDeployment `
>>  -Name SentiaAssessment `
>>  -location westeurope `
>>  -TemplateUri $templateUri `
>>  -resourceGroupName $resourceGroupName `
>>  -environmentName accp `
>>  -dnsPrefix WordpressSentiaAssessment-dns `
>>  -servicePrincipalClientId $servicePrincipalClientId `
>>  -servicePrincipalClientSecret $servicePrincipalClientSecret

New-AzDeployment : 10:20:02 PM - Resource Microsoft.Resources/deployments 'storageDeployment' failed with message '{
  

"error": {
    "code": "InvalidTemplate",
    "message": "Unable to process template language expressions for resource '/subscriptions/9148bd11-f32b-4b5d-a6c0-5ac5317f29ca/resourceGroups/sentia-assessment/providers/Microsoft.Resources/deployments/storageDeployment' at line '150' and column '9'. 'The provided value 'sentia-assessment' is not valid subscription identifier. Please see https://aka.ms/arm-template-expressions/#resourceid for usage details.'",
    "additionalInfo": [
      {
        "type": "TemplateViolation",
        "info": {
          "lineNumber": 150,
          "linePosition": 9,
          "path": ""
        }
      }
    ]
  }
}'
At line:1 char:1
+ New-AzDeployment `
+ ~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [New-AzDeployment], Exception
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.NewAzureDeploymentCmdlet
 
New-AzDeployment : 10:20:02 PM - Template output evaluation skipped: at least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.
At line:1 char:1
+ New-AzDeployment `
+ ~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [New-AzDeployment], Exception
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.NewAzureDeploymentCmdlet
 
New-AzDeployment : 10:20:02 PM - Template output evaluation skipped: at least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.
At line:1 char:1
+ New-AzDeployment `
+ ~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [New-AzDeployment], Exception
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.NewAzureDeploymentCmdlet
 

DeploymentName          : MyDeployment
Location                : westeurope


ProvisioningState       : Failed
Timestamp               : 10/23/19 8:19:57 PM
Mode                    : Incremental
TemplateLink            : 
Parameters              : 
                          Name                            Type                       Value     
                          ==============================  =========================  ==========
                          resourceGroupName               String                     sentia-assessment
                          location                        String                     West Europe
                          vnetName                        String                     AssessmentVNet
                          vnetAddressPrefix               String                     10.0.0.0/8
                          subnet1Prefix                   String                     10.240.0.0/16
                          subnet1Name                     String                     aks-subnet
                          subnet2Prefix                   String                     10.241.0.0/27
                          subnet2Name                     String                     persistence-subnet
                          k8s_cluster_name                String                     WordpressSentiaAssessment
                          kubernetesVersion               String                     1.14.7    
                          dnsPrefix                       String                     WordpressSentiaAssessment-dns
                          nodeCount                       Int                        3         
                          agentVMSize                     String                     Standard_D2_v2
                          servicePrincipalClientId        SecureString                         
                          servicePrincipalClientSecret    SecureString                         
                          serviceCidr                     String                     100.0.0.0/16
                          dnsServiceIP                    String                     100.0.0.10
                          dockerBridgeCidr                String                     172.17.0.1/16
                          environmentName                 String                     accp      
                          
Outputs                 : 
DeploymentDebugLogLevel : 


PS /home/bgarcial/projects/my-project/Deployments/ARMTemplates/ResourceGroup> 
```

Looks like I would need to include the suscriptionId value inside the resourceId template function that I am using, but currently is not clear for me how to do it despite that I am querying [the template reference][3]

By the way, other detail is that I am using as a `networkPlugin`, kubenet.

Do I need to use the **Azure CNI** as a network plugin in order to get it? 

This is the complete ARM template. I put it here in order to give an idea that what I am doing here

```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "resourceGroupName": {
      "type": "string"  
    },
    "location": {
      "type": "string",
      "defaultValue": "West Europe",
      "metadata": {
        "description": "Geographic Location for all resources."
      }
    },
    "vnetName": {
      "type": "string",
      "defaultValue": "AssessmentVNet",
      "metadata": {
        "description": "Sentia Wordpress Assesment Vnet"
      }
    },
    "vnetAddressPrefix": {
      "type": "string",
      "defaultValue": "10.0.0.0/8",
      "metadata": {
        "description": "Address prefix"
      }
    },
    "subnet1Prefix": {
      "type": "string",
      "defaultValue": "10.240.0.0/16",
      "metadata": {
        "description": "AKS Subnet"
      }
    },
    "subnet1Name": {
      "type": "string",
      "defaultValue": "aks-subnet",
      "metadata": {
        "description": "aks-subnet"
      }
    },
    "subnet2Prefix": {
      "type": "string",
      "defaultValue": "10.241.0.0/27",
      "metadata": {
        "description": "Persistence subnet"
      }
    },
    "subnet2Name": {
      "type": "string",
      "defaultValue": "persistence-subnet",
      "metadata": {
        "description": "persistence-subnet"
      }
    },
    "k8s_cluster_name":{
      "type": "string",
      "defaultValue": "WordpressSentiaAssessment",
      "metadata": {
        "description": "The name of the Azure Kubernetes Service Cluster"
      }
    },
    "kubernetesVersion": {
      "type": "string",
        "defaultValue": "1.14.7",
        "metadata": {
          "description": "The version of the Azure Kubernetes Service Cluster"
        }
    },
    "dnsPrefix": {
      "type": "string",
      "metadata": {
          "description": "Optional DNS prefix to use with hosted Kubernetes API server FQDN."
      }
    },
    "nodeCount": {
      "type": "int",
      "defaultValue": 3,
      "metadata": {
          "description": "The number of nodes that should be created along with the cluster."
      },
      "minValue": 1,
      "maxValue": 100
    },
    "agentVMSize": {
      "type": "string",
      "defaultValue": "Standard_D2_v2",
      "metadata": {
        "description": "The size of the Virtual Machine."
      }
    },
    "servicePrincipalClientId": {
      "metadata": {
        "description": "Client ID (used by cloudprovider)."
      },
      "type": "securestring"
    },
    "servicePrincipalClientSecret": {
      "metadata": {
        "description": "The Service Principal Client Secret."
      },
      "type": "securestring"
    },
    "serviceCidr": {
      "type": "string",
        "metadata": {
            "description": "A CIDR notation IP range from which to assign service cluster IPs."
        },
        "defaultValue": "100.0.0.0/16"
    },
    "dnsServiceIP": {
      "type": "string",
        "metadata": {
          "description": "Containers DNS server IP address."
        },
        "defaultValue": "100.0.0.10"
    },
    "dockerBridgeCidr": {
      "type": "string",
      "metadata": {
          "description": "A CIDR notation IP for Docker bridge."
      },
      "defaultValue": "172.17.0.1/16"
    },
    "environmentName": {
      "type": "string",
      "metadata": {
        "description": "Environment name for tagging purposes, e.g. dev, accp, prod"
      }
    }
  },
  "variables": {
    "osDiskSizeGB": 0,
    "osType": "Linux",
    "maxPods": 110,
    "networkPlugin": "kubenet"
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2018-05-01",
      "location": "[parameters('location')]",
      "name": "[parameters('resourceGroupName')]",
      "properties": {}
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-05-01",
      "name": "storageDeployment",
      "resourceGroup": "[parameters('resourceGroupName')]",
      "dependsOn": [
          "[resourceId('Microsoft.Resources/resourceGroups/', parameters('resourceGroupName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
            {
              "apiVersion": "2018-10-01",
              "type": "Microsoft.Network/virtualNetworks",
              "name": "[parameters('vnetName')]",
              "location": "[parameters('location')]",
              "properties": {
                "subnets": [
                  {
                    "name": "[parameters('subnet1Name')]",
                    "properties": {
                      "addressPrefix": "[parameters('subnet1Prefix')]"
                    }
                  },
                  {
                    "name": "[parameters('subnet2Name')]",
                    "properties": {
                      "addressPrefix": "[parameters('subnet2Prefix')]",
                      "serviceEndpoints": [
                        {
                          "service": "Microsoft.Storage",
                          "locations": "[parameters('location')]"
                        },
                        {
                          "service": "Microsoft.Sql",
                          "locations": "[parameters('location')]"
                        }
                      ]
                    }
                  }
                ],      
                "addressSpace": {
                  "addressPrefixes": [
                    "[parameters('vnetAddressPrefix')]"
                  ]
                }
              },
              "tags": {
                "Environment": "[parameters('environmentName')]"
              }
            },
            {
              "apiVersion": "2019-06-01",
              "dependsOn": [
                "Microsoft.Network/virtualNetworks/AssessmentVNet"
              ],
              "type": "Microsoft.ContainerService/managedClusters",
              "location":"[parameters('location')]",
              "name": "[concat(parameters('k8s_cluster_name'), '-aks')]",
              "properties":{
                "kubernetesVersion":"[parameters('kubernetesVersion')]",
                "enableRBAC": true,
                "dnsPrefix": "[concat(parameters('k8s_cluster_name'),'-dns')]",
                "agentPoolProfiles":[
                  {
                    "name":"defaultpool",
                    "osDiskSizeGB": "[variables('osDiskSizeGB')]",
                    "count":"[parameters('nodeCount')]",
                    "vmSize": "[parameters('agentVMSize')]",
                    "osType": "[variables('osType')]",
                    "storageProfile": "ManagedDisks",
                    "type": "VirtualMachineScaleSets",
                    "vnetSubnetID": "[resourceId(parameters('resourceGroupName'),'Microsoft.Network/virtualNetworks/subnets',parameters('vnetName'),parameters('subnet1Name'))]",
                    "maxPods": "[variables('maxPods')]"
                  }
                ],
                "servicePrincipalProfile": {
                    "ClientId": "[parameters('servicePrincipalClientId')]",
                    "Secret": "[parameters('servicePrincipalClientSecret')]"
                },
                "networkProfile": {
                  "networkPlugin": "[variables('networkPlugin')]",
                  "serviceCidr": "[parameters('serviceCidr')]",
                  "dnsServiceIP": "[parameters('dnsServiceIP')]",
                  "dockerBridgeCidr": "[parameters('dockerBridgeCidr')]"
                }
              },
              "tags": {
                "Environment": "[parameters('environmentName')]"
              }
            },
            {

            }  
          ]
        }
      }
    }    
  ]
}

```


  [1]: https://docs.microsoft.com/bs-latn-ba/azure/azure-resource-manager/deploy-to-subscription
  [2]: https://github.com/Azure/azure-quickstart-templates/blob/master/101-aks-advanced-networking/azuredeploy.json#L157
  [3]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-functions-resource#resourceid