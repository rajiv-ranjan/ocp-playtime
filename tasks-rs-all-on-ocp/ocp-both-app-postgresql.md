## App and Postgresql deployment via s2i template

Copy the user and role property file to configuration folder.


Running a local nexus for quicker build of maven projects.

```sh
docker run -d -p 8081:8081 --name nexus sonatype/nexus3
curl -u admin:admin123 http://localhost:8081/service/metrics/ping
```

```sh
pong
```

Compile and package the code base.

```sh
git clone https://github.com/rajiv-ranjan/ocp-playtime.git
cd tasks-rs-all-on-ocp
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
oc new-project task-rs-all-on-ocp --display-name="Full App on OCP" --description="s2i Deployment - Task list management where both app and postgresql are on ocp"
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
oc tag openshift/jboss-eap71-openshift:1.2 task-rs-all-on-ocp/jboss-eap71-openshift:1.2
oc tag openshift/jboss-eap71-openshift:1.3 task-rs-all-on-ocp/jboss-eap71-openshift:1.3
oc tag openshift/jboss-eap71-openshift:latest task-rs-all-on-ocp/jboss-eap71-openshift:latest
```
Make sure the builder image is available in the developers namespace.

```sh
oc get is  | grep ^jboss-eap71 | cut -f1 -d ' '
```
Log back as developer

```sh
oc login -u developer -p developer
```
Add the keystore file that will be used by the EAP instance.

```sh
#follow the instructions
keytool -genkey -keyalg RSA -alias eapdemo-selfsigned -keystore keystore.jks -validity 360 -keysize 2048
oc create secret generic  eap7-app-secret --from-file=keystore.jks
```
Check the details of secret object created.

```sh
oc describe secret eap7-app-secret
```
Sample output

```sh
Name:         eap7-app-secret
Namespace:    task-rs-all-on-ocp
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
keystore.jks:  2255 bytes
```
Create the build config to process binary (war in this case)

```sh
oc new-app --template=eap71-postgresql-s2i \
-p SOURCE_REPOSITORY_URL="https://github.com/rajiv-ranjan/ocp-playtime" \
-p SOURCE_REPOSITORY_REF="master" \
-p CONTEXT_DIR='tasks-rs-all-on-ocp' \
-p APPLICATION_NAME="task-rs" \
-p DB_JNDI=java:jboss/datasources/TasksRsQuickstartDSDockerPostgresql \
-p DB_DATABASE=tasksRsXmlQuickStart \
-p DB_USERNAME='dbUser' \
-p DB_PASSWORD='dbUserPassword#123' \
-p MAVEN_ARGS_APPEND=-Pjboss-community-repository \
-p POSTGRESQL_IMAGE_STREAM_TAG=9.5 \
--labels=name=task-rs-all-on-ocp


-p IMAGE_STREAM_NAMESPACE="task-rs-all-on-ocp" \

oc new-app jboss-eap71-openshift~https://github.com/rajiv-ranjan/ocp-playtime.git#master \
--context-dir=tasks-rs \
--env-file=ocp/app-variables.env \
--build-env-file=ocp/build-variables.env \
--labels=name=tasklist \
--name=task-rs
```
Sample output

```sh

```
Expose the service create

```sh
oc expose svc/task-rs
```
Test the application running

```sh
curl -i -u 'quickstartUser:quickstartPwd1!' -H "Content-Length: 0" -X POST http://task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io/tasks-rs-all-on-ocp/tasks/title/buyMilk
curl -i -u 'quickstartUser:quickstartPwd1!' -H "Content-Length: 0" -X POST http://task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io/tasks-rs-all-on-ocp/tasks/title/buyFruits
curl -H "Accept: application/json" -u 'quickstartUser:quickstartPwd1!' -X GET http://task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io/tasks-rs-all-on-ocp/tasks/title | jq
```


```sh
curl -i -u 'quickstartUser:quickstartPwd1!' -X DELETE http://task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io.192.168.64.9.nip.io:8080/tasks-rs/tasks/id/1
```
