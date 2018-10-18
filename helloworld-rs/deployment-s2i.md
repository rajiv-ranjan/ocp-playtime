#Deploying hello world application via s2i

## Using s2i template available in OpenShift
Developer is expected to commit code in the SCM like Git. Openshift can pull the source code; use the s2i template to provision all openshift and kubernetes objects.

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
oc new-project helloword-rs-dev --display-name="Dev - Hello World Rest Api"
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
oc tag openshift/jboss-eap71-openshift:1.3 helloword-rs-dev/jboss-eap71-openshift:1.3
oc tag openshift/jboss-eap71-openshift:latest helloword-rs-dev/jboss-eap71-openshift:latest
```

Create the application based s2i template

```sh
oc new-app --template=eap71-basic-s2i \
 -p IMAGE_STREAM_NAMESPACE="helloword-rs-dev" \
 -p SOURCE_REPOSITORY_URL="https://github.com/rajiv-ranjan/ocp-playtime" \
 -p SOURCE_REPOSITORY_REF="master" \
 -p CONTEXT_DIR="helloworld-rs"
```


Curl & check if the application is up and running

```sh
curl http://eap-app-helloword-rs-dev.192.168.64.9.nip.io/helloworld-rs/rest/json
```

```json
{"result":"Hello World!"}
```