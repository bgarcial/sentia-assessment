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
kubectl create secret docker-registry regcred --docker-server=wordpresssentiaassessment.azurecr.io --docker-username=WordpressSentiaAssessment --docker-password=Umrd906H8LJ3Gjfuqi2=zeqL0gJYjyX0 --docker-email=b.garcia.l@outlook.com

```


- External db

```
helm3 install wordpress-site-1 ./Deployments/HelmCharts/wordpress/  --set image.registry=$acr_login_server,image.repository=$acr_repository,image.tag=5.2.4,image.pullPolicy=Always,wordpressUsername=$wordpressUsername,wordpressPassword=$wordpressPassword,wordpressEmail=$wordpressEmail,mariadb.enabled=false,externalDatabase.host=$database_host,externalDatabase.user=database_user,externalDatabase.password=database_password,externalDatabase.database=$database_name,externalDatabase.port=3306

```

- I am getting this in the k describe pod



- It works
```
helm3 install wordpress-site-1 stable/wordpress --set image.registry=wordpresssentiaassessment.azurecr.io,image.repository=wordpress,image.tag=5.2.4,image.pullPolicy=Always,wordpressUsername=site1,wordpressPassword=5H3%%y5lh5Pn,wordpressEmail=botibagl@gmail.com,mariadb.enabled=false,externalDatabase.host=wordpresssentiaassessment-db.mysql.database.azure.com,externalDatabase.user=sentia@wordpresssentiaassessment-db,externalDatabase.password=EsVMna5OSUt4Ziw2Dl,externalDatabase.database=wordpress-site-1,externalDatabase.port=3306,ingress.enabled=true
NAME: wordpress-site-1
LAST DEPLOYED: Mon Oct 28 11:33:24 2019
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the WordPress URL:

  You should be able to access your new WordPress installation through

2. Login with the following credentials to see your blog

  echo Username: site1
  echo Password: $(kubectl get secret --namespace default wordpress-site-1 -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```



externalDatabase.host.valueFrom.secretKeyRef.key=host,
externalDatabase.host.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,

externalDatabase.user.valueFrom.secretKeyRef.key=username,externalDatabase.user.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,

externalDatabase.password.valueFrom.secretKeyRef.key=password,externalDatabase.password.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,

externalDatabase.database.valueFrom.secretKeyRef.key=database,externalDatabase.database.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,externalDatabase.port=3306



elm3 install wordpress-site-1 stable/wordpress --set image.registry=my-registryname.azurecr.io,image.repository=wordpress,image.tag=5.2.4,image.pullPolicy=Always,wordpressUsername=site1,wordpressPassword=mypostgrespassword,wordpressEmail=botibagl@gmail.com,mariadb.enabled=false,externalDatabase.host.valueFrom.secretKeyRef.key=host,externalDatabase.host.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,externalDatabase.user.valueFrom.secretKeyRef.key=username,externalDatabase.user.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,externalDatabase.password.valueFrom.secretKeyRef.key=password,externalDatabase.password.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,externalDatabase.database.valueFrom.secretKeyRef.key=database,externalDatabase.database.valueFrom.secretKeyRef.name=mysql-wordpress-secrets,externalDatabase.port=3306