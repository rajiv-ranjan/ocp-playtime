Create a new project:

```sh
oc new-project test-hpa
```

Deploy hello-openshift in the new project:

```sh
oc new-app openshift/hello-openshift:v3.9 -n test-hpa
oc expose svc hello-openshift
```

Create a LimitRange object:

```sh
echo '{
    "kind": "LimitRange",
    "apiVersion": "v1",
    "metadata": {
        "name": "limits",
        "creationTimestamp": null
    },
    "spec": {
        "limits": [
            {
                "type": "Pod",
                "max": {
                    "cpu": "100m",
                    "memory": "750Mi"
                },
                "min": {
                    "cpu": "10m",
                    "memory": "5Mi"
                }
            },
            {
                "type": "Container",
                "max": {
                    "cpu": "100m",
                    "memory": "750Mi"
                },
                "min": {
                    "cpu": "10m",
                    "memory": "5Mi"
                },
                "default": {
                    "cpu": "50m",
                    "memory": "100Mi"
                }
            }
        ]
    }
}' | oc create -f - -n test-hpa
```

Create an HPA for the hello-openshift deployment to scale between one and five replicas and set it to scale up when the CPU utilization reaches 80%:

```sh
oc autoscale dc/hello-openshift --min 1 --max 5 --cpu-percent=80
```

List the status of the autoscaler:

```sh
oc get hpa/hello-openshift -n test-hpa
```

Redeploy the application to pick up the default request from the LimitRange object:

```sh
oc rollout latest hello-openshift -n test-hpa
```

In a separate window, create work for the pod and monitor the environment.Run this command in a few windows concurrently to produce more work for the pods(at least three parallel windows to generate enough load):

```sh
ROUTE=$(oc get route hello-openshift --template "{{ .spec.host }}")
for time in {1..15000}
  do
   echo time $time
   curl ${ROUTE}
  done
```
