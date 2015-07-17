# Configuration

## Maven

When using a database, JPA, and integration of JPA and Spring, we should
add following dependencies based on chapter 9's configuration: (We add
`spring-web` and `cblib` for Spring framework which is explained in
previous chapter.)

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
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>2.2.2</version>
        </dependency>
        <!-- JPA(Hibernate) and HSQL dependencies -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>4.0.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hsqldb</groupId>
            <artifactId>hsqldb</artifactId>
            <version>2.2.6</version>
        </dependency>
...
```

-   Line 5, 16\~17: Spring provides a module to integrate several ORM
    (Object Relation Mapping) frameworks, integrating JPA requires this
    dependency.
-   Line 27\~28: There are various implementations of JPA specification,
    we choose Hibernate's one which is the most popular.
-   Line 32\~33: For using HSQL, we should add its JDBC driver.


## Persistence Unit Configuration

We should describe persistence unit configuration in an XML file called
`persistence.xml` and we need to specify `name`, `transaction-type`, and
`properties`. The `properties` are used by persistence provider
(Hibernate) to establish database connection and setup vendor specific
configurations.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <persistence-unit name="myapp" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.ejb.HibernatePersistence</provider>
        <properties>
            <property name="hibernate.dialect"
                        value="org.hibernate.dialect.HSQLDialect" />
            <property name="hibernate.connection.driver_class"
                        value="org.hsqldb.jdbcDriver" />
            <property name="hibernate.connection.username" value="sa" />
            <property name="hibernate.connection.password" value="" />
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.connection.url"
                        value="jdbc:hsqldb:file:data/store" />
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
</persistence>
```

-   Line 3: The persistence unit name `myapp` will be used later in
    Spring configuration.


## Deployment Descriptor

You aren't required to add any special configuration for JPA to work.
Here we add a Spring provided `OpernEntityMangerInViewFilter` to resolve
an issue caused by lazy-fetching in one-to-many mapping. Please refer to
[ Developer's
Reference](http://books.zkoss.org/wiki/ZK Developer%27s Reference/Integration/Persistence Layer/JPA "wikilink")
for this issue in more detail.

**Extracted from web.xml**

```xml
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

    <filter>
        <filter-name>OpenEntityManagerInViewFilter
        </filter-name>
        <filter-class>
        org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>OpenEntityManagerInViewFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
...
```

## Entity Annotation


Before storing objects into a database, we should specify OR
(object-relation) mapping for Java classes with meta data. After JDK
5.0, we can specify OR mapping as annotations instead of XML files. JPA
supports *configuration by exception* which means that it defines
default for most cases of application and users only need to override
the configuration value when it is exception to the default, not
necessary.

First, annotate the class with `@Entity` to turn it into an entity, and
annotate the member field for primary key with `@Id`. All other
annotations are optional and we use them to override default values.

**Todo class used in todo-list management**

```java
@Entity
@Table(name="apptodo")
public class Todo implements Serializable, Cloneable {


    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    Integer id;

    boolean complete;

    @Column(nullable=false,length=128)
    String subject;

    @Column(nullable=false,length=128)
    @Enumerated(EnumType.ORDINAL)
    Priority priority;

    @Temporal(TemporalType.TIMESTAMP)
    Date date;

    String description;

    //omit getter, setter, hashCode(), equals() and other methods

}
```

## Spring Beans Configuration


Spring JPA , available under the `org.springframework.orm.jpa` package,
offers integration support for Java Persistence API. According to Spring
framework reference, there are three options for JPA setup. We choose
the simplest one.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">

    <context:component-scan base-package="org.zkoss.essentials" />

    <!-- jpa(hibernate) configuration -->
    <bean id="entityManagerFactory"
        class="org.springframework.orm.jpa.LocalEntityManagerFactoryBean">
        <property name="persistenceUnitName" value="myapp"/>
    </bean>

    <bean id="transactionManager"
        class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory" />
    </bean>

    <tx:annotation-driven />
</beans>
```

-   Line 16,17: The simplest Spring setup for JPA is to add a
    `LocalEntityManagerFactoryBean` which create a
    `EntityManagerFactory` for simple deployment environments and
    specify its `persistenceUnitName` property.
-   Line 18: Set property `persistenceUnitName` with the name we
    specified in `persistence.xml`.
-   Line 21,26: Enable Spring's declarative transaction management.
