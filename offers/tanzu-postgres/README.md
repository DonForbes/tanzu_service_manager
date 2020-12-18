
# VMWare Tanzu Postgres

A VMWare supported deployment of Postgres, running on Kubernetes.  This can be deployed via Tanzu Service Manager onto a Kubernetes instance and can then be called from the Cloud Foundry Command line such that a service can be created and then bound to applications running in Cloud Foundry.

## Tanzu Postgres Architecture
The Tanzu postgres for Kubernetes architecture is described in the Tanzu [documentation](https://postgres-kubernetes.docs.pivotal.io/about.html) but in summary the deployment is done in two steps.  Firstly is the deployment of the Postgres operator and then secondly is the usage of the operator to create an instance of Postgres.  The postgres documentation does the latter step via a simple YAML configuration that will interact with the operator CRD to create the instance.  

Tanzu Service Manager does manage yaml files directly itself but manages helm charts.  This repository includes a section of helm charts which has been created to deploy postgres via helm.  It is this chart that Tanzu Service Manager will use to create instances of Postgres.


