# ocp-playtime


# Prerequisites
## Red Hat CDK (Minishift)
Instructions

```sh

minishift setup-cdk

minishift profile set cp-poc
➜  ~ minishift  profile list
- minishift	Does Not Exist
- ocp-poc	Does Not Exist	(Active)
➜  ~ minishift config set memory 10240
No Minishift instance exists. New 'memory' setting will be applied on next 'minishift start'
➜  ~ minishift config set disk-size 30G
No Minishift instance exists. New 'disk-size' setting will be applied on next 'minishift start'
➜  ~ minishift config set cpus 4
No Minishift instance exists. New 'cpus' setting will be applied on next 'minishift start'
➜  ~ minishift config view
- cpus                               : 4
- disk-size                          : 30G
- iso-url                            : file:///Users/rajranja/.minishift/cache/iso/minishift-rhel7.iso
- memory                             : 10240
- vm-driver                          : xhyve
➜  ~
```



### Game 1:Create user quota for the cluster. For detailed steps Read [follow](https://github.com/rajiv-ranjan/ocp-playtime/blob/master/create_quota.md "multi-project-quota").
### Game 2:DISCONNECTED installation: Populating the registry with the latest Red Hat-certified Source-to-Image (S2I) builder images. For detailed steps Read [follow](https://github.com/rajiv-ranjan/ocp-playtime/blob/master/populate-registry.md "Populate-Registry").
### Game 3:Sample application deployment with quota. For detailed steps Read [follow](https://github.com/rajiv-ranjan/ocp-playtime/blob/master/Quota_deployment_example.md "deployment-with-quota"). 
### Game 4: Deployment Strategies
There are three deployment strategies explain and demonstrated. 
*   Deployment of application via s2i template in OpenShift. Read further [here](https://github.com/rajiv-ranjan/ocp-playtime/blob/master/helloworld-rs/deployment-from-source-code.md "deployment-from-source-code").
*   Deployment of application from binary (war, jar etc) outside OpenShift. Read further [here](https://github.com/rajiv-ranjan/ocp-playtime/blob/master/helloworld-rs/deployment-from-binary.md "deployment-from-binary").
*   Deployment of application from docker image created outside OpenShift. Read further [here](https://github.com/rajiv-ranjan/ocp-playtime/blob/master/helloworld-rs/deployment-from-docker-image.md "deployment-from-docker-image").

### Game 5:
### Game 6:
### Game 7:Create a Horizontal Pod Autoscaler. For detailed steps Read [follow](https://github.com/rajiv-ranjan/ocp-playtime/blob/master/HPA.md "HPA").

