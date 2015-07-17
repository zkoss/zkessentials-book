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


## Source Code

As we mentioned in the [Project Struecture](../project_structure.md), our source code has 3 branches in github. The source code of this chapter's example belongs to the branch: **zk8-jpa**.

We don't create new examples in this chapter, but we add 2 DAO classes
implemented in JPA under the package `org.zkoss.essentials.services.impl`.





