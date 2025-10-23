                                                            Components
       
Namespace:

We need a specific namespace for running the ETCD backup pods. For this purpose, we would be using “default” namespace that is already available in the Openshift cluster. Later in the cronjob, we will be creating pods to perform the actual task of backing up the cluster and these pods are scheduled in the Namespace we pick here.

![image](https://github.ec.va.gov/itops/vapo-governance/assets/15054/475f50dc-02ce-46e8-b244-f65311bf3fd1)

ServiceAccount:

A ServiceAccount is needed, which will be responsible for performing backup commands for the master nodes. For this purpose, we will be using an existing ServiceAccount “approver” which is already present in the cluster and available in the “default” namespace. We will be adding the ServiceAccount details to the ACM policy anyways to avoid some cases of missing ServiceAccount in the cluster.

![image](https://github.ec.va.gov/itops/vapo-governance/assets/15054/2be7be29-2ef3-4147-81b8-d964a6021e25)

ClusterRoleBinding:

We need to create a ClusterRoleBinding to link the ServiceAccount and Cluster Role. We haven’t mentioned about the Cluster Role in this document, as we will be using an existing one. To achieve the backup, we would be using a “cluster-admin” Role.
We will be creating a new ClusterRoleBinding with name “cluster-admin-approver-sa-crb” to link the Service Account and Cluster Role.

![image](https://github.ec.va.gov/itops/vapo-governance/assets/15054/373f49e3-a090-4250-a52a-cc4a477643ad)

ClusterRole:

As a security measure, the service account created earlier can not have excessive permissions on the cluster, so you must create a Cluster Role with specific permissions for running the backup.

This Cluster Role has specific permissions to specific resources and verbs, such as:

     Resource: Nodes
     Verbs: Get and List
     Resource: pods and pods/log
     Verbs: Get, List, Create, Delete, and Watch

![image](https://github.ec.va.gov/itops/vapo-governance/assets/15054/de13ff5f-e82a-49a8-b04d-e11b666d2547)


PVC:

By default, whenever a new backup is created it will be saved in a directory on control plane host. To make it more easily accessible in case of complete cluster failure, it will be better using a PVC. To achieve the same, we will be creating a new PVC in each cluster we intend to take a backup and limit this size to 20Gi.
We have a code logic in place, which keeps the most recent 5 backup files and delete the old backups automatically when the cronjob is ran. This will allow us to keep our PVC space available for continuous backups.

![image](https://github.ec.va.gov/itops/vapo-governance/assets/15054/30a9d4d7-8bb5-4f23-b2a4-89a0e8393238)

ConfigMap:

To achieve the backup, we will be executing a “cluster-backup.sh” which is present on the Master nodes by default.
In this step, we will be creating a new ConfigMap and execute the script mentioned above. This configmaps will also takes care of copying the backup files to the PVC.

![image](https://github.ec.va.gov/itops/vapo-governance/assets/15054/a38c8b48-3cda-417c-bf38-850b39770b15)

Cronjob:

Finally, we need to create the actual cronjob which uses OpenShift client image to create the backup and debug pods.
The backup will be created in the location mentioned and make sure the pvc has only latest 5 backups and the older backups are deleted.
This cronjob will be executed every 6 hours starting at 12:00 AM.

ACM Policy:

Once we have all the above-mentioned documents, we need to create the ACM policy to enforce it on all the clusters without manual intervention.
Just like all other policies, we will follow the standard approach to create the ACM policy. Below is the link to ACM policy in SBX environment. Similar policy will be created in PPD and PROD.

https://github.ec.va.gov/itops/vapo-governance/tree/sandbox/acm-git-policies/CM-Configuration-Management/etcd-backup

Debugging:

Once the job is run at scheduled time, we can validate the pvc content by creating a sample pod.
Once the pod is created, we can login using terminal and access pvc contents. We would be seeing recent 5 backups, and, in each backup, we will have two files, a snapshot and kuberesources.

![image](https://github.ec.va.gov/itops/vapo-governance/assets/15054/1d8012ce-eb54-4795-9248-430458109726)

![image](https://github.ec.va.gov/itops/vapo-governance/assets/15054/21056942-7a4e-4924-9cca-beb897291bc0)
