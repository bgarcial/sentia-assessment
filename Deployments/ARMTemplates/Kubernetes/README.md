I had problems with create the SP from the ARM template, so in order to continue executing some tests
I decided to use some SP already created. (`dev-SentiaWordpress-BernardoSP-20191020155429`) 


- Creating the variables `$clientId` and `$clientSecret`

```
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Kubernetes> $clientId = Read-Host -AsSecureString
************************************
```

```
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Kubernetes> $clientSecret = Read-Host -AsSecureString                                 
********************************
```


- Validating the ARM Template

```
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Kubernetes> Test-AzResourceGroupDeployment -ResourceGroupName dev-SentiaWordpress-Bernardo -TemplateFile ./SentiaWordpressInfrastructure.json -omsWorkspaceId 3f7e43fe-f98f-4f6b-9cef-339f4a576e1f -workspaceName dev-Bernardo-Sentia -dnsPrefix dev-SentiaWordpress-Bernardo-dns -principalId a3906348-46a7-470b-a1ae-4bb9e9a9c32a -environmentName dev -servicePrincipalClientId $clientId -servicePrincipalClientSecret $clientSecret

Code    : InvalidTemplate
Message : Deployment template validation failed: 'The template resource 
          'dev-SentiaWordpress-Bernardo/Microsoft.Authorization/251d14bb-b971-47bd-98db-cef1bf4cc8f6' at line '259' and column 
          '9' is not valid: The resource identificator 
          'Microsoft.ContainerService/managedClustersdev-SentiaWordpress-Bernardo-aks' is malformed. Please see 
          https://aka.ms/arm-template-expressions/#reference for usage details.. Please see 
          https://aka.ms/arm-template-expressions for usage details.'.
Details : 


PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Kubernetes> 

```
I need to check the 303 line in the template, 
```
"dependsOn": [
    "[concat('Microsoft.ContainerService/managedClusters', resourceGroup().name, '-aks')]"
            ]    
```

---

- Executing the template, creating the resources

```
New-AzResourceGroupDeployment -ResourceGroupName dev-SentiaWordpress-Bernardo -TemplateFile ./SentiaWordpressInfrastructure.json -omsWorkspaceId 3f7e43fe-f98f-4f6b-9cef-339f4a576e1f -workspaceName dev-Bernardo-Sentia -dnsPrefix dev-SentiaWordpress-Bernardo-dns -principalId a3906348-46a7-470b-a1ae-4bb9e9a9c32a -environmentName dev -servicePrincipalClientId $clientId -servicePrincipalClientSecret $clientSecret 
New-AzResourceGroupDeployment : 6:13:46 PM - Resource Microsoft.Network/virtualNetworks/subnets/providers/roleAssignments 'dev-SentiaWordpress-Bernardo-vnet/default/Microsoft.Authorization/3ca31ef5-2436-481e-9102-bc09c02dae29' failed with message '{
  "error": {
    "code": "PrincipalNotFound",
    "message": "Principal a390634846a7470ba1ae4bb9e9a9c32a does not exist in the directory 4e6b0716-50ea-4664-90a8-998f60996c44."
  }
}'
At line:1 char:1
+ New-AzResourceGroupDeployment -ResourceGroupName dev-SentiaWordpress- ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [New-AzResourceGroupDeployment], Exception
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.NewAzureResourceGroupDeploymentCmdlet
 
New-AzResourceGroupDeployment : 6:13:46 PM - Resource Microsoft.ContainerService/managedClusters/providers/roleAssignments 'dev-SentiaWordpress-Bernardo/Microsoft.Authorization/251d14bb-b971-47bd-98db-cef1bf4cc8f6' failed with message '{
  "error": {
    "code": "ResourceNotFound",
    "message": "The Resource 'Microsoft.ContainerService/managedClusters/dev-SentiaWordpress-Bernardo' under resource group 'dev-SentiaWordpress-Bernardo' was not found."
  }
}'
At line:1 char:1
+ New-AzResourceGroupDeployment -ResourceGroupName dev-SentiaWordpress- ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [New-AzResourceGroupDeployment], Exception
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.NewAzureResourceGroupDeploymentCmdlet
 
New-AzResourceGroupDeployment : 6:13:51 PM - Resource Microsoft.OperationsManagement/solutions 'ContainerInsights' failed with message '{
  "error": {
    "code": "BadRequest",
    "message": "Bad request. Diagnostic information: timestamp '20191020T161349Z', subscription id '9148bd11-f32b-4b5d-a6c0-5ac5317f29ca', tracking id 'f9eaf2c3-2e0e-4f31-8382-24327884a46e', request correlation id '9126fbf3-b7e7-459c-9c90-671449f254dd'."
  }
}'
At line:1 char:1
+ New-AzResourceGroupDeployment -ResourceGroupName dev-SentiaWordpress- ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [New-AzResourceGroupDeployment], Exception
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.NewAzureResourceGroupDeploymentCmdlet
```


Workspace Analytics


- Testing the Vnet

```
Get-AzResource -ResourceType Microsoft.Network/virtualNetworks -ResourceGroupName dev-SentiaWordpress-Bernardo        

Name              : dev-SentiaWordpress-Bernardo-vnet
ResourceGroupName : dev-SentiaWordpress-Bernardo
ResourceType      : Microsoft.Network/virtualNetworks
Location          : westeurope
ResourceId        : /subscriptions/xxxxxxxxxxx/resourceGroups/dev-SentiaWordpress-Bernardo/providers/Microsoft.Network/virtualNetworks/dev-SentiaWordpress-Bernardo-vnet


PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Kubernetes>



- Testing the AKS cluster existence
```
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Kubernetes> Get-AzResource -ResourceType Microsoft.ContainerService/managedClusters -ResourceGroupName dev-SentiaWordpress-Bernardo

Name              : dev-SentiaWordpress-Bernardo-aks
ResourceGroupName : dev-SentiaWordpress-Bernardo
ResourceType      : Microsoft.ContainerService/managedClusters
Location          : westeurope
ResourceId        : /subscriptions/9148bd11-f32b-4b5d-a6c0-5ac5317f29ca/resourceGroups/dev-SentiaWordpress-Bernardo/providers/Microsoft.Con
                    tainerService/managedClusters/dev-SentiaWordpress-Bernardo-aks


PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Kubernetes> 
```

Summary
With this ARM Template I 