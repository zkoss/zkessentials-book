# JPA Integration
In previous chapters, we mimic a database with a static list as follows:

```java
public class TodoListServiceChapter6Impl implements TodoListService {

    static int todoId = 0;
    static List<Todo> todoList = new ArrayList<Todo>();
    static{
        todoList.add(new Todo(todoId++,"Buy some milk",Priority.LOW,null,null));
        todoList.add(new Todo(todoId++,"Dennis' birthday gift",Priority.MEDIUM,dayAfter(10),null));
        todoList.add(new Todo(todoId++,"Pay credit-card bill",Priority.HIGH,dayAfter(5),"$1,000"));
    }

    /** synchronized is just because we use static userList in this demo to prevent concurrent access **/
    public synchronized List<Todo>getTodoList() {
        List<Todo> list = new ArrayList<Todo>();
        for(Todo todo:todoList){
            list.add(Todo.clone(todo));
        }
        return list;
    }

...
}
```

-   Line 4, 12: Store objects in a static list and perform all data
    operation on it.

Originally we perform all persistence operations on the list, but we
will replace this part with a real database and a persistence framework,
*JPA*.

*Java Persistence API* (JPA) is a POJO-based persistence specification.
It offers an *object-relational mapping* solution to enterprise Java
applications. In this chapter, we don't create new applications but
re-write data persistence part based on chapter 9 with JPA. We will
create a simple database with HSQL and implement a persistence layer
using the DAO (Data Access Object) pattern to encapsulate all database
related operations. We also have to annotate all entity classes that
will be stored in the database with JPA annotations. To make the example
close to a real application, we keep using the Spring framework and
demonstrate how to integrate Spring with JPA.

# Source Code


As we mentioned in the Project Struecture, our source code has [ 3
branches](../project_structure.md)
in github. The source code of this chapter's example belongs to a
different branch in the github: **zk8-jpa**.

We don't create new examples in this chapter, but we add 2 DAO classes
written in JPA under the package `org.zkoss.essentials.services.impl`.

![ center](../images/ze-ch10-source.png)

# Configuration


## Maven


When using a database, JPA, and integration of JPA and Spring, we should
add following dependencies based on chapter 9's configuration: (We add
`spring-web` and `cblib` for Spring framework which is explained in
previous chapter.)

```xml

    <properties>
        <zk.version>6.5.1</zk.version>
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


# DAO Implementation

The *Data Access Object (DAO)* pattern is a good practice to implement a
persistence layer and it encapsulates data access codes from the
business tier. A DAO object exposes an interface to a business object
and performs persistence operation relating to a particular persistent
entity. Now, we can implement persistence operations like CRUD in a DAO
class with JPA and `EntityManager` injected by Spring.

```java
@Repository
public class TodoDao {

    @PersistenceContext
    private EntityManager em;

    @Transactional(readOnly=true)
    public List<Todo> queryAll() {
        Query query = em.createQuery("SELECT t FROM Todo t");
        List<Todo> result = query.getResultList();
        return result;
    }

    ...

    @Transactional
    public Todo save(Todo todo){
        em.persist(todo);
        return todo;
    }

    @Transactional
    public Todo update(Todo todo){
        todo = em.merge(todo);
        return todo;
    }

    @Transactional
    public void delete(Todo todo){
        Todo r = get(todo.getId());
        if(r!=null){
            em.remove(r);
        }
    }
}
```

-   Line 1: We register `TodoDao` as a Spring bean with `@Repository`
    because it is a DAO class according to Spring's suggestion.
-   Line 4: As Spring manages our entity manager factory, it can
    understand `@PersistenceContext` and inject a transaction scope
    `EntityManager` for us. Hence, we don't need to create
    `EntityManager` by our own.
-   Line 7: We have enabled Spring's declarative transaction management
    so that we can apply `@Transactional` on a methods.

After completing DAO classes, we can inject them to our service class
with Spring's `@Autowired` because they are all Spring beans.

```java
@Service("todoListService")
@Scope(value="singleton",proxyMode=ScopedProxyMode.TARGET_CLASS)
public class TodoListServiceImpl implements TodoListService {

    @Autowired
    TodoDao dao;

    public List<Todo>getTodoList() {
        return dao.queryAll();
    }

    ...
}
```

Completing the above steps, we have created a dependency relationship
among the controller, service, and persistence classes as follows:

![](../images/ze-ch10-dependencies.png)

Each of these classes encapsulates cohesive functions and has decoupled
relationships with others. You can easily expand the architecture by
adding more classes or create dependencies between two layers.

You can visit <http://localhost:8080/essentials> to see the result.


# Conclusion

Our book ends here. But the adventure toward ZK just begins. This book
opens you a door and introduces you some basic concepts and usages of
ZK. You can start to deploy ZK to your server according to [ZK
Installation Guide](http://books.zkoss.org/wiki/ZK Installation Guide "wikilink") which contains
information you will use in deploying to web servers. For further
development help, the [ ZK Developer's
Reference](http://books.zkoss.org/wiki/ZK_Developer%27s_Reference "wikilink") contains complete
references for various topics in developing ZK based applications (i.e.
for this chapter: [ Middleware
Layer](http://books.zkoss.org/wiki/ZK%20Developer's%20Reference/Integration/Middleware%20Layer "wikilink")/[
Spring](http://books.zkoss.org/wiki/ZK%20Developer's%20Reference/Integration/Middleware%20Layer/Spring "wikilink")
or [ Persistence
Layer](http://books.zkoss.org/wiki/ZK%20Developer's%20Reference/Integration/Persistence%20Layer "wikilink")/[
JPA](http://books.zkoss.org/wiki/ZK%20Developer's%20Reference/Integration/Persistence%20Layer/JPA "wikilink")).
[ ZUML Reference ](http://books.zkoss.org/wiki/ZUML Reference) describes syntax and EL
expression used in a zul. If you want to know details of a component,
please refer to the [ ZK Component
Reference](http://books.zkoss.org/wiki/ZK_Component_Reference "wikilink"). ZK also provides lots of
configuration, you can take a look at [ ZK Configuration
Reference](http://books.zkoss.org/wiki/ZK_Configuration_Reference "wikilink").

We hope you can make use of what you learn here to obtain an even
greater knowledge of ZK.

