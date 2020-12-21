# Tanzu Service Manager - Some example charts and offers

This repository holds some examples of using the Tanzu Service Manager to link a helm chart up and enable the lifecycle management of a service from a Cloud Foundry/Tanzu Application Service (TAS) environent.



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
