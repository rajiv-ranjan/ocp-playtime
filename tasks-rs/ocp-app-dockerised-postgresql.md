## App deployment via s2i buildConfig. Postgresql outside OCP.

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
oc tag openshift/jboss-eap71-openshift:1.2 task-rs-non-ocp-psql/jboss-eap71-openshift:1.2
oc tag openshift/jboss-eap71-openshift:1.3 task-rs-non-ocp-psql/jboss-eap71-openshift:1.3
oc tag openshift/jboss-eap71-openshift:latest task-rs-non-ocp-psql/jboss-eap71-openshift:latest
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
Log back as developer

```sh
oc login -u developer -p developer
```
Create the build config to process binary (war in this case)

```sh
oc new-app jboss-eap71-openshift~https://github.com/rajiv-ranjan/ocp-playtime.git#master \
--context-dir=tasks-rs \
--env-file=ocp/app-variables.env \
--build-env-file=ocp/build-variables.env \
--labels=name=tasklist \
--name=task-rs
```
Sample output

```sh
--> Found image 810ff88 (2 months old) in image stream "task-rs-non-ocp-psql/jboss-eap71-openshift" under tag "latest" for "jboss-eap71-openshift"

    JBoss EAP 7.1
    -------------
    Platform for building and running JavaEE applications on JBoss EAP 7.1

    Tags: builder, javaee, eap, eap7

    * A source build using source code from https://github.com/rajiv-ranjan/ocp-playtime.git#master will be created
      * The resulting image will be pushed to image stream "task-rs:latest"
      * Use 'start-build' to trigger a new build
    * This image will be deployed in deployment config "task-rs"
    * Ports 8080/tcp, 8443/tcp, 8778/tcp will be load balanced by service "task-rs"
      * Other containers can access this service through the hostname "task-rs"

--> Creating resources with label name=tasklist ...
    imagestream "task-rs" created
    buildconfig "task-rs" created
    deploymentconfig "task-rs" created
    service "task-rs" created
--> Success
    Build scheduled, use 'oc logs -f bc/task-rs' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/task-rs'
    Run 'oc status' to view your app.
```
Expose the service create

```sh
oc expose svc/task-rs
```
Test the application running

```sh
curl -i -u 'quickstartUser:quickstartPwd1!' -H "Content-Length: 0" -X POST http://task-rs-task-rs-non-ocp-psql.192.168.64.9.nip.io/tasks-rs/tasks/title/buyMilk
curl -i -u 'quickstartUser:quickstartPwd1!' -H "Content-Length: 0" -X POST http://task-rs-task-rs-non-ocp-psql.192.168.64.9.nip.io/tasks-rs/tasks/title/buyFruits
curl -H "Accept: application/json" -u 'quickstartUser:quickstartPwd1!' -X GET http://task-rs-task-rs-non-ocp-psql.192.168.64.9.nip.io/tasks-rs/tasks/title | jq
```


```sh
curl -i -u 'quickstartUser:quickstartPwd1!' -X DELETE http://task-rs-task-rs-non-ocp-psql.192.168.64.9.nip.io.192.168.64.9.nip.io:8080/tasks-rs/tasks/id/1
```
