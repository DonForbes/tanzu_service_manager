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
```

