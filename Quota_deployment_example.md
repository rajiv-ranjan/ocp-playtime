Create a namespace:

```sh
oc create namespace quota-pod-example
```

Create the ResourceQuota:

```sh
Prageetikas-MacBook-Pro:~ pragshar$ cat /Users/pragshar/Downloads/quota-pod.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-demo
spec:
  hard:
    pods: "2"

oc create -f /Users/pragshar/Downloads/quota-pod.yaml --namespace=quota-pod-example
```

View detailed information about the ResourceQuota. The output shows that the namespace has a quota of two Pods, and that currently there are no Pods; that is, none of the quota is used.:
```sh
Prageetikas-MacBook-Pro:~ pragshar$ oc get resourcequota pod-demo --namespace=quota-pod-example --output=yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: 2018-10-20T14:10:07Z
  name: pod-demo
  namespace: quota-pod-example
  resourceVersion: "68875"
  selfLink: /api/v1/namespaces/quota-pod-example/resourcequotas/pod-demo
  uid: d97a0fb0-d471-11e8-8319-3a0a13341011
spec:
  hard:
    pods: "2"
status:
  hard:
    pods: "2"
  used:
    pods: "0"
```

Create the Deployment:

```sh
Prageetikas-MacBook-Pro:~ pragshar$ cat /Users/pragshar/Downloads/quota-pod-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-quota-demo
spec:
  selector:
    matchLabels:
      purpose: quota-demo
  replicas: 3
  template:
    metadata:
      labels:
        purpose: quota-demo
    spec:
      containers:
      - name: pod-quota-demo
        image: nginx

oc create -f /Users/pragshar/Downloads/quota-pod-deployment.yaml --namespace=quota-pod-example
```

View detailed information about the Deployment.The output shows that even though the Deployment specifies three replicas, only two Pods were created because of the quota.:

```sh
Prageetikas-MacBook-Pro:~ pragshar$ oc get deployment pod-quota-demo --namespace=quota-pod-example --output=yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: 2018-10-20T14:11:35Z
  generation: 1
  name: pod-quota-demo
  namespace: quota-pod-example
  resourceVersion: "70134"
  selfLink: /apis/extensions/v1beta1/namespaces/quota-pod-example/deployments/pod-quota-demo
  uid: 0db88a9d-d472-11e8-8319-3a0a13341011
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      purpose: quota-demo
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        purpose: quota-demo
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: pod-quota-demo
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  conditions:
  - lastTransitionTime: 2018-10-20T14:11:35Z
    lastUpdateTime: 2018-10-20T14:11:35Z
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  - lastTransitionTime: 2018-10-20T14:11:35Z
    lastUpdateTime: 2018-10-20T14:11:35Z
    message: 'pods "pod-quota-demo-78c994f776-d76jv" is forbidden: exceeded quota:
      pod-demo, requested: pods=1, used: pods=2, limited: pods=2'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  - lastTransitionTime: 2018-10-20T14:21:36Z
    lastUpdateTime: 2018-10-20T14:21:36Z
    message: ReplicaSet "pod-quota-demo-78c994f776" has timed out progressing.
    reason: ProgressDeadlineExceeded
    status: "False"
    type: Progressing
  observedGeneration: 1
  replicas: 2
  unavailableReplicas: 3
  updatedReplicas: 2
```


