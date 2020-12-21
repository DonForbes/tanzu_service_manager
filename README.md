# tanzu_service_manager
Repository holding configuration used in the setup of the Tanzu Service Manager and the offers for various services.
Introduction
This example includes a Helm chart for the Tanzu Postgres Operator Instance.

Prerequisites
TAS environment with Tanzu Service Manager installed and configured.
Connection setup for the tsmgr CLI to properly target an environment
export TSMGR_TARGET=http://<change_by_your_tsmgr_server>:<change_by_your_tsmgr_server_port>
export TSMGR_TOKEN=<change_by_your_tsmgr_kubernetes_token>
export TSMGR_INSECURE=true # if using a lab environment
Kubernetes cluster registered with TSMGR and set as default
tsmgr cluster register my-cluster-name my-cluster-creds-file.yaml
tsmgr cluster set-default my-cluster-name
Installing the Tanzu Postgres Operator for Kubernetes
Please follow the steps to install the Tanzu Postgres Operator for Kubernetes to your cluster from here. NOTE: the dockerRegistrySecretName in values.yaml file must be set to the registry-secret value instead of regsecret, matching the TSMGR requirement.

Validate your operator is working by running:

 $ kubectl get all
 NAME                                     READY   STATUS    RESTARTS   AGE
 pod/postgres-operator-7456f6b775-m5xnv   1/1     Running   0          24h

 NAME                                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
 service/kubernetes                          ClusterIP   10.80.0.1             443/TCP   28h
 service/postgres-operator-webhook-service   ClusterIP   10.80.2.116           443/TCP   24h

 NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
 deployment.apps/postgres-operator   1/1     1            1           24h

 NAME                                           DESIRED   CURRENT   READY   AGE
 replicaset.apps/postgres-operator-7456f6b775   1         1         1       24h
 
Saving the Offer
Once the operator is working, download the postgres-oper-instance-0.1.0.tgz helm chart and postgres-oper-inst-offer.tgz TSMGR offer files to a new empty folder.

Untar the postgres-oper-inst-offer.tgz file using the command:

tar xvzf postgres-oper-inst-offer.tgz
The offer will provide the TSMGR files as plans, overrides and bind template. Please verify those files and customize them if you need to set any specifics.

From the root level of the new folder where you saved both files, save the offer:

$ tsmgr offer save postgres-oper-inst-offer/ postgres-oper-instance-0.1.0.tgz
Enabling CF access
The offer access is not available by default in cf. You can verify that by calling the follow commands. Notice that postgres is not available in the marketplace, even though it is listed by service-access (with access=none):
