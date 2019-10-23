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