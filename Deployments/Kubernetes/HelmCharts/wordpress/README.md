
- CReating IP address
```
⟩ az network public-ip create --resource-group MC_sentia-assessment-testing_WordpressSentiaAssessment-aks_westeurope --name WordpressTestingStaticIpAddress --allocation-method static
{
  "publicIp": {
    "ddosSettings": null,
    "dnsSettings": null,
    "etag": "W/\"7a6e9ff7-2485-4101-b4db-b5c81bf432d8\"",
    "id": "/subscriptions/xxxxxxxx/resourceGroups/MC_sentia-assessment-testing_WordpressSentiaAssessment-aks_westeurope/providers/Microsoft.Network/publicIPAddresses/WordpressTestingStaticIpAddress",
    "idleTimeoutInMinutes": 4,
    "ipAddress": "xxxxxx",
    "ipConfiguration": null,
    "ipTags": [],
    "location": "westeurope",
    "name": "WordpressTestingStaticIpAddress",
    "provisioningState": "Succeeded",
    "publicIpAddressVersion": "IPv4",
    "publicIpAllocationMethod": "Static",
    "publicIpPrefix": null,
    "resourceGroup": "MC_sentia-assessment-testing_WordpressSentiaAssessment-aks_westeurope",
    "resourceGuid": "0a1da889-5bbe-43f7-85ec-6db850b2a66e",
    "sku": {
      "name": "Basic",
      "tier": "Regional"
    },
    "tags": null,
    "type": "Microsoft.Network/publicIPAddresses",
    "zones": null
  }
}
```

- Creating NGINX ingress controller

Namespace
```
⟩ kubectl create namespace nginx
namespace/nginx created
[I] 

```
helm deployment

```
⟩ helm3 install nginx-ingress stable/nginx-ingress --namespace nginx --set controller.service.loadBalancerIP=40.113.125.247 
NAME: nginx-ingress
LAST DEPLOYED: Tue Oct 29 18:39:00 2019
NAMESPACE: nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace nginx get services -o wide -w nginx-ingress-controller'

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

- Relate ip with domain

- Cert manager 


```
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml
```
```
kubectl create namespace cert-manager
```
```
⟩ helm3 repo add jetstack https://charts.jetstack.io
  
"jetstack" has been added to your repositories
```
```
 helm3 repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jetstack" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
```
```
⟩ helm3 install cert-manager --namespace cert-manager --version v0.11.0 jetstack/cert-manager
NAME: cert-manager
LAST DEPLOYED: Tue Oct 29 20:37:14 2019
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
cert-manager has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://docs.cert-manager.io/en/latest/reference/issuers.html

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://docs.cert-manager.io/en/latest/reference/ingress-shim.html

```

Mysql DB port 3306 is exposed to the Internet. We do not recommend exposing database ports to the Internet and suggest only exposing them to your front-end tier inside your virtual network.

nano wp-config-sample.php
cp wp-config-sample.php wp-config.php




- Wordpress helm command installation


```
helm3 install wordpress-site-1 stable/wordpress
```
By default install maria db pod
and a Load Balancer public IP address ON AZURE
That public ip address allow to reachwordpress from
outside
http://52.157.209.126/
http://52.157.209.126/wp-admin

If I delete the deployment, the public IP persist.
We can use it or we can create it from ARM template
I prefer to create it from ARM template and specify it 
in the helm command installation, so in the ingress as well
we can work with it.

```
⟩ helm3 install wordpress-site-1 stable/wordpress --set image.registry=wordpresssentiaassessment.azurecr.io,image.repository=wordpress,image.tag=5.2.4

```

- dOKCER REG secret
```
kubectl create secret docker-registry regcred --docker-server=myregistryname.azurecr.io --docker-username=myusername --docker-password=my-password --docker-email=b.garcia.l@outlook.com

```


- External db

```
helm3 install wordpress-site-1 ./Deployments/HelmCharts/wordpress/  --set image.registry=$acr_login_server,image.repository=$acr_repository,image.tag=5.2.4,image.pullPolicy=Always,wordpressUsername=$wordpressUsername,wordpressPassword=$wordpressPassword,wordpressEmail=$wordpressEmail,mariadb.enabled=false,externalDatabase.host=$database_host,externalDatabase.user=database_user,externalDatabase.password=database_password,externalDatabase.database=$database_name,externalDatabase.port=3306

```

helm upgrade wordpress-site-1 ./zcrm365 --install

```
helm3 upgrade wordpress-site-1 ./Deployments/HelmCharts/wordpress/  --set image.registry=$acr_login_server,image.repository=$acr_repository,image.tag=5.2.4,image.pullPolicy=Always,wordpressUsername=$wordpressUsername,wordpressPassword=$wordpressPassword,wordpressEmail=$wordpressEmail,mariadb.enabled=false,externalDatabase.host=$database_host,externalDatabase.user=database_user,externalDatabase.password=database_password,externalDatabase.database=$database_name,externalDatabase.port=3306

```

- Referencing secrets name and values in detail via helm



helm3 install wordpress-site-1 stable/wordpress --set image.registry=my-registryname.azurecr.io,image.repository=wordpress,image.tag=5.2.4,image.pullPolicy=Always,wordpressUsername=site1,wordpressPassword=mypostgrespassword,wordpressEmail=botibagl@gmail.com,mariadb.enabled=false,externalDatabase.host.valueFrom.secretKeyRef.key=host,externalDatabase.host.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,externalDatabase.user.valueFrom.secretKeyRef.key=username,externalDatabase.user.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,externalDatabase.password.valueFrom.secretKeyRef.key=password,externalDatabase.password.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,externalDatabase.database.valueFrom.secretKeyRef.key=database,externalDatabase.database.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,externalDatabase.port=3306

- Connecting to MySQL

```
⟩ mysql -u sentia@wordpresssentiaassessment-db -p -h 191.237.232.75
```