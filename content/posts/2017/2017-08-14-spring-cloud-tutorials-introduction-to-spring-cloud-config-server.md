---
title: Spring Cloud Tutorials – Introduction to Spring Cloud Config Server
author: Siva
type: post
date: 2017-08-14T03:35:07+00:00
url: /2017/08/spring-cloud-tutorials-introduction-to-spring-cloud-config-server/
categories:
  - Java
  - Spring
tags:
  - Spring
  - SpringBoot
  - SpringCloud
popular: true
---
# Problem

SpringBoot provides lot of flexibility in externalizing configuration properties via properties or YAML files. We can also configure properties for each environment (dev, qa, prod etc) separately using profile specific configuration files such as **application.properties**, **application-dev.properties**, **application-prod.properties** etc. But once the application is started we can not update the properties at runtime. If we change the properties we need to restart the application to use the updated configuration properties.

Also, in the context of large number of MicroService based applications, we want the ability to configure and manage the configuration properties of all MicroServices from a centralized place.

# Solution

We can use **Spring Cloud Config Server** ([http://cloud.spring.io/spring-cloud-static/Dalston.SR2/#\_spring\_cloud_config][1]) to centralize all the applications configuration and use **Spring Cloud Config Client** module from the applications to consume configuration properties from Config Server. We can also update the configuration properties at runtime without requiring to restart the application.

> Many of the Spring Cloud modules can be used in SpringBoot applications even though you are not going to deploy your application in any Cloud platforms such as AWS, Pivotal CloudFoundry etc.

## Spring Cloud Config Server

Spring Cloud Config Server is nothing but a SpringBoot application with a configured configuration properties source. The configuration source can be a **git** repository, **svn** repository or Consul service (<https://www.consul.io/>).

<img class="aligncenter size-full" src="/images/config.png" sizes="(max-width: 648px) 100vw, 648px" data-recalc-dims="1" />

In this post we are going to use a git repository as configuration properties source.

### Git Config Repository

Create a git repository to store properties files. I have created a repository **config-repo** in GitHub ie <https://github.com/sivaprasadreddy/config-repo.git>.

Suppose we are going to develop two SpringBoot applications **catalog-service** and **order-service**. 
Let us create configuration files **catalogservice.properties** and **orderservice.properties** for **catalog-service** and **order-service** respectively.

**config-repo/catalogservice.properties**

{{< highlight properties >}}
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/catalog
spring.datasource.username=root
spring.datasource.password=admin
{{< / highlight >}}

**config-repo/orderservice.properties**

{{< highlight properties >}}
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
{{< / highlight >}}

We can also create profile specific configuration files such as **catalogservice-dev.properties**, **catalogservice-prod.properties**, **orderservice-dev.properties**, **orderservice-prod.properties**.

**config-repo/catalogservice-prod.properties**

{{< highlight properties >}}
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://appsrv1:3306/catalog
spring.datasource.username=appuser46
spring.datasource.password=T(iV&#)X84@1!
{{< / highlight >}}

**config-repo/orderservice-prod.properties**

{{< highlight properties >}}
spring.rabbitmq.host=srv245.ind.com
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin23
spring.rabbitmq.password=uY7&%we@1!
{{< / highlight >}}

Now commit all the configuration properties files in **config-repo** git repository.

#### Spring Cloud Config Server Application

Let us create a SpringBoot application **spring-cloud-config-server** from <http://start.spring.io> or from your favorite IDE by selecting the starters **Config Server** and **Actuator**.

This will generate the maven project with following **pom.xml**.

{{< highlight xml >}}

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
  http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.sivalabs</groupId>
	<artifactId>spring-cloud-config-server</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<packaging>jar</packaging>
	<name>spring-cloud-config-server</name>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.6.RELEASE</version>
		<relativePath/>
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Dalston.SR2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
{{< / highlight >}}

To make our SpringBoot application as a SpringCloud Config Server, we just need to add **@EnableConfigServer** annotation to the main entry point class and configure **spring.cloud.config.server.git.uri** property pointing to the git repository.

{{< highlight java >}}
package com.sivalabs.configserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@EnableConfigServer
@SpringBootApplication
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}
{{< / highlight >}}

**spring-cloud-config-server/src/main/resources/application.properties**

{{< highlight properties >}}
server.port=8888
spring.cloud.config.server.git.uri=https://github.com/sivaprasadreddy/config-repo.git
management.security.enabled=false
{{< / highlight >}}

In addition to configuring git repo uri, we configured **server.port** to 8888 and **disabled actuator security**. Now you can start the application which will start on port 8888.

Spring Cloud Config Server exposes the following REST endpoints to get application specific configuration properties:

{{< highlight properties >}}
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
{{< / highlight >}}

Here **{application}** refers to value of **spring.config.name** property, **{profile}** is an active profile and **{label}** is an optional git label (defaults to &#8220;master&#8221;).

Now if you access the URL http://localhost:8888/catalogservice/default then you will get the following response with catalogservice **default** configuration details:

{{< highlight json >}}
{
    "name": "catalogservice",
    "profiles": [
        "default"
    ],
    "label": null,
    "version": "8a06f25aeb3f28a8f06b5634eae01858b2c6465d",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/sivaprasadreddy/config-repo.git/catalogservice.properties",
            "source": {
                "spring.datasource.username": "root",
                "spring.datasource.driver-class-name": "com.mysql.jdbc.Driver",
                "spring.datasource.password": "admin",
                "spring.datasource.url": "jdbc:mysql://localhost:3306/catalog"
            }
        }
    ]
}
{{< / highlight >}}

If you access the URL http://localhost:8888/catalogservice/prod then you will get the following response with catalogservice **prod** configuration details.

{{< highlight json >}}
{
    "name": "catalogservice",
    "profiles": [
        "prod"
    ],
    "label": null,
    "version": "8a06f25aeb3f28a8f06b5634eae01858b2c6465d",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/sivaprasadreddy/config-repo.git/catalogservice-prod.properties",
            "source": {
                "spring.datasource.username": "appuser46",
                "spring.datasource.driver-class-name": "com.mysql.jdbc.Driver",
                "spring.datasource.password": "T(iV&#)X84@1!",
                "spring.datasource.url": "jdbc:mysql://appsrv1:3306/catalog"
            }
        },
        {
            "name": "https://github.com/sivaprasadreddy/config-repo.git/catalogservice.properties",
            "source": {
                "spring.datasource.username": "root",
                "spring.datasource.driver-class-name": "com.mysql.jdbc.Driver",
                "spring.datasource.password": "admin",
                "spring.datasource.url": "jdbc:mysql://localhost:3306/catalog"
            }
        }
    ]
}
{{< / highlight >}}

In addition to the application specific configuration files like **catalogservice.properties**, **orderservice.properties**, you can create **application.properties** file to contain common configuration properties for all applications. As you might have guessed you can have profile specific files like **application-dev.properties, application-prod.properties**.

Suppose you have **application.properties** file in **config-repo** with the following properties:

{{< highlight properties >}}
message=helloworld
jdbc.datasource.url=jdbc:mysql://localhost:3306/defapp
{{< / highlight >}}

Now if you access http://localhost:8888/catalogservice/prod then you will get the following response:

{{< highlight json >}}
{
    "name": "catalogservice",
    "profiles": [
        "prod"
    ],
    "label": null,
    "version": "8a06f25aeb3f28a8f06b5634eae01858b2c6465d",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/sivaprasadreddy/config-repo.git/catalogservice-prod.properties",
            "source": {
              "spring.datasource.username": "appuser46",
              "spring.datasource.driver-class-name": "com.mysql.jdbc.Driver",
              "spring.datasource.password": "T(iV&#)X84@1!",
              "spring.datasource.url": "jdbc:mysql://appsrv1:3306/catalog"
            }
        },
        {
            "name": "https://github.com/sivaprasadreddy/config-repo.git/catalogservice.properties",
            "source": {
                "spring.datasource.username": "root",
                "spring.datasource.driver-class-name": "com.mysql.jdbc.Driver",
                "spring.datasource.password": "admin",
                "spring.datasource.url": "jdbc:mysql://localhost:3306/catalog"
            }
        },
        {
            "name": "https://github.com/sivaprasadreddy/config-repo.git/application.properties",
            "source": {
                "message": "helloworld",
                "jdbc.datasource.url": "jdbc:mysql://localhost:3306/defapp"
            }
        }
    ]
}
{{< / highlight >}}

Similarly you can access http://localhost:8888/orderservice/default to get the orderservice configuration details.

{{< highlight json >}}
{
    "name": "orderservice",
    "profiles": [
        "default"
    ],
    "label": null,
    "version": "8a06f25aeb3f28a8f06b5634eae01858b2c6465d",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/sivaprasadreddy/config-repo.git/orderservice.properties",
            "source": {
              "spring.rabbitmq.host": "localhost"
              "spring.rabbitmq.port": "5672"
              "spring.rabbitmq.username": "guest"
              "spring.rabbitmq.password": "guest"
            }
        },
        {
            "name": "https://github.com/sivaprasadreddy/config-repo.git/application.properties",
            "source": {
                "message": "helloworld",
                "jdbc.datasource.url": "jdbc:mysql://localhost:3306/defapp"
            }
        }
    ]
}
{{< / highlight >}}

Now that we have seen how to create configuration server using Spring Cloud Config Server and how to fetch the application specific configuration properties using REST API.

Let us see how we can create a SpringBoot application and use configuration properties from Config Server instead of putting them inside the application.

#### Spring Cloud Config Client (catalog-service)

Create a SpringBoot application **catalog-service** with **Config Client,** **Web** and **Actuator** starters.

{{< highlight xml >}}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
  http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.sivalabs</groupId>
	<artifactId>catalog-service</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>spring-cloud-config-client</name>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.6.RELEASE</version>
		<relativePath/>
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Dalston.SR2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
{{< / highlight >}}

