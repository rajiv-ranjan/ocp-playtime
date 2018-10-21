
Obtaining OpenShift Container Platform packages:
On the RHEL 7 server with an internet connection, sync the repositories:
Ensure that the packages are not deleted after you sync the repository, import the GPG key:
```sh
$ rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

Install required packages:
```sh
$ sudo yum -y install yum-utils createrepo docker git
```

Make a directory to store the software in the server’s storage or to a USB drive or other external device:
```sh
$ mkdir -p </path/to/repos>
```

Sync the packages and create the repository for each of them:
```sh
$ for repo in \
rhel-7-server-rpms \
rhel-7-server-extras-rpms \
rhel-7-server-ansible-2.6-rpms \
rhel-7-server-ose-3.11-rpms
do
  reposync --gpgcheck -lm --repoid=${repo} --download_path=</path/to/repos> 
  createrepo -v </path/to/repos/>${repo} -o </path/to/repos/>${repo} 
done
```

Obtaining images
Start the Docker daemon:
```sh
$ systemctl start docker
```

Pull the Red Hat-certified Source-to-Image (S2I) builder images . Make sure to indicate the correct tag by specifying the version number.  FYI OpenShift and Atomic Platform Tested Integrations page .

```sh
$ docker pull registry.redhat.io/jboss-eap-6/eap64-openshift:<tag>
$ docker pull registry.redhat.io/jboss-eap-7/eap70-openshift:<tag>
$ docker pull registry.redhat.io/jboss-webserver-3/webserver31-tomcat7-openshift:<tag>
$ docker pull registry.redhat.io/jboss-webserver-3/webserver31-tomcat8-openshift:<tag>
$ docker pull registry.redhat.io/rhscl/mongodb-32-rhel7:<tag>
$ docker pull registry.redhat.io/rhscl/mysql-57-rhel7:<tag>
$ docker pull registry.redhat.io/rhscl/php-56-rhel7:<tag>
$ docker pull registry.redhat.io/rhscl/postgresql-95-rhel7:<tag>
$ docker pull registry.redhat.io/redhat-openjdk-18/openjdk18-openshift:<tag>
$ docker pull registry.redhat.io/rhscl/mariadb-101-rhel7:<tag>
```

Exporting images
Create a directory to store your compressed images in and change to it:
```sh
$ mkdir </path/to/images>
$ cd </path/to/images>
```

Export the S2I builder images . 
```sh
docker save -o ose3-builder-images.tar \
    registry.redhat.io/jboss-webserver-3/webserver31-tomcat7-openshift:<tag> \
    registry.redhat.io/jboss-webserver-3/webserver31-tomcat8-openshift:<tag> \
    registry.redhat.io/jboss-eap-7/eap70-openshift:<tag> \
    registry.redhat.io/jboss-eap-6/eap64-openshift:<tag \
    registry.redhat.io/rhscl/mongodb-32-rhel7:<tag> \
    registry.redhat.io/rhscl/mysql-57-rhel7:<tag> \
    registry.redhat.io/rhscl/php-56-rhel7:<tag> \
    registry.redhat.io/rhscl/postgresql-95-rhel7:<tag> \
    registry.redhat.io/redhat-openjdk-18/openjdk18-openshift:<tag> \
    registry.redhat.io/rhscl/mariadb-101-rhel7:<tag> 
```

Copy the compressed files from your Internet-connected host to your internal host.
Load the images that you copied:
```sh
$ docker load -i ose3-builder-images.tar
```

Prepare and populate the repository server
Prepare the webserver, install Apache on the server:

```sh
$ sudo yum install httpd
```

Place the repository files into Apache’s root folder.
```sh
$ mv /path/to/repos /var/www/html/
$ chmod -R +r /var/www/html/repos
$ restorecon -vR /var/www/html
```

Add the firewall rules:
```sh
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --reload
```

Enable and start Apache for the changes to take effect:
```sh
$ systemctl enable httpd
$ systemctl start httpd
```

Populate the registry
Before pushing the images into the registry, re-tag each image.
```sh
$ docker tag registry.redhat.io/rhscl/mariadb-101-rhel7:<tag> registry.example.com/rhscl/mariadb-101-rhel7:<tag>
```

Push each image into the registry.

```sh
$ docker push registry.example.com/rhscl/mariadb-101-rhel7:<tag>
```

— If internet connection can be activated for registry server : 

# from registry.access.redhat.com
for image in rhel7/etcd rhscl/postgresql-96-rhel7 jboss-eap-7/eap70-openshift
do
  skopeo copy --dest-tls-verify=false docker://registry.access.redhat.com/$image:latest docker://localhost:5000/${image}:latest
done






