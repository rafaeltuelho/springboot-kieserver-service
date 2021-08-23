Business Application Service
=============================

This project was generated using https://start.jbpm.org/

It is a Spring Boot runtime based Kie Server used to execute Business Rules and Decision Models packaged as KJARs.

Alternatively use the following command to bootstrap a Business Application using the [KIE Service Spring Boot Archetype](https://github.com/kiegroup/droolsjbpm-knowledge/tree/master/kie-archetypes/kie-service-spring-boot-archetype):

```
mvn archetype:generate \
   -DarchetypeGroupId=org.kie \
   -DarchetypeArtifactId=kie-service-spring-boot-archetype \
   -DarchetypeVersion=7.52.0.Final \
   -DappType=brm
```

> NOTE: remember to update the `application.properties` and configure the `kie-maven-plugin` to properly add your decision/rules `kjar` artifact as a dependency.

## Self contained Immutable Fatjar

Starting with version `7.44.0.Final` you can use the `kie-maven-plugin` to package your kjar in the spring boot uberjar. Add these props in your Spring Boot `application.properties`:
```
kieserver.classPathContainer=true
kieserver.autoScanDeployments=true
```

and configure your Spring Boot app `kie-maven-plugin` in your pom.xml like this:
```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.kie</groupId>
        <artifactId>kie-maven-plugin</artifactId>
        <version>${version.org.kie}</version>
        <executions>
          <execution>
            <id>copy</id>
            <phase>prepare-package</phase>
            <goals>
              <goal>package-dependencies-kjar</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <artifactItems>
            <artifactItem>
              <groupId>com.myspace</groupId>
              <artifactId>your kjar artifcat id</artifactId>
              <version>1.0.0-SNAPSHOT</version>
            </artifactItem>
          </artifactItems>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
```

> With the above configuration you don't need to provide the Kie Server state file (`.xml`) in order to get your kie container deployed as the kie-maven-plugin will take care of the deployment.

## Enable Swagger API Docs

> If want to enable swagger Api Docs on your Spring Boot based application add these deps in your `pom.xml`

```xml
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-rt-rs-service-description-swagger</artifactId>
      <version>3.2.6</version>
    </dependency>
    <dependency>
      <groupId>io.swagger</groupId>
      <artifactId>swagger-jaxrs</artifactId>
      <version>1.5.15</version>
      <exclusions>
        <exclusion>
          <groupId>javax.ws.rs</groupId>
          <artifactId>jsr311-api</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>org.webjars</groupId>
      <artifactId>swagger-ui</artifactId>
      <version>3.43.0</version>
    </dependency>
```

and add this property on your `application.properties`
```
kieserver.swagger.enabled=true
```

For more details on how customize and configure the Kie Server on Spring Boot check the official Docs at https://access.redhat.com/documentation/en-us/red_hat_process_automation_manager/7.10/html-single/integrating_red_hat_process_automation_manager_with_other_products_and_components/index#assembly-springboot-business-apps

## Build the Spring Boot app

```
mvn clean install
```

## Run the service locally

```
java -jar target/springboot-kieserver-service-1.0-SNAPSHOT.jar
```

## Access the Kie Server (Swagger) API Docs

http://localhost:8090/rest/api-docs?url=http://localhost:8090/rest/swagger.json

you can authenticate with `kieserver/kieserver1!`


## Docker Image build
A Docker Image can be built using the [jKube **Kubernates** Maven Plugin](https://www.eclipse.org/jkube/docs/kubernetes-maven-plugin)

Assuming you have Docker Engine up and running on your local environment, execute the following command to generate the container image for this app.
```
mvn clean install -Pdocker
```

expect an output similar to the following:
```
...
[INFO] k8s: Building Docker image in Kubernetes mode
[INFO] k8s: [apps/springboot-kieserver-service:1.0-SNAPSHOT]: Created docker-build.tar in 1 second 
[INFO] k8s: [apps/springboot-kieserver-service:1.0-SNAPSHOT]: Built image sha256:01593
[INFO] k8s: [apps/springboot-kieserver-service:1.0-SNAPSHOT]: Removed old image sha256:31b49
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  17.043 s
[INFO] Finished at: 2021-03-04T13:35:30-05:00
[INFO] ------------------------------------------------------------------------
```

> The image generation configuration can be tuned via the `kubernates-maven-plugin` in the `pom.xml`

## Openshift build/deployment
The application can be directly deployed into an existing Openshift cluster using the [jKube **Openshift** Maven Plugin](https://www.eclipse.org/jkube/docs/openshift-maven-plugin)

Assuming you are logged into an Openshift Cluster (`oc login...`), execute the following command to start the deployment process of your app into the current namespace (openshift project).
```
mvn clean install -Popenshift
```

expect an output similar to the following:
```
[INFO] oc: Build springboot-kieserver-service-s2i-1 in status Complete
[INFO] oc: Found tag on ImageStream springboot-kieserver-service tag: sha256:e98fa1316b236b0c70c6d97c0e5e22544e982327ecabfc9902d0fcaaf0ff20a5
[INFO] oc: ImageStream springboot-kieserver-service written to /Users/rsoares/dev/github/rafaeltuelho/my-business-automation-showcase/springboot-kieserver-service/target/springboot-kieserver-service-is.yml
[INFO] 
[INFO] --- openshift-maven-plugin:1.1.1:apply (default-cli) @ springboot-kieserver-service ---
[INFO] oc: Using OpenShift at https://api.cluster.com:6443/ in namespace dmlab with manifest /Users/rsoares/dev/github/rafaeltuelho/my-business-automation-showcase/springboot-kieserver-service/target/classes/META-INF/jkube/openshift.yml 
[INFO] oc: OpenShift platform detected
[INFO] oc: Creating a Service from openshift.yml namespace dmlab name springboot-kieserver-service
[INFO] oc: Created Service: target/jkube/applyJson/dmlab/service-springboot-kieserver-service.json
[INFO] oc: Creating a DeploymentConfig from openshift.yml namespace dmlab name springboot-kieserver-service
[INFO] oc: Created DeploymentConfig: target/jkube/applyJson/dmlab/deploymentconfig-springboot-kieserver-service.json
[INFO] oc: HINT: Use the command `oc get pods -w` to watch your pods start up
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  03:07 min
[INFO] Finished at: 2021-03-04T19:51:52-05:00
[INFO] ------------------------------------------------------------------------
```

The above command will generate the openshif resources and trigger a Binary Build to build the app image inside the current namespace. After a while you can execute:

```
mvn oc:resource oc:apply -Popenshift
```

to create the DeploymentConfig, Service and Route resources.

## Start your Business Application Service

```
java -jar target/springboot-kieserver-service-1.0-SNAPSHOT.jar
```