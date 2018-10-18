# Deploying hello world application via binary deployment

## 

Compile and package the code base.

```sh
git clone https://github.com/rajiv-ranjan/ocp-playtime.git
cd ocp-playtime
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
oc new-project helloword-rs-binary --display-name="Binary Deployment- Hello World Rest Api"
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
Sample output

```sh
--> Found image 810ff88 (2 months old) in image stream "helloword-rs-binary/jboss-eap71-openshift" under tag "latest" for "jboss-eap71-openshift"

    JBoss EAP 7.1
    -------------
    Platform for building and running JavaEE applications on JBoss EAP 7.1

    Tags: builder, javaee, eap, eap7

    * A source build using binary input will be created
      * The resulting image will be pushed to image stream "eap-app:latest"
      * A binary build was created, use 'start-build --from-dir' to trigger a new build

--> Creating resources with label build=eap-app ...
    imagestream "eap-app" created
    buildconfig "eap-app" created
--> Success
```

Start the build process

```sh
oc start-build eap-app --from-dir=./ocp --follow
```
Sample output

```sh
Uploading directory "ocp" as binary input for the build ...
build "eap-app-1" started
Receiving source from STDIN as archive ...
Copying all war artifacts from /tmp/src directory into /opt/eap/standalone/deployments for later deployment...
Copying all ear artifacts from /tmp/src directory into /opt/eap/standalone/deployments for later deployment...
Copying all rar artifacts from /tmp/src directory into /opt/eap/standalone/deployments for later deployment...
Copying all jar artifacts from /tmp/src directory into /opt/eap/standalone/deployments for later deployment...
Copying all war artifacts from /tmp/src/deployments directory into /opt/eap/standalone/deployments for later deployment...
'/tmp/src/deployments/helloworld-rs.war' -> '/opt/eap/standalone/deployments/helloworld-rs.war'
Copying all ear artifacts from /tmp/src/deployments directory into /opt/eap/standalone/deployments for later deployment...
Copying all rar artifacts from /tmp/src/deployments directory into /opt/eap/standalone/deployments for later deployment...
Copying all jar artifacts from /tmp/src/deployments directory into /opt/eap/standalone/deployments for later deployment...
'/tmp/src/deployments/helloworld-rs-sources.jar' -> '/opt/eap/standalone/deployments/helloworld-rs-sources.jar'
Pushing image 172.30.1.1:5000/helloword-rs-binary/eap-app:latest ...
Pushed 1/7 layers, 15% complete
Pushed 2/7 layers, 29% complete
Pushed 3/7 layers, 45% complete
Pushed 3/7 layers, 59% complete
Pushed 3/7 layers, 73% complete
Pushed 4/7 layers, 85% complete
Pushed 5/7 layers, 93% complete
Pushed 6/7 layers, 99% complete
Pushed 7/7 layers, 100% complete
Push successful
```

Create the deployment config

```sh
oc new-app eap-app
```

Get the service and expose it via route

```sh
oc get svc -o name
oc expose svc eap-app --port=8080
oc get routes
```

Check of the application is working as expected.

```sh
curl -i -k http://eap-app-helloword-rs-binary.192.168.64.9.nip.io/helloworld-rs/rest/json
```

Sample output

```http
HTTP/1.1 200 OK
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Content-Type: application/json
Content-Length: 25
Date: Wed, 17 Oct 2018 19:02:11 GMT
Set-Cookie: b178ea2c879fca74fe5371c8f5297153=1e7e61fbf2c2c40dbd962cb2415ca92e; path=/; HttpOnly
Cache-control: private

{"result":"Hello World!"}
```