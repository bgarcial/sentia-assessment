
- Create the resource group
```
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Infrastructure> New-AzResourceGroup `                       >> -Name $resourceGroupName `                                                  
>> -Location westeurope 

ResourceGroupName : testing
Location          : westeurope
ProvisioningState : Succeeded
Tags              : 
ResourceId        : /subscriptions/9148bd11-f32b-4b5d-a6c0-5ac5317f29ca/resourceGroups/testing


PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Infrastructure>
```

- Define environment variables to use in the `New-AzResourceGroupDeployment` command.
  
```
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Infrastructure> $resourceGroupName="testing"                
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Infrastructure> $templateUriTesting="/home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json"
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Infrastructure> 

```


- Command to execute the deployment
```
PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Infrastructure> New-AzResourceGroupDeployment `
>> -ResourceGroupName $resourceGroupName `       
>> -TemplateFile $templateUriTesting `
>> -environmentName accp `  

cmdlet New-AzResourceGroupDeployment at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
resourceGroupNameFromTemplate: testing
dnsPrefix: WordpressAssessment-dns
servicePrincipalClientId: ************************************
servicePrincipalClientSecret: ********************************

DeploymentName          : testing
ResourceGroupName       : testing
ProvisioningState       : Succeeded
Timestamp               : 10/24/19 1:18:36 PM
Mode                    : Incremental
TemplateLink            : 
Parameters              : 
                          Name                            Type                       Value     
                          ==============================  =========================  ==========
                          resourceGroupName               String                     testing   
                          location                        String                     West Europe
                          vnetName                        String                     AssessmentVNetTesting
                          vnetAddressPrefix               String                     10.0.0.0/8
                          subnet1Prefix                   String                     10.240.0.0/16
                          subnet1Name                     String                     aks-subnet
                          subnet2Prefix                   String                     10.241.0.0/27
                          subnet2Name                     String                     persistence-subnet-testing
                          k8s_cluster_name                String                     WordpressSentiaAssessment
                          kubernetesVersion               String                     1.14.7    
                          dnsPrefix                       String                     WordpressAssessment-dns
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


PS /home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Infrastructure>
```

## New Deployment adding MySQL

- Defining variables

Add 

resourceGroupNameFromTemplate
administratorLogin

```
PS /home/bgarcial> $templateUriTesting="/home/bgarcial/projects/sentia-assesment/Deployments/ARMTemplates/Infrastructure/AzResourceGroupDeploymentApproach/testing.json"   
PS /home/bgarcial> $environmentName="accp"                                                                                       PS /home/bgarcial> $resourceGroupName="testing"                                                                                  PS /home/bgarcial> $servicePrincipalClientId = Read-Host -AsSecureString                                                         ************************************                              
PS /home/bgarcial> $servicePrincipalClientSecret = Read-Host -AsSecureString
********************************                                  
PS /home/bgarcial>   
```

- Executing Deployment

```
PS /home/bgarcial> New-AzResourceGroupDeployment `
>> -ResourceGroupName $resourceGroupName `
>> -TemplateFile $templateUriTesting `
>> -environmentName $environmentName `
>> -servicePrincipalClientId $servicePrincipalClientId `
>> -servicePrincipalClientSecret $servicePrincipalClientSecret

cmdlet New-AzResourceGroupDeployment at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
resourceGroupNameFromTemplate: testing
administratorLogin: bgarcial
administratorLoginPassword: *********

DeploymentName          : testing
ResourceGroupName       : testing
ProvisioningState       : Succeeded
Timestamp               : 10/25/19 10:13:14 AM
Mode                    : Incremental
TemplateLink            : 
Parameters              : 
                          Name                            Type                       Value     
                          ==============================  =========================  ==========
                          resourceGroupName               String                     testing   
                          location                        String                     West Europe
                          vnetName                        String                     AssessmentVNetTesting
                          vnetAddressPrefix               String                     10.0.0.0/8
                          subnet1Prefix                   String                     10.240.0.0/16
                          subnet1Name                     String                     aks-subnet
                          subnet2Prefix                   String                     10.241.0.0/27
                          subnet2Name                     String                     persistence-subnet-testing
                          k8s_cluster_name                String                     WordpressSentiaAssessment
                          kubernetesVersion               String                     1.14.7    
                          nodeCount                       Int                        3         
                          agentVMSize                     String                     Standard_D2_v2
                          servicePrincipalClientId        SecureString                         
                          servicePrincipalClientSecret    SecureString                         
                          serviceCidr                     String                     100.0.0.0/16
                          dnsServiceIP                    String                     100.0.0.10
                          dockerBridgeCidr                String                     172.17.0.1/16
                          serverName                      String                     WordpressSentiaAssessment
                          administratorLogin              String                     bgarcial  
                          administratorLoginPassword      SecureString                         
                          skuCapacity                     Int                        2         
                          skuName                         String                     GP_Gen5_2 
                          skuSizeMB                       Int                        5120      
                          skuTier                         String                     GeneralPurpose
                          skuFamily                       String                     Gen5      
                          mysqlVersion                    String                     5.7       
                          storageAutoGrow                 String                     Disabled  
                          backupRetentionDays             Int                        7         
                          geoRedundantBackup              String                     Disabled  
                          virtualNetworkRuleName          String                     AllowSubnet
                          environmentName                 String                     accp      
                          
Outputs                 : 
DeploymentDebugLogLevel : 


PS /home/bgarcial>
```
