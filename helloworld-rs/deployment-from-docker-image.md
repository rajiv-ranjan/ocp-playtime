
Install s2i by following the steps at https://github.com/openshift/source-to-image. Below is an example. 

```sh
brew install source-to-image
```

Login to the redhat container registry

```sh
docker login registry.access.redhat.com
```

Pull the builder image from redhat registry 

```sh
docker pull registry.access.redhat.com/jboss-eap-7/eap71-openshift
```
Sample output

```sh
Using default tag: latest
latest: Pulling from jboss-eap-7/eap71-openshift
378837c0e24a: Already exists
e17262bc2341: Already exists
0eeb656bc1e6: Pull complete
77187f02cbef: Pull complete
72a204610612: Pull complete
fb9ea86bb958: Pull complete
Digest: sha256:7c253bc286e9c1c3595e70bce167781d55c81169f6965a3a3518a13d90d3eb07
Status: Downloaded newer image for registry.access.redhat.com/jboss-eap-7/eap71-openshift:latest
```

We have two choices to create the docker image

*   Use s2i to build from source code
*   Use s2i to build from binary (war file)

### Method 1: Image Creation from source
If developer/devops engineer wants to create docker image from source code then below step will be helpful. 

```sh
s2i build --pull-policy=never --env=MAVEN_ARGS_APPEND=-Pjboss-community-repository . registry.access.redhat.com/jboss-eap-7/eap71-openshift eap-app
```
 
### Method 2: Image creation from binary
If the developer/devops engineer wants to create docker image directly from compiled code i.e. war file then this method comes useful.

```sh
s2i build --pull-policy=never --env=MAVEN_ARGS_APPEND=-Pjboss-community-repository target registry.access.redhat.com/jboss-eap-7/eap71-openshift eap-app
```

### Method 3: Image creation from SCM
If the developer/devops engineer has already committed the code in SCM (e.g GitHub) then below command will come useful.  

```sh
s2i build --pull-policy=never --env=MAVEN_ARGS_APPEND=-Pjboss-community-repository --context-dir=helloworld-rs https://github.com/rajiv-ranjan/ocp-playtime registry.access.redhat.com/jboss-eap-7/eap71-openshift eap-app
```

Irrespective of which method is followed; below sample output should indicate the successful build.

```sh

Checking if image "registry.access.redhat.com/jboss-eap-7/eap71-openshift" is available locally ...
Checking if image "registry.access.redhat.com/jboss-eap-7/eap71-openshift" is available locally ...
Copying all war artifacts from /tmp/src directory into /opt/eap/standalone/deployments for later deployment...
'/tmp/src/helloworld-rs.war' -> '/opt/eap/standalone/deployments/helloworld-rs.war'
Copying all ear artifacts from /tmp/src directory into /opt/eap/standalone/deployments for later deployment...
Copying all rar artifacts from /tmp/src directory into /opt/eap/standalone/deployments for later deployment...
Copying all jar artifacts from /tmp/src directory into /opt/eap/standalone/deployments for later deployment...
'/tmp/src/helloworld-rs-sources.jar' -> '/opt/eap/standalone/deployments/helloworld-rs-sources.jar'
Copying all war artifacts from /tmp/src/deployments directory into /opt/eap/standalone/deployments for later deployment...
Copying all ear artifacts from /tmp/src/deployments directory into /opt/eap/standalone/deployments for later deployment...
Copying all rar artifacts from /tmp/src/deployments directory into /opt/eap/standalone/deployments for later deployment...
Copying all jar artifacts from /tmp/src/deployments directory into /opt/eap/standalone/deployments for later deployment...
Build completed successfully
```

Check if image is available in docker local registry

```sh
docker images | grep -i --color eap-app
```
Sample output
 
 ```sh
eap-app                                                   latest              4fe3f82d50c9        About a minute ago   1.01GB
 ```
 
 Run the local images to see the output on the local machine

```sh
docker run -p 8080:8080 eap-app:latest
```
 
 ```sh
curl -i -k http://localhost:8080/helloworld-rs/rest/json

HTTP/1.1 200 OK
Connection: keep-alive
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Content-Type: application/json
Content-Length: 25
Date: Thu, 18 Oct 2018 17:19:31 GMT

{"result":"Hello World!"}
```

Check minishift is running and capture docker registry url

```sh
minishift status

Minishift:  Running
Profile:    ocp-poc
OpenShift:  Running (openshift v3.10.45)
DiskUsage:  21% of 29G (Mounted On: /mnt/vda1)
CacheUsage: 1.359 GB (used by oc binary, ISO or cached images)
RHSM: 	    Registered

```

```sh
minishift openshift registry

172.30.1.1:5000
```

When running OpenShift in a single VM, you can reuse the Docker daemon managed by Minishift for other Docker use-cases as well. By using the same Docker daemon as Minishift, you can speed up your local development.

```sh
eval $(minishift docker-env)
```

Test the connection by running the following command. If successful, the shell will print a list of running containers.

```sh
docker ps
```

Now it's time to login to the minishift docker registry. *oc* is logged in as developer

```sh
docker login -u developer -p $(oc whoami -t) $(minishift openshift registry)
```
Create the new OpenShift project
 
```sh
oc new-project helloword-rs-docker --display-name="Docker Deployment - Hello World Rest Api"
```
Tag the image against the OpenShift registry

```sh
docker tag eap-app:latest $(minishift openshift registry)/helloword-rs-docker/eap-app:latest
```
Push the docker image to the registry

```sh
docker push $(minishift openshift registry)/helloword-rs-docker/eap-app:latest
```

Deploy the new application in OpenShift

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
curl -i -k http://eap-app-helloword-rs-docker.192.168.64.9.nip.io/helloworld-rs/rest/json
```

Sample output

```http
HTTP/1.1 200 OK
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Content-Type: application/json
Content-Length: 25
Date: Wed, 18 Oct 2018 19:02:11 GMT
Set-Cookie: b178ea2c879fca74fe5371c8f5297153=1e7e61fbf2c2c40dbd962cb2415ca92e; path=/; HttpOnly
Cache-control: private

{"result":"Hello World!"}
```