# PostgreSQL Operator

[PostgreSQL](https://www.postgresql.org/)

This example shows how to use the Tanzu Service Manager to manage the instances for Tanzu Postgres Operator for Kubernetes.

## Introduction

This example includes a Helm chart for the Tanzu Postgres Operator Instance.

## Prerequisites

- TAS environment with Tanzu Service Manager installed and configured.
- Connection setup for the `tsmgr` CLI to properly target an environment

```
export TSMGR_TARGET=http://<change_by_your_tsmgr_server>:<change_by_your_tsmgr_server_port>
export TSMGR_TOKEN=<change_by_your_tsmgr_kubernetes_token>
export TSMGR_INSECURE=true # if using a lab environment
```

- Kubernetes [cluster registered with TSMGR](https://docs.pivotal.io/tanzu-service-manager/managing-clusters.html) and set as default
```bash
tsmgr cluster register my-cluster-name my-cluster-creds-file.yaml
tsmgr cluster set-default my-cluster-name
```

## Installing the Tanzu Postgres Operator for Kubernetes

1) Please follow the steps to install the Tanzu Postgres Operator for Kubernetes to your cluster from
   [here](http://postgres-kubernetes.docs.pivotal.io/1-0/create-release.html). **NOTE**: the dockerRegistrySecretName
   in `values.yaml` file must be set to the `registry-secret` value instead of `regsecret`, matching the TSMGR requirement.

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

## Saving the Offer

1) Once the operator is working, download the [postgres-oper-instance-0.1.0.tgz](./postgres-oper-instance-0.1.0.tgz)
   helm chart and [postgres-oper-inst-offer.tgz](./postgres-oper-inst-offer.tgz) TSMGR offer files to a new empty folder.

1) Untar the `postgres-oper-inst-offer.tgz` file using the command:
   ```bash
   tar xvzf postgres-oper-inst-offer.tgz
   ```

1) The offer will provide the TSMGR files as plans, overrides and bind template. Please verify those files and customize
   them if you need to set any specifics.

1) From the root level of the new folder where you saved both files, save the offer:

    ```bash
    $ tsmgr offer save postgres-oper-inst-offer/ postgres-oper-instance-0.1.0.tgz
    ```

## Enabling CF access

The offer access is not available by default in cf. You can verify that by calling the follow commands.
Notice that postgres is not available in the marketplace, even though it is listed by service-access (with access=none):

In order to enable the access, use the following command:
```bash
$ cf enable-service-access postgres-oper-inst-offer
Enabling access to all plans of service postgres-oper-inst-offer for all orgs as admin...
OK

$ cf marketplace
Getting services from marketplace in org system / space dev as admin...
OK

service                    plans           description                                                                         broker
postgres-oper-inst-offer   default, huge   A Helm chart for creating the Tanzu Postgress Operator for Kubernetes instance.     tanzu-service-manager
```

## Creating an instance

After enabling access to the offer in cf, it's possible to provision a new instance.

```bash
$ cf create-service postgres-oper-inst-offer default pg-instance
Creating service instance pg-instance in org system / space dev as admin...
OK

Create in progress. Use 'cf services' or 'cf service pg-instance' to check operation status.

$ cf services
Getting services in org system / space dev as admin...

name             service                    plan      bound apps   last operation     broker                  upgrade available
pg-instance      postgres-oper-inst-offer   default                create succeeded   tanzu-service-manager   no
```

## External References

For more details and customizations for Postgres chart, see https://github.com/helm/charts/tree/master/stable/postgres

For more details on tsmgr usage see https://docs.pivotal.io/tanzu-service-manager/using.html

For other Tanzu documents see https://docs.pivotal.io/
