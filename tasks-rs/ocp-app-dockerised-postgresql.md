Compile and package the code base.

```sh
git clone https://github.com/rajiv-ranjan/ocp-playtime.git
cd tasks-rs
mvn clean package 
```

Get the EAP template collection from github 

```sh
git clone https://github.com/jboss-openshift/application-templates.git
cd application-templates/eap
```
Login as developer

```sh
oc login -u developer -p developer
```

Create the namespace for the application

```sh
oc new-project task-rs-non-ocp-psql --display-name="Binary Deployment - Task list management where postgresql resides outside ocp"
```

Login as system:admin to load the template in namespace *openshift* 

```sh 
oc login -u system:admin
```
Add the s2i template

```sh
oc create -f eap71-basic-s2i.json -n openshift
```

If the template already exists in the *openshift* namespace then tag the same to working namespeace

```sh
oc tag openshift/jboss-eap71-openshift:1.2 helloword-rs-binary/jboss-eap71-openshift:1.2
oc tag openshift/jboss-eap71-openshift:1.3 helloword-rs-binary/jboss-eap71-openshift:1.3
oc tag openshift/jboss-eap71-openshift:latest helloword-rs-binary/jboss-eap71-openshift:latest
```
Create the deployments folder. This will be referenced for uploading the binary.

```sh
mkdir -p ocp/deployments
```
Copy the compiled code to the deployments folder

```sh
cp target/helloworld-rs.war ocp/deployments
```

Make sure the builder image is available in the developers namespace.

```sh
oc get is  | grep ^jboss-eap71 | cut -f1 -d ' '
```

Create the build config to process binary (war in this case)

```sh
oc new-build --binary=true \                                                                                                                                 
--image-stream=jboss-eap71-openshift \
--name=eap-app
```
