# Task Restful API with Postgresql 9.5 backend 
## App and Postgresql deployment via s2i template


> for other methods please check here.

Copy the user and role property file to configuration folder.


Running a local nexus for quicker build of maven projects.

```sh
docker run -d -p 8081:8081 --name nexus sonatype/nexus3
curl -u admin:admin123 http://localhost:8081/service/metrics/ping
```

```sh
pong
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
Get the EAP template collection from github 

```sh
git clone https://github.com/jboss-openshift/application-templates.git
cd application-templates/eap
```
Add the s2i template

```sh
oc create -f eap71-basic-s2i.json -n openshift
```

If the template already exists in the *openshift* namespace then tag the same to working namespace

```sh
oc tag openshift/jboss-eap71-openshift:1.2 task-rs-all-on-ocp/jboss-eap71-openshift:1.2
oc tag openshift/jboss-eap71-openshift:1.3 task-rs-all-on-ocp/jboss-eap71-openshift:1.3
oc tag openshift/jboss-eap71-openshift:latest task-rs-all-on-ocp/jboss-eap71-openshift:latest
oc tag openshift/postgresql:9.5 task-rs-all-on-ocp/postgresql:9.5
oc tag openshift/postgresql:latest task-rs-all-on-ocp/postgresql:latest
```
Make sure the builder image is available in the developers namespace.

```sh
oc get is  | grep ^jboss-eap71 | cut -f1 -d ' '
```
Log back as developer

```sh
oc login -u developer -p developer
oc project task-rs-all-on-ocp
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
-p IMAGE_STREAM_NAMESPACE="task-rs-all-on-ocp" \
-p APPLICATION_NAME="task-rs" \
-p DB_JNDI=java:jboss/datasources/TasksRsQuickstartDSDockerPostgresql \
-p DB_DATABASE=tasksRsXmlQuickStart \
-p DB_USERNAME='dbUser' \
-p DB_PASSWORD='dbUserPassword#123' \
-p MAVEN_ARGS_APPEND=-Pjboss-community-repository \
-p POSTGRESQL_IMAGE_STREAM_TAG=9.5 \
--labels=name=task-rs-all-on-ocp
```
Sample output

```sh
--> Deploying template "openshift/eap71-postgresql-s2i" to project task-rs-all-on-ocp

     JBoss EAP 7.1 + PostgreSQL (Ephemeral with https)
     ---------
     An example EAP 7 application with a PostgreSQL database. For more information about using this template, see https://github.com/jboss-openshift/application-templates.

     A new EAP 7 and PostgreSQL based application with SSL support has been created in your project. The username/password for accessing the PostgreSQL database "tasksRsXmlQuickStart" is dbUser/dbUserPassword#123. Please be sure to create the following secrets: "eap7-app-secret" containing the keystore.jks file used for serving secure content; "eap7-app-secret" containing the jgroups.jceks file used for securing JGroups communications.

     * With parameters:
        * Application Name=task-rs
        * Custom http Route Hostname=
        * Custom https Route Hostname=
        * Git Repository URL=https://github.com/rajiv-ranjan/ocp-playtime
        * Git Reference=master
        * Context Directory=tasks-rs-all-on-ocp
        * Database JNDI Name=java:jboss/datasources/TasksRsQuickstartDSDockerPostgresql
        * Database Name=tasksRsXmlQuickStart
        * Queues=
        * Topics=
        * Server Keystore Secret Name=eap7-app-secret
        * Server Keystore Filename=keystore.jks
        * Server Keystore Type=
        * Server Certificate Name=
        * Server Keystore Password=
        * Datasource Minimum Pool Size=
        * Datasource Maximum Pool Size=
        * Datasource Transaction Isolation=
        * PostgreSQL Maximum number of connections=
        * PostgreSQL Shared Buffers=
        * A-MQ cluster password=uFgLRsoF # generated
        * Database Username=dbUser
        * Database Password=dbUserPassword#123
        * Github Webhook Secret=2UiYv7GQ # generated
        * Generic Webhook Secret=Di2ndNf1 # generated
        * ImageStream Namespace=task-rs-all-on-ocp
        * JGroups Secret Name=eap7-app-secret
        * JGroups Keystore Filename=jgroups.jceks
        * JGroups Certificate Name=
        * JGroups Keystore Password=
        * JGroups Cluster Password=mrdIuvRe # generated
        * Deploy Exploded Archives=false
        * Maven mirror URL=
        * Maven Additional Arguments=-Pjboss-community-repository
        * ARTIFACT_DIR=
        * PostgreSQL Image Stream Tag=9.5
        * MEMORY_LIMIT=1Gi

--> Creating resources with label name=task-rs-all-on-ocp ...
    service "task-rs" created
    service "secure-task-rs" created
    service "task-rs-postgresql" created
    service "task-rs-ping" created
    route "task-rs" created
    route "secure-task-rs" created
    imagestream "task-rs" created
    buildconfig "task-rs" created
    deploymentconfig "task-rs" created
    deploymentconfig "task-rs-postgresql" created
--> Success
    Access your application via route 'task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io'
    Access your application via route 'secure-task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io'
    Build scheduled, use 'oc logs -f bc/task-rs' to track its progress.
    Run 'oc status' to view your app.
```
### Test the restful api
Add couple of tasks in the list and get the final list

```sh
curl -i -u 'quickstartUser:quickstartPwd1!' -H "Content-Length: 0" -X POST http://task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io/tasks-rs-all-on-ocp/tasks/title/buyMilk
curl -i -u 'quickstartUser:quickstartPwd1!' -H "Content-Length: 0" -X POST http://task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io/tasks-rs-all-on-ocp/tasks/title/buyFruits
curl -H "Accept: application/json" -u 'quickstartUser:quickstartPwd1!' -X GET http://task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io/tasks-rs-all-on-ocp/tasks/title | jq
```
Delete task 1 as another test of the functionality

```sh
curl -i -u 'quickstartUser:quickstartPwd1!' -X DELETE http://task-rs-task-rs-all-on-ocp.192.168.64.9.nip.io/tasks-rs-all-on-ocp/tasks/id/1
```
