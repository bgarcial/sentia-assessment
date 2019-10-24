# ARM Templates Deployments

I am using Powershell to deploy the templates:

`Deployments/ARMTemplates/Vnet/SentiaWordpressDeployment.json` file deploy a Resource group 
and Vnet
```
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Vnet> New-AzDeployment `
>> -Name SentiaAssessment `
>> -location westeurope `
>> -TemplateUri $templateUri `
>> -resourceGroupName $resourceGroupName  
WARNING: Breaking changes in the cmdlet 'New-AzDeployment' :
WARNING:  - A new parameter "ScopeType" will be introduced to the cmdlet and will be mandatory. ScopeType will be an enum with four values: ResourceGroup, Subscription, ManagementGroup, Tenant. Adding this parameter allows us to use one cmdlet for all Azure Resource Manager template deployments but still determine the intended level of scope.


WARNING: Note :The change is expected to take effect from the version :  '3.0'


WARNING: NOTE : Go to https://aka.ms/azps-changewarnings for steps to suppress this breaking change warning, and other information on breaking changes in Azure PowerShell.

DeploymentName          : SentiaAssessment
Location                : westeurope
ProvisioningState       : Succeeded
Timestamp               : 10/23/19 9:54:57 AM
Mode                    : Incremental
TemplateLink            : 
Parameters              : 
                          Name                 Type                       Value     
                          ===================  =========================  ==========
                          resourceGroupName    String                     sentia-assessment
                          location             String                     West Europe
                          vnetName             String                     AssessmentVNet
                          vnetAddressPrefix    String                     10.0.0.0/8
                          subnet1Prefix        String                     10.240.0.0/16
                          subnet1Name          String                     aks-subnet
                          subnet2Prefix        String                     10.241.0.0/27
                          subnet2Name          String                     persistence-subnet
                          environmentName      Object                     {
                            "Environment": "ACCP",
                            "Project": "Sentia Wordpress Assessment"
                          }
                          
Outputs                 : 
DeploymentDebugLogLevel : 


PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Vnet> 

```

- Now I've added the K8s cluster  to the subnetID and I got this error.
- Define the variables that I will use to the deploymnent


```
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/ResourceGroup> ^C
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/ResourceGroup> $servicePrincipalClientId = Read-Host -AsSecureString                    
************************************
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/ResourceGroup> $servicePrincipalClientSecret = Read-Host -AsSecureString                               
********************************
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/ResourceGroup> $templateUri="/home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Vnet/SentiaWordpressDeployment.json"
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/ResourceGroup> $resourceGroupName="sentia-assessment"       PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/ResourceGroup> 
```
- Execute the deployment template

```
New-AzDeployment : 9:33:32 PM - Resource Microsoft.Resources/deployments 'storageDeployment' failed with message '{
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

```

I have some inconvenients to execute the deployment `the provided value 'sentia-assessment' is not valid subscription identifier`

``` 
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/ResourceGroup> New-AzDeployment `                           >>  -Name SentiaAssessment `                                           
>>  -location westeurope `                                                 
>>  -TemplateUri $templateUri `               
>>  -resourceGroupName $resourceGroupName `                       
>>  -environmentName accp `                                    
>>  -dnsPrefix WordpressSentiaAssessment-dns `                    
>>  -servicePrincipalClientId $servicePrincipalClientId `      
>>  -servicePrincipalClientSecret $servicePrincipalClientSecret
WARNING: Breaking changes in the cmdlet 'New-AzDeployment' :
WARNING:  - A new parameter "ScopeType" will be introduced to the cmdlet and will be mandatory. ScopeType will be an enum with four values: ResourceGroup, Subscription, ManagementGroup, Tenant. Adding this parameter allows us to use one cmdlet for all Azure Resource Manager template deployments but still determine the intended level of scope.


WARNING: Note :The change is expected to take effect from the version :  '3.0'


WARNING: NOTE : Go to https://aka.ms/azps-changewarnings for steps to suppress this breaking change warning, and other information on breaking changes in Azure PowerShell.
New-AzDeployment : 10:57:46 PM - Resource Microsoft.Resources/deployments 'storageDeployment' failed with message '{
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
 
New-AzDeployment : 10:57:46 PM - Template output evaluation skipped: at least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.
At line:1 char:1
+ New-AzDeployment `
+ ~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [New-AzDeployment], Exception
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.NewAzureDeploymentCmdlet
 
New-AzDeployment : 10:57:46 PM - Template output evaluation skipped: at least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.
At line:1 char:1
+ New-AzDeployment `
+ ~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [New-AzDeployment], Exception
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.NewAzureDeploymentCmdlet
 

DeploymentName          : SentiaAssessment
Location                : westeurope
ProvisioningState       : Failed
Timestamp               : 10/23/19 8:57:42 PM
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


PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/ResourceGroup> 



```















