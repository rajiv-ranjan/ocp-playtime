To instantiate the postgres instance

```sh
docker run -d --name local-postgres --env POSTGRESQL_USER='dbUser' --env POSTGRESQL_PASSWORD='dbUserPassword#123' --env POSTGRESQL_DATABASE='tasksRsXmlQuickStart' --env POSTGRESQL_ADMIN_PASSWORD='dbAdminPassword123' -p 5432:5432 registry.access.redhat.com/rhscl/postgresql-95-rhel7:latest
```

To connect to the postgres instance from another app on docker.

```sh
docker run -it --rm --link local-postgres:postgres registry.access.redhat.com/rhscl/postgresql-95-rhel7:latest psql -h postgres -U dbUser tasksRsXmlQuickStart
```

Create the datasource with below details. Please mark the ip of the machine on which dockerised postgresql is running.

```sh
<datasource jta="true" jndi-name="java:jboss/datasources/TasksRsQuickstartDSDockerPostgresql" pool-name="tasks-rs-xml-quickstart-DockerPostgresql" enabled="true" use-ccm="false" statistics-enabled="true">
    <connection-url>jdbc:postgresql://192.168.0.103:5432/tasksRsXmlQuickStart</connection-url>
    <driver-class>org.postgresql.Driver</driver-class>
    <driver>postgresql-9.4.1212.jar</driver>
    <security>
        <user-name>dbUser</user-name>
        <password>dbUserPassword#123</password>
    </security>
    <validation>
        <valid-connection-checker class-name="org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLValidConnectionChecker"/>
        <background-validation>true</background-validation>
        <exception-sorter class-name="org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLExceptionSorter"/>
    </validation>
</datasource>
```



Changed the datasource in the file  **/tasks-rs/src/main/resources/META-INF/persistence.xml** by editing below tag:

```xml
<jta-data-source>${postgresqlDS}</jta-data-source>
```
Make changes to standalone.xml to accept environment variables

```xml
	<subsystem xmlns="urn:jboss:domain:ee:4.0">
		------
		<spec-descriptor-property-replacement>true</spec-descriptor-property-replacement>
		<jboss-descriptor-property-replacement>true</jboss-descriptor-property-replacement>
		------
	</subsystem>
```


Start the EAP standalone passing datasource variable

```sh
./standalone.sh -DpostgresqlDS=java:jboss/datasources/TasksRsQuickstartDSDockerPostgresql
```

To Test the application

Create Tasks

```sh
curl -i -u 'quickstartUser:quickstartPwd1!' -H "Content-Length: 0" -X POST http://localhost:8080/tasks-rs/tasks/title/task1
```

```sh
HTTP/1.1 201 Created
Expires: 0
Cache-Control: no-cache, no-store, must-revalidate
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Pragma: no-cache
Location: http://localhost:8080/tasks-rs/tasks/id/2
Date: Sun, 21 Oct 2018 13:08:57 GMT
Connection: keep-alive
Content-Length: 0
```
Get all the task with id=1 for the user

```sh
 curl -H "Accept: application/json" -u 'quickstartUser:quickstartPwd1!' -X GET http://localhost:8080/tasks-rs/tasks/id/2
```

```sh
{"id":2,"title":"buyMilk","ownerName":"quickstartUser"}
```


Get all the tasks for the user

```sh
 curl -H "Accept: application/json" -u 'quickstartUser:quickstartPwd1!' -X GET http://localhost:8080/tasks-rs/tasks/title | jq
```

```sh
[
  {
    "id": 1,
    "title": "task1",
    "ownerName": "quickstartUser"
  },
  {
    "id": 2,
    "title": "buyMilk",
    "ownerName": "quickstartUser"
  }
]
```
Delete a specific task (id=1) for a user

```sh
curl -i -u 'quickstartUser:quickstartPwd1!' -X DELETE http://localhost:8080/tasks-rs/tasks/id/1
``` 

```sh
HTTP/1.1 204 No Content
Expires: 0
Cache-Control: no-cache, no-store, must-revalidate
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Pragma: no-cache
Date: Sun, 21 Oct 2018 13:11:52 GMT
```

## Stage 2: Take the application to OpenShift and Postgresql runs from the docker container (outside OCP) 

