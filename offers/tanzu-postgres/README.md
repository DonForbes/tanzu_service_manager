
# VMWare Tanzu Postgres

A VMWare supported deployment of Postgres, running on Kubernetes.  This can be deployed via Tanzu Service Manager onto a Kubernetes instance and can then be called from the Cloud Foundry Command line such that a service can be created and then bound to applications running in Cloud Foundry.

## Tanzu Postgres Architecture
The Tanzu postgres for Kubernetes architecture is described in the Tanzu [documentation](https://postgres-kubernetes.docs.pivotal.io/about.html) but in summary the deployment is done in two steps.  Firstly is the deployment of the Postgres operator and then secondly is the usage of the operator to create an instance of Postgres.  The postgres documentation does the latter step via a simple YAML configuration that will interact with the operator CRD to create the instance.  

Tanzu Service Manager does not manage yaml files directly itself but manages helm charts.  This repository includes a directory containing the files necessary to create a simple helm chart that can create an instance of Postgres, then under the offers structure is a configuration for an offer which will can be deployed with TSMGR to expose a couple of plans to applications running in Cloud Foundry/Tanzu Application Service.  The offer includes the definition for binding required for a spring boot application ([Spring Music](https://github.com/cloudfoundry-samples/spring-music)) to connect to the postgres database.

## Deployment of the offer

1. Depoy the Postgres operator following the instructions in the [documentation](https://postgres-kubernetes.docs.pivotal.io/installing.html) and take note of the naming convention required for the **[registry secret](https://github.com/DonForbes/tanzu_service_manager/tree/main/helm-charts/tanzu-postgres)**.  
2. Following the instructions create a helm chart for the deployment of a Postgres Database instance.
3. Either using the offer for tanzu-postgres in this repo or create your own using ``tsmgr offer init`` and edit the fields according to your need.
4. Deploy the offer along with the helm chart created in step 2 to Tanzu Service Manager.  (First you must have configured TSMGR for a cloud foundry foundation)
5. Enable the service in cloud foundry
6. Create the service and bind it to an application
7. Test all is as we would hope
8. Unbind the service and delete

<pre>
$ cd <Base Directory of the tanzu_service_manager>
$ tsmgr offer save offers/tanzu-postgres ./helm-charts/tanzu-postgres-0.1.0.tgz
Offer saved [tanzu-postgres]
$ tsmgr offer list
OFFER           	INCLUDED CHARTS        	PLANS
tanzu-postgres  	[*tanzu-postgres:0.1.0]	[small large]
bitnami-postgres	[*postgresql:10.1.4]   	[small medium]

$ cf login -u admin -p <PASSWORD>
API endpoint: https://api.sys.tas.donaldforbes.com

Authenticating...
OK

Select an org:
1. donald
2. system

Org (enter to skip): 1
Targeted org donald.

Targeted space dev.

API endpoint:   https://api.sys.tas.donaldforbes.com
API version:    3.90.0
user:           admin
org:            donald
space:          dev

$ cf service-access
Getting service access as admin...

broker: tanzusmgr
   offering           plan     access   orgs
   bitnami-postgres   medium   all
   bitnami-postgres   small    all
   tanzu-postgres     large    none
   tanzu-postgres     small    none$ cf enable-service-access tanzu-postgres
Enabling access to all plans of service offering tanzu-postgres for all orgs as admin...
OK

$ cf service-access
Getting service access as admin...

broker: tanzusmgr
   offering           plan     access   orgs
   bitnami-postgres   medium   all
   bitnami-postgres   small    all
   tanzu-postgres     large    all
   tanzu-postgres     small    all
   
$ cf login -u donald -p donald
API endpoint: https://api.sys.tas.donaldforbes.com


Authenticating...
OK

Targeted org donald.

Targeted space dev.

API endpoint:   https://api.sys.tas.donaldforbes.com
API version:    3.90.0
user:           donald
org:            donald
space:          dev

$ cf marketplace
Getting all service offerings from marketplace in org donald / space dev as donald...

offering           plans           description                                                                                                                                     broker
bitnami-postgres   small, medium   Chart for PostgreSQL, an object-relational database management system (ORDBMS) with an emphasis on extensibility and on standards-compliance.   tanzusmgr
tanzu-postgres     small, large    A Helm chart for creating the Tanzu Postgress instance for Kubernetes instance.                                                                 tanzusmgr

TIP: Use 'cf marketplace -e SERVICE_OFFERING' to view descriptions of individual plans of a given service offering.

$ cf create-service tanzu-postgres small my-postgres
Creating service instance my-postgres in org donald / space dev as donald...
OK

Create in progress. Use 'cf services' or 'cf service my-postgres' to check operation status.

$ kubectl config set-context --namespace=tsmgr-b50074c0-34f9-4858-8dd2-26b34bfa1a0b
error: you must specify a non-empty context name or --current
./tmp/tanzu_service_manager$ kubectl config set-context --namespace=tsmgr-b50074c0-34f9-4858-8dd2-26b34bfa1a0b --current
Context "services-admin@services" modified.
./tmp/tanzu_service_manager$ kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/tanzu-postgres-0           1/1     Running   0          66s
pod/tanzu-postgres-monitor-0   1/1     Running   0          66s

NAME                           TYPE           CLUSTER-IP     EXTERNAL-IP                                                              PORT(S)          AGE
service/tanzu-postgres         LoadBalancer   100.69.90.76   ac585b5fa1eac40e09ec85c57bf62865-661569893.eu-west-2.elb.amazonaws.com   5432:31147/TCP   66s
service/tanzu-postgres-agent   ClusterIP      None           <none>                                                                   <none>           66s

NAME                                      READY   AGE
statefulset.apps/tanzu-postgres           1/1     66s
statefulset.apps/tanzu-postgres-monitor   1/1     66s

NAME                                           STATUS    AGE
postgres.sql.tanzu.vmware.com/tanzu-postgres   Running   66s

$ cf s
Getting services in org donald / space dev as donald...

name          service          plan    bound apps   last operation     broker      upgrade available
my-postgres   tanzu-postgres   small                create succeeded   tanzusmgr   no
$ cf bind-service spring-music my-postgres
Binding service my-postgres to app spring-music in org donald / space dev as donald...
OK

TIP: Use 'cf restage spring-music' to ensure your env variable changes take effect

</pre>