Usually in SpringBoot application we configure properties in **_application.properties_**. But while using Spring Cloud Config Server we use **_bootstrap.properties_** or **_bootstrap.yml_** file to configure the URL of Config Server and Spring Cloud Config Client module will take care of starting the application by fetching the application properties from Config Server.

Configure the following properties in **src/main/resources/bootstrap.properties**:

{{< highlight properties >}}
server.port=8181
spring.application.name=catalogservice
spring.cloud.config.uri=http://localhost:8888
management.security.enabled=false
{{< / highlight >}}

We have configured the url of configuration server using **spring.cloud.config.uri** property. Also we have specified the application name using **spring.application.name** property.

> Note that the value of **spring.application.name** property should match with base filename (catalogservice) in config-repo.

Now run the following catalog-service main entry point class:

{{< highlight java >}}
package com.sivalabs.catalogservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class CatalogServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(CatalogServiceApplication.class, args);
	}
}
{{< / highlight >}}

We can access the actuator endpoint http://localhost:8181/env to see all the configuration properties.

{{< highlight json >}}
{
    "profiles": [],
    "server.ports": {
        "local.server.port": 8080
    },
    "configService:configClient": {
        "config.client.version": "8a06f25aeb3f28a8f06b5634eae01858b2c6465d"
    },
    "configService:https://github.com/sivaprasadreddy/config-repo.git/catalogservice.properties": {
        "spring.datasource.username": "root",
        "spring.datasource.driver-class-name": "com.mysql.jdbc.Driver",
        "spring.datasource.password": "******",
        "spring.datasource.url": "jdbc:mysql://localhost:3306/catalog"
    },
    "configService:https://github.com/sivaprasadreddy/config-repo.git/application.properties": {
        "message": "helloworld",
        "jdbc.datasource.url": "jdbc:mysql://localhost:3306/defapp"
    },
    "servletContextInitParams": {},
    "systemProperties": {
        ...
        ...
    },
    "systemEnvironment": {
        ...
        ...
    },
    "springCloudClientHostInfo": {
        "spring.cloud.client.hostname": "192.168.0.101",
        "spring.cloud.client.ipAddress": "192.168.0.101"
    },
    "applicationConfig: [classpath:/bootstrap.properties]": {
        "management.security.enabled": "false",
        "spring.cloud.config.uri": "http://localhost:8888",
        "spring.application.name": "catalogservice"
    },
    "defaultProperties": {}
}
{{< / highlight >}}

You can see that catalog-service application fetches the catalogservice properties from Config Server during bootstrap time. You can bind these properties using **@Value** or **@EnableConfigurationProperties** just the way you bind if they are defined within the application itself.

#### Precedence of properties

Now that we know there are many ways to provide configuration properties in many files such as **application.properties, bootstrap.properties** and their profile variants inside application **src/main/resources** and **{application-name}-{profile}.properties, application-{profile}.properties** in config-repo.

The following diagrams shows the precedence of configuration properties from various properties locations.

<img class="aligncenter size-full" src="/images/precedence.png" sizes="(max-width: 648px) 100vw, 648px" data-recalc-dims="1" />

#### Refresh properties at runtime

Let us see how we can update the configuration properties of catalog-service at runtime without requiring to restart the application.

Update the **catalogservice.properties** in config-repo git repository and commit the changes. Now if you access http://localhost:8181/env you will still see the old properties.

In order to reload the configuration properties we need to do the following:

  * Mark Spring beans that you want to reload on config changes with @RefreshScope
  * Issue http://localhost:8181/refresh request using **POST** method

To test the reloading behaviour let&#8217;s add a property **name=Siva** in **config-repo/catalogservice.properties** and commit it.

Create a simple RestController to display **name** value as follows:


{{< highlight java >}}

@RestController
@RefreshScope
class HomeController
{
	@Value("${name}")
	String name;

	@GetMapping("/name")
	public String name()
	{
		return name;
	}
}
{{< / highlight >}}

Now access http://localhost:8181/name which which will display **Siva**. Now change the property value to **name=Prasad** in **config-repo/catalogservice.properties** and commit it.

In order to reload the config changes trigger http://localhost:8181/refresh request using **POST** method and again access http://localhost:8181/name which should display **Prasad**.

But issuing **/refresh** requests manually is tedious and impractical in case of large number of applications and multiple instances of same application. We will cover how to handle this problem using **Spring Cloud Bus** in next post
  
[Spring Cloud Tutorials – Auto Refresh Config Changes using Spring Cloud Bus]({{< relref "2017-08-14-spring-cloud-tutorials-auto-refresh-config-changes-using-spring-cloud-bus.md" >}}).

The source code for this article is at https://github.com/sivaprasadreddy/spring-cloud-tutorial

&nbsp;

 [1]: http://cloud.spring.io/spring-cloud-static/Dalston.SR2/#_spring_cloud_config
