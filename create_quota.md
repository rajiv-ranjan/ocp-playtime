Switch to the system:admin user to set a quota on user "prag":

```sh
oc login -u system:admin
```

Create a limited cluster resource quota:

```sh
export OCP_USERNAME=prag

oc create clusterquota clusterquota-${OCP_USERNAME} \
 --project-annotation-selector=openshift.io/requester=$OCP_USERNAME \
 --hard pods=25 \
 --hard requests.memory=6Gi \
 --hard requests.cpu=5 \
 --hard limits.cpu=25  \
 --hard limits.memory=40Gi \
 --hard configmaps=25 \
 --hard persistentvolumeclaims=25  \
 --hard services=25
```

View the cluster resource quota:

```sh
oc get clusterresourcequota
```

Describe the cluster resource quota:

```sh
oc describe clusterresourcequota clusterquota-prag
```
