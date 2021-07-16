Spring Music
============

This is a sample application for using database services on [Cloud Foundry](http://cloudfoundry.org) with the [Spring Framework](http://spring.io) and [Spring Boot](http://projects.spring.io/spring-boot/).

This application has been built to store the same domain objects in one of a variety of different persistence technologies - relational, document, and key-value stores. This is not meant to represent a realistic use case for these technologies, since you would typically choose the one most applicable to the type of data you need to store, but it is useful for testing and experimenting with different types of services on Cloud Foundry.

The application use Spring Java configuration and [bean profiles](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html) to configure the application and the connection objects needed to use the persistence stores. It also uses the [Java CFEnv](https://github.com/pivotal-cf/java-cfenv/) library to inspect the environment when running on Cloud Foundry. See the [Cloud Foundry documentation](http://docs.cloudfoundry.org/buildpacks/java/spring-service-bindings.html) for details on configuring a Spring application for Cloud Foundry.

## Building

This project requires a Java version between 8 and 15 to compile. Java 16 and later versions are not yet supported.

To build a runnable Spring Boot jar file, run the following command: 

~~~
$ ./gradlew clean assemble
~~~

## Running the application locally

One Spring bean profile should be activated to choose the database provider that the application should use. The profile is selected by setting the system property `spring.profiles.active` when starting the app.

The application can be started locally using the following command:

~~~
$ java -jar -Dspring.profiles.active=<profile> build/libs/spring-music.jar
~~~

where `<profile>` is one of the following values:

* `mysql`
* `postgres`
* `mongodb`
* `redis`

If no profile is provided, an in-memory relational database will be used. If any other profile is provided, the appropriate database server must be started separately. Spring Boot will auto-configure a connection to the database using it's auto-configuration defaults. The connection parameters can be configured by setting the appropriate [Spring Boot properties](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html).

If more than one of these profiles is provided, the application will throw an exception and fail to start.

### Targetting Kubernetes

If you are targetting Kubernetes instead of Cloud Foundry, then you can use these profiles instead:-

* `mysql-k8s`
* `postgres-k8s`

These will use Kubernetes config map and secret variables instead of hardcoded values or values
from the CF environment variables.

## Running the application on Kubernetes

A Helm chart is provided that sets up postgres and links it to the spring music app, with the generated security secret to access the database.

```sh
cd helm
helm install springmusic ./spring-music-with-postgres --create-namespace -n sm
kubectl get pods -n sm
```

By default this uses a LoadBalancer. You can change this configuration though - see service.yaml in the Helm chart for details.

To list the ingress load balancer URL/IP for access, execute:-

```sh
kubectl get service -n sm spring-music-loadbalancer -o wide
```

LIMITATIONS:-

- Assumes you have a loadbalancer auto config (E.g. on AWS EKS) - otherwise append `--set lb.enabled=false` in the `helm install` command 
- Must use the `sm` namespace currently
- Must manually delete the pvc for postgres in order for a new deployment to have the correct postgres password set `kubectl get pvc -n sm; kubectl delete pvc -n sm NAME`
- Using admin postgres user instead of non-admin postgres user

## (Optional) Secure ingress with istio

If you have istio installed and injecting configuration into your Spring Music namespace, you can do the following to enable TLS ingress and mTLS between pods.

```sh
./gen-cert.sh                                                                                                                                                               ✔  took 6s   at 16:36:43 
Generating a 4096 bit RSA private key
............++
........................................++
writing new private key to 'adamfowler.co.uk.key'
-----
Generating a 4096 bit RSA private key
.............................................................++
...................................................................................................................................................................................++
writing new private key to 'sm.adamfowler.co.uk.key'
-----
Signature ok
subject=/CN=sm.adamfowler.co.uk/O=springmusic organization
Getting CA Private Key

kubectl create -n istio-system secret tls sm-tls-credential --key=sm.adamfowler.co.uk.key --cert=sm.adamfowler.co.uk.crt                          ✔  took 5s   at education-eks-9AHQJ9dw ⎈  at 16:42:20 
secret/sm-tls-credential created

kubectl apply -f spring-music-istio.yml                                                                                                           ✔  took 4s   at education-eks-9AHQJ9dw ⎈  at 16:44:30 
gateway.networking.istio.io/sm-gateway configured
virtualservice.networking.istio.io/spring-music-istio-svc unchanged
destinationrule.networking.istio.io/spring-music-dest unchanged

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.10/samples/addons/prometheus.yaml                                       ✔  took 10s   at education-eks-9AHQJ9dw ⎈  at 16:44:55 
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus unchanged
clusterrolebinding.rbac.authorization.k8s.io/prometheus unchanged
service/prometheus created
deployment.apps/prometheus created

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.10/samples/addons/kiali.yaml
customresourcedefinition.apiextensions.k8s.io/monitoringdashboards.monitoring.kiali.io unchanged
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer unchanged
clusterrole.rbac.authorization.k8s.io/kiali unchanged
clusterrolebinding.rbac.authorization.k8s.io/kiali unchanged
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
monitoringdashboard.monitoring.kiali.io/envoy created
monitoringdashboard.monitoring.kiali.io/go created
monitoringdashboard.monitoring.kiali.io/kiali created
monitoringdashboard.monitoring.kiali.io/micrometer-1.0.6-jvm-pool created
monitoringdashboard.monitoring.kiali.io/micrometer-1.0.6-jvm created
monitoringdashboard.monitoring.kiali.io/micrometer-1.1-jvm created
monitoringdashboard.monitoring.kiali.io/microprofile-1.1 created
monitoringdashboard.monitoring.kiali.io/microprofile-x.y created
monitoringdashboard.monitoring.kiali.io/nodejs created
monitoringdashboard.monitoring.kiali.io/quarkus created
monitoringdashboard.monitoring.kiali.io/springboot-jvm-pool created
monitoringdashboard.monitoring.kiali.io/springboot-jvm created
monitoringdashboard.monitoring.kiali.io/springboot-tomcat created
monitoringdashboard.monitoring.kiali.io/thorntail created
monitoringdashboard.monitoring.kiali.io/tomcat created
monitoringdashboard.monitoring.kiali.io/vertx-client created
monitoringdashboard.monitoring.kiali.io/vertx-eventbus created
monitoringdashboard.monitoring.kiali.io/vertx-jvm created
monitoringdashboard.monitoring.kiali.io/vertx-pool created
monitoringdashboard.monitoring.kiali.io/vertx-server created

./istioctl dashboard kiali
```

## Running the application on Cloud Foundry

When running on Cloud Foundry, the application will detect the type of database service bound to the application (if any). If a service of one of the supported types (MySQL, Postgres, Oracle, MongoDB, or Redis) is bound to the app, the appropriate Spring profile will be configured to use the database service. The connection strings and credentials needed to use the service will be extracted from the Cloud Foundry environment.

If no bound services are found containing any of these values in the name, an in-memory relational database will be used.

If more than one service containing any of these values is bound to the application, the application will throw an exception and fail to start.

After installing the 'cf' [command-line interface for Cloud Foundry](http://docs.cloudfoundry.org/cf-cli/), targeting a Cloud Foundry instance, and logging in, the application can be built and pushed using these commands:

~~~
$ cf push
~~~

The application will be pushed using settings in the provided `manifest.yml` file. The output from the command will show the URL that has been assigned to the application.

### Creating and binding services

Using the provided manifest, the application will be created without an external database (in the `in-memory` profile). You can create and bind database services to the application using the information below.

#### System-managed services

Depending on the Cloud Foundry service provider, persistence services might be offered and managed by the platform. These steps can be used to create and bind a service that is managed by the platform:

~~~
# view the services available
$ cf marketplace
# create a service instance
$ cf create-service <service> <service plan> <service name>
# bind the service instance to the application
$ cf bind-service <app name> <service name>
# restart the application so the new service is detected
$ cf restart
~~~

#### User-provided services

Cloud Foundry also allows service connection information and credentials to be provided by a user. In order for the application to detect and connect to a user-provided service, a single `uri` field should be given in the credentials using the form `<dbtype>://<username>:<password>@<hostname>:<port>/<databasename>`.

These steps use examples for username, password, host name, and database name that should be replaced with real values.

~~~
# create a user-provided Oracle database service instance
$ cf create-user-provided-service oracle-db -p '{"uri":"oracle://root:secret@dbserver.example.com:1521/mydatabase"}'
# create a user-provided MySQL database service instance
$ cf create-user-provided-service mysql-db -p '{"uri":"mysql://root:secret@dbserver.example.com:3306/mydatabase"}'
# bind a service instance to the application
$ cf bind-service <app name> <service name>
# restart the application so the new service is detected
$ cf restart
~~~

#### Changing bound services

To test the application with different services, you can simply stop the app, unbind a service, bind a different database service, and start the app:

~~~
$ cf unbind-service <app name> <service name>
$ cf bind-service <app name> <service name>
$ cf restart
~~~

#### Database drivers

Database drivers for MySQL, Postgres, Microsoft SQL Server, MongoDB, and Redis are included in the project.

To connect to an Oracle database, you will need to download the appropriate driver (e.g. from http://www.oracle.com/technetwork/database/features/jdbc/index-091264.html). Then make a `libs` directory in the `spring-music` project, and move the driver, `ojdbc7.jar` or `ojdbc8.jar`, into the `libs` directory.
In `build.gradle`, uncomment the line `compile files('libs/ojdbc8.jar')` or `compile files('libs/ojdbc7.jar')` and run `./gradle assemble`

#### Further K8S Reading

https://spring.io/projects/spring-cloud-kubernetes

https://hackmd.io/@ryanjbaxter/spring-on-k8s-workshop

