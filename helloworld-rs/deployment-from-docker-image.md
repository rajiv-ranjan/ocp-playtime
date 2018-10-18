

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
  