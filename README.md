# RedHat ex280v42 Preparation Guide

## What it is

This preparation guide is written for Red Hat Certified Specialist in OpenShift Administration exam for OpenShift V4.2.

- Exam format: task based. The exam itself contains a set of tasks (16) that you have to perform, which represents the tasks for an OpenShift administrator.

- Exam duration: 3 hours

- Exam passing grade: 70% - you must got 12 questions right out of 16. (note that partial completion of a question is not counted as a correct answer)

## Exam objectives

These objectives was retrieved from [https://www.redhat.com/en/services/training/ex280-red-hat-certified-specialist-in-openshift-administration-exam?section=Objectives](https://www.redhat.com/en/services/training/ex280-red-hat-certified-specialist-in-openshift-administration-exam?section=Objectives).

To become a Red Hat Certified Specialist in OpenShift Administration, you should be able to perform these tasks:

- Manage OpenShift Container Platform
	- Use the command-line interface to manage and configure an OpenShift cluster
	- Use the web console to manage and configure an OpenShift cluster
	- Create and delete projects
	- Import, export, and configure Kubernetes resources
	- Examine resources and cluster status
	- View logs
	- Monitor cluster events and alerts
	- Troubleshoot common cluster events and alerts
	- Use product documentation
- Manage users and policies
	- Configure the HTPasswd identity provider for authentication
	- Create and delete users
	- Modify user passwords
	- Modify user and group permissions
	- Create and manage groups
- Control access to resources
	- Define role-based access controls
	- Apply permissions to users
	- Create and apply secrets to manage sensitive information
	- Create service accounts and apply permissions using security context constraints
- Configure networking components
	- Troubleshoot software defined networking
	- Create and edit external routes
	- Control cluster network ingress
	- Create a self signed certificate
	- Secure routes using TLS certificates
- Configure pod scheduling
	- Limit resource usage
	- Scale applications to meet increased demand
	- Control pod placement across cluster nodes
- Configure cluster scaling
	- Manually control the number of cluster workers
	- Automatically scale the number of cluster workers

## Manage OpenShift Container Platform

This topic covers around 15%.

Executing troubleshooting commands:

- Getting node informations:
```
oc get node
oc describe node <nodename>
```

- Getting busiest nodes
```
oc adm top nodes
```

- Getting `journalctl` logs from a node
```
oc adm node-logs -u kubelet my-node-name
```

- Running a remote shell for a node
```
oc debug node/<nodename>
```

- Work with cluster installers
```
oc get clusteroperators
oc get clusterversion -o yaml
```

- Getting a pod logs
```
oc logs <pod> [-c <container>] [-f]
```

- Debugging a deployment or a pod
```
oc debug deployment/<deplname> [--as-root]
oc rsh <podname>
oc port-forward <pod> <localport>:<remoteport>
```

- Getting a file from a pod
```
oc cp file <pod>:/file
oc cp <pod>:/file file
```

## Manage Users and Policies

This topic covers around 15%.

Removing the default kubeadmin:
```
oc delete secret kubeadmin -n kube-system
```

Working with htpasswd

- Create: `htpasswd -c -B -b /tmp/htpasswd student redhat123`
- Update: `htpasswd -b /tmp/htpasswd student redhat1234`

Create a secret:
```
oc create secret generic htpasswd-secret \
> --from-file htpasswd=/tmp/htpasswd -n openshift-config
```

Adding to OAuth:
```
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpasswd-secret
```

Getting users and identities
```
oc get users
oc get identity
```

## Manage Resource Access

This topic covers around 15%.

Defining and Applying Permissions Using RBAC  
```
oc adm policy add-cluster-role-to-user cluster-admin username
oc adm policy remove-cluster-role-from-user cluster-admin username
oc adm policy who-can delete user
oc adm policy add-role-to-user basic-user dev -n wordpress
```

Default roles
-	**basic-user** Users with this role have read access to the project.
-	**cluster-admin** Users with this role have superuser access to the cluster resources. These users can perform any action on the cluster, and have full control of all projects.
-	**cluster-status** Users with this role can get cluster status information.
-	**self-provisioner** Users with this role can create new projects.

Default roles that can be added or removed from a project level:
-	**admin** Users with this role can manage all project resources, including granting access to other users to the project.
-	**edit** Users with this role can create, change, and delete common application resources from the project, such as services and deployment configurations. These users cannot act on management resources such as limit ranges and quotas, and cannot manage access permissions to the project.
-	**view** Users with this role can view project resources, but cannot modify project resources.

System users: system:admin, system:openshift-registry, and system:node:node1.example.com.

Managing Sensitive Information with Secrets  
```
oc create secret generic secret_name \
> --from-literal key1=secret1 \
> --from-literal key2=secret2
oc secrets add --for mount serviceaccount/serviceaccount-name \
> secret/secret_name
oc set env dc/demo --from=secret/demo-secret
oc set volume dc/demo \
> --add \
> --type=secret \
> --secret-name=demo-secret \
> --mount-path=/app-secrets
```

Controlling Application Permissions with Security Context Constraints (SCCs) (anyuid, privileged etc)  
```
oc adm policy add-scc-to-user anyuid -z default
```


## Configure Networking Components

This topic covers around 15%.

Service types: ClusterIP, NodePort, LoadBalancer, ExternalService
```
oc describe dns.operator/default
```
CoreDNS entries:
•	A `svcname.namespace.svc.cluster.local`
•	SRV `_port-name._port-protocol.svc.namespace.svc.cluster.local`

```
oc get Network.config.openshift.io cluster -o yaml
```

Ingress rule:
```
oc get ingress
```

Certificate generation:
```
openssl genrsa -out file.key
openssl req -new -subj <subject> -out file.req -key file.key
openssl x509 -req -in file.req -out file.crt -signkey file.key
```

Secure route (edge/passthru):
```
oc create route edge \
> --service <svcname> --hostname <host>.apps.acme.com \
> --key file.key --cert file.crt
```

## Configure Pod Scheduling

This topic covers around 30%.

Controlling pod scheduling behavior (factors that can affect on which nodes a pod can or cannot be run)  
```
oc label node node1.us-west-1.compute.internal env[-|=dev] [--overwrite]
oc get node node2.us-west-1.compute.internal --show-labels
oc get node -L failure-domain.beta.kubernetes.io/region
oc patch deployment/myapp --patch \
> '{"spec":{"template":{"spec":{"nodeSelector":{"env":"dev"}}}}}'
oc adm new-project demo --node-selector "tier=1"
oc annotate namespace demo \
> openshift.io/node-selector="tier=2" --overwrite
```

Limiting resource usage (factors that can affect the resources that a pod is allowed use or run)  
```
oc adm top nodes -l node-role.kubernetes.io/worker
oc set resources deployment hello-world-nginx \
> --requests cpu=10m,memory=20Mi --limits cpu=80m,memory=100Mi
oc create quota dev-quota --hard services=10,cpu=1300,memory=1.5Gi -n <ns>
oc get resourcequota -n <ns>
oc describe limitrange dev-limits
```

A violation of LimitRange constraints prevents pod creation, and resulting error messages are displayed. A violation of ResourceQuota constraints prevents a pod from being scheduled to any node. The pod might be created but remain in the pending state

```
oc create clusterquota user-qa \
> --project-annotation-selector openshift.io/requester=qa \
> --hard pods=12,secrets=20
oc create clusterquota env-qa \
> --project-label-selector environment=qa \
> --hard pods=10,services=5
```

Scaling an Application  
```
oc scale --replicas 3 deployment/myapp
oc autoscale dc/hello --min 1 --max 10 --cpu-percent 80
oc get hpa
```

## Configure Cluster Scaling

This topic covers around 10% questions

Manually Scaling an OpenShift Cluster  
```
oc scale --replicas=2 \
> machineset MACHINE-SET -n openshift-machine-api
```

Automatically Scaling an OpenShift Cluster
```
oc get clusterautoscaler
oc get machineautoscaler -n openshift-machine-api
```
