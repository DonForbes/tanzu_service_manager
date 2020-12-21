# Tanzu Service Manager - Some example charts and offers

## Introduction

This repository holds some examples of using the Tanzu Service Manager to link a helm chart up and enable the lifecycle management of a service from a Cloud Foundry/Tanzu Application Service (TAS) environent. The architecture of the Tanzu Service Manager is detailed in the [documentation](https://docs.pivotal.io/tanzu-service-manager/index.html).


## Prerequisites

- TAS environment
- K8s environment with Tanzu Service Manager installed and configured.

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

## Deploying service Helm Chart and creating the "offer"
TSMGR will deploy a service using a helm chart, it has the concept of an offer which wraps up the helm chart and makes it available to cloud foundry.  The offer allows the author to override the configuration of the helm chart and create separate plans which can further override the default chart deployment allowing for different "T-Shirt" instance configurations to be available.  eg.  A small database for development/functional testing and larger plans for scale testing and production.

### Steps to create and deploy an offer
1. Download or create the helm chart to deploy the service offering.
2. Create the offer directory structure.  The initial structure can be created via the tsmgr command line.
```
$ tsmgr offer init service-offer
$ tree
.
└── service-offer
    ├── bind.yaml
    ├── overrides
    │   └── override-values.yaml
    ├── plans
    │   ├── medium.yaml
    │   └── small.yaml
    ├── plans.yaml
    └── tsmgr.yaml
```
Where:-
- tsmgr.yaml holds the details of what the service will be called when registered with cloud foundry
- override-values.yaml holds override values that must match to the field/values in the helm chart.  A default selection of overrides for the environment that the service is to be deployed into.  (eg. Setup the storageClass appropriate to the k8s environment)
- plans.yaml - specifies the plan names and descriptions that will appear in Cloud Foundry with the ``cf marketplace`` command. This file specifies the "plan.yaml" files which are further overrides above the override-values.yaml to specify values used for this particular plan size.
- bind.yaml is the file used to provide environment variables to a running application, the values can be selected from the kubernetes object configurations and formatted according to the needs of the application.

3. Once the offer has been populated appropriately we can use tsmgr to deploy the offer.
```
$ tsmgr offer save
Error: missing required arguments
Usage:
  tsmgr offer save [PATH-TO-TANZU-SERVICE-MANAGER-DIR/] [PATH-TO-CHART.tgz] [...] [flags]
```
i.e. You create the offer in TSMGR using the specific configuration created and the helm chart for the service to be deployed.  This will register the service with Cloud Foundry, which by default will not be made available to any of the organisations.

```
$ tsmgr offer list
OFFER           	INCLUDED CHARTS                	PLANS
bitnami-postgres	[*postgresql:10.1.4]           	[small medium]
tanzu-postgres  	[*postgres-oper-instance:0.1.0]	[default huge]
```

## Enabling CF access

The offer access is not available by default in cf. You can verify that by calling the follow commands.
Notice that postgres is not available in the marketplace, even though it is listed by service-access (with access=none):

In order to enable the access, use the following command:
```bash
$ cf enable-service-access tanzu-postgres
Enabling access to all plans of service tanzu-postgres for all orgs as admin...
OK

$ cf marketplace
Getting services from marketplace in org system / space dev as admin...
OK

service                    plans           description                                                                         broker
tanzu-postgres.            default, huge   A Helm chart for creating the Tanzu Postgress Operator for Kubernetes instance.     tanzu-service-manager
```

## Creating an instance

After enabling access to the offer in cf, it's possible to provision a new instance.

```bash
$ cf create-service tanzu-postgres default pg-instance
Creating service instance pg-instance in org system / space dev as admin...
OK

Create in progress. Use 'cf services' or 'cf service pg-instance' to check operation status.

$ cf services
Getting services in org system / space dev as admin...

name             service                    plan      bound apps   last operation     broker                  upgrade available
pg-instance      tanzu-postgres.            default                create succeeded   tanzu-service-manager   no
```

## External References

For more details and customizations for Postgres chart, see https://github.com/helm/charts/tree/master/stable/postgres

For more details on tsmgr usage see https://docs.pivotal.io/tanzu-service-manager/using.html

For other Tanzu documents see https://docs.pivotal.io/
