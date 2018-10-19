

Install s2i by following the steps at https://github.com/openshift/source-to-image 

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
### Method 2: Image creation from binary
### Method 3: Image creation from SCM
If the developer has already committed the code in SCM (e.g GitHub) then below command will come useful.  

```sh
s2i build --pull-policy=never --env=MAVEN_ARGS_APPEND=-Pjboss-community-repository --context-dir=helloworld-rs https://github.com/rajiv-ranjan/ocp-playtime registry.access.redhat.com/jboss-eap-7/eap71-openshift eap-app
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