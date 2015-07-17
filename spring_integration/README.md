# Spring Integration

The Spring Framework is a popular application development framework for
enterprise Java. One key element is its infrastructural support: a
light-weighted IoC (Inversion of Control) container that manages POJOs
as Spring beans and their dependency relationship.

The most common integration way is to let Spring manage class
dependencies of an application. When a class "A" references a class "B"
and calls B's method, we say that A depends on B. In examples of
previous chapters, we create this dependency by instantiating B class in
A class as follows:

```java
public class ProfileViewController extends SelectorComposer<Component>{

    AuthenticationService authService = new AuthenticationServiceChapter8Impl();
    ...
}
```

-   `ProfileViewController` depends on `AuthenticationService`.

Spring can help us manage these dependencies without instantiating
dependent classes manually. In this chapter, we won't create new example
applications but will make previous examples integrate with Spring.


## Source Code

As we mentioned in the Introduction, our source code has [ 3
branches](../project_structure.md)
in github. The source code of this chapter's example belongs to the
branch: **zk8-spring**. You can select the branch and click
"zip" icon to download as a zip.

We don't create new examples in this chapter, but we re-organize some
classes. You can see from the image below. We move all service class
implementations to the package `org.zkoss.essentials.services.impl`.





