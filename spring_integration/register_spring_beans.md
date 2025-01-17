# Register Spring Beans

Starting from 2.0, Spring provides an option to detect beans by scanning
the classpath. You can use annotations (e.g. `@Component`) to
register bean definitions in a Spring container, and those remove the
use of XML. We can use `@Component` which is a generic stereotype
annotation or those specialized stereotype annotations: `@Controller`,
`@Service`, or `@Repository` for presentation, service, and persistence
layer, respectively. These annotations work equally for registering
beans but using specialized annotation makes your classes suited for
processing by tools.

When you register a bean, its bean scope is a "singleton" by default if
you don't specify it. Our service class is stateless, so it is
suitable to be a singleton-scoped bean.

```java
@Service("authService")
@Scope
public class AuthenticationServiceImpl implements AuthenticationService,Serializable{
...
}
```

-   Line 1: You could specify bean's name in `@Service` or its bean is
    derived from class name with the first character in lower case (e.g.
    `authenticationServiceImpl` in this case).
-   Line 2: If you want to specify a bean's scope, use `@Scope`. For
    those beans used in composers, you should use scoped-proxy to ensure
    every time you get the latest bean.
