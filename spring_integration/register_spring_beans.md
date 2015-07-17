# Register Spring Beans

Starting from 2.0, Spring provides an option to detect beans by scanning
the classpath. Developers can use annotations (e.g. `@Component`) to
register bean definitions in the Spring container and this removes the
use of XML. We can use `@Component` which is a generic stereotype
annotation or those specialized stereotype annotation: `@Controller`,
`@Service`, or `@Repository` for presentation, service, persistence
layer, respectively. These annotations work equally for registering
beans but using specialized annotation makes your classes suited for
processing by tools.

When you register a bean, its bean scope is a "singleton" by default if
you don't specify it. Our service class is stateless so that it is
suitable to be a singleton-scoped bean. For those beans used in
composers, they should use scoped-proxy to ensure every time Spring will
retrieve them when a composer uses them. (Please use scoped-proxy even
for a singleton scoped bean, because scope of Spring beans doesn't match
scope of composers. Scoped-proxy can ensure composers get the latest
bean under their context. For furthermore explanation, please refer to [
Developer's Reference/Integration/Middleware
Layer/Spring](ZK_Developer%27s_Reference/Integration/Middleware_Layer/Spring "wikilink")
)

```java
@Service("authService")
@Scope(value="singleton",proxyMode=ScopedProxyMode.TARGET_CLASS)
public class AuthenticationServiceImpl implements AuthenticationService,Serializable{
...
}
```

-   Line 1: You could specify bean's name in `@Service` or its bean is
    derived from class name with first character in lower case (e.g.
    `authenticationServiceImpl` in this case).
-   Line 2: If you want to specify a bean's scope, use `@Scope`. For
    those beans used in composers, you should use scoped-proxy to ensure
    every time you get the latest bean.
