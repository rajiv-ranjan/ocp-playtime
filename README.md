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
