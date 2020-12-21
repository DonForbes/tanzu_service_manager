# PostgreSQL Operator

Postgres information - [PostgreSQL](https://www.postgresql.org/)
Tanzu Postgres information - [Tanzu Postgres](https://postgres-kubernetes.docs.pivotal.io/about.html)

Tanzu postgres is deployed by using a kubernetes operator, once deployed a fairly simple yaml deployment can use the operator to create an instance of the postgres database.  TSMGR manages instances via a helm chart so the first step is to create a helm chart which can deploy the instance then create the offer that will result in the service and its plans being available to Cloud Foundry.

This example shows how to use the Tanzu Service Manager to manage the instances for Tanzu Postgres Operator for Kubernetes.


## Installing the Tanzu Postgres Operator for Kubernetes

1) Please follow the steps to install the Tanzu Postgres Operator for Kubernetes to your cluster from
   [here](http://postgres-kubernetes.docs.pivotal.io/1-0/create-release.html). **NOTE**: the dockerRegistrySecretName
   in `values.yaml` file must be set to the **`registry-secret`** value instead of `regsecret`, matching the TSMGR requirement.

1) Validate your operator is working by running:
    <pre>
    <b>$ kubectl get all</b>
    NAME                                     READY   STATUS    RESTARTS   AGE
    pod/postgres-operator-7456f6b775-m5xnv   1/1     Running   0          24h

    NAME                                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes                          ClusterIP   10.80.0.1     <none>        443/TCP   28h
    service/postgres-operator-webhook-service   ClusterIP   10.80.2.116   <none>        443/TCP   24h

    NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/postgres-operator   1/1     1            1           24h

    NAME                                           DESIRED   CURRENT   READY   AGE
    replicaset.apps/postgres-operator-7456f6b775   1         1         1       24h
    </pre>

Note - the postgres operator deployes into the default namespace.

## Create the helm chart 
1) Once the operator is working make the postgres instance helm chart.  This can be created from the content of the tanzu-postgres directory in this repository.
```
$ cd <path to hem-charts>
$ helm package tanzu-postgres
```
This will create a file in your working directory called tanzu-postgres-<Version>.tgz.
   
This package can be deployed to your kubernetes cluster where the operator is running to test the creation of a postgres instance.

```
$ kubectl config set-context --current --namespace=default
Context "services-admin@services" modified.
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
postgres-operator-b787bfd69-2mkm2   1/1     Running   0          3d1h

$ helm install tanzu-postgres tanzu-postgres-0.1.0.tgz
NAME: tanzu-postgres
LAST DEPLOYED: Mon Dec 21 13:59:06 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/postgres-operator-b787bfd69-2mkm2   1/1     Running   0          3d2h
pod/tanzu-postgres-0                    1/1     Running   0          2m17s
pod/tanzu-postgres-monitor-0            1/1     Running   0          2m17s

NAME                                        TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)          AGE
service/kubernetes                          ClusterIP      100.64.0.1       <none>                                                                   443/TCP          12d
service/postgres-operator-webhook-service   ClusterIP      100.69.213.31    <none>                                                                   443/TCP          3d2h
service/tanzu-postgres                      LoadBalancer   100.67.243.156   ad92b2df2cb4f4bed971a666d53a13c7-543759362.eu-west-2.elb.amazonaws.com   5432:32423/TCP   2m17s
service/tanzu-postgres-agent                ClusterIP      None             <none>                                                                   <none>           2m17s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgres-operator   1/1     1            1           3d2h

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/postgres-operator-b787bfd69   1         1         1       3d2h

NAME                                      READY   AGE
statefulset.apps/tanzu-postgres           1/1     2m17s
statefulset.apps/tanzu-postgres-monitor   1/1     2m17s

NAME                                           STATUS    AGE
postgres.sql.tanzu.vmware.com/tanzu-postgres   Running   2m17s

```

This shows that the pods for the postgres instance have started successfully (tanzu-postgres-monitor-0 and tanzu-postgres-0) and the service has been exposed as a load balancer as defined in the values.yaml.

Note - it is likely that to test this helm chart out it will be necessary to adjust the storageClass in the values.yaml file to ensure that it maps to a storageClass appropriate for your Kubernetes cluster.
