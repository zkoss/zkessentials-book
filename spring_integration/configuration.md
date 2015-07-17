# Configuration

## Maven

In order to integrate our ZK application with Spring, we must add
dependencies for Spring. The cglib is an optional dependency and we add
it because our application uses Spring's scoped-proxy that requires it.

**Extracted from pom.xml**

```xml
<properties>
    <zk.version>8.0.0-Eval</zk.version>
    <maven.build.timestamp.format>yyyy-MM-dd</maven.build.timestamp.format>
    <packname>-${project.version}-FL-${maven.build.timestamp}</packname>
    <spring.version>3.1.2.RELEASE</spring.version>
</properties>
...
    <!-- Spring 3 dependencies -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>2.2.2</version>
    </dependency>
```


## Deployment Descriptor

The deployment descriptor (web.xml) also needs two more listeners from
Spring.

**Extracted from web.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app version="2.4" xmlns="http://java.sun.com/xml/ns/j2ee"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

    <description><![CDATA[ZK Essentials]]></description>
    <display-name>ZK Essentials</display-name>

    <!-- ZK configuration-->
    ...

    <!-- Spring configuration -->
    <!-- Initialize spring context -->
    <listener>
        <listener-class>
        org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>
    <!-- Enable webapp Scopes-->
     <listener>
        <listener-class>
        org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>

    <welcome-file-list>
        <welcome-file>index.zul</welcome-file>
    </welcome-file-list>
</web-app>
```

-   Line 16,17: The `ContextLoaderListener` reads Spring configuration,
    and default location is `WEB-INF/applicationContext.xml`.
-   Line 21,22: Use `RequestContextListener` to support web-scoped beans
    (`request`, `session`, `global session`).

## Spring Configuration File


Create Spring configuration file with default name
(`applicationContext.xml`). We enable Spring's classpath scanning to
detect and register those class with annotation as beans automatically.

**WEB-INF/applicationContext.xml**

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd">

    <context:component-scan base-package="org.zkoss.essentials" />

</beans>
```

-   Line 10: This configuration enables classpath scanning. Spring will
    automatically detect those classes with Spring bean annotations and
    register them in bean definitions. You should
    specify a common parent package or a comma-separated package list that
    includes all candidate classes at `base-package` attribute.

