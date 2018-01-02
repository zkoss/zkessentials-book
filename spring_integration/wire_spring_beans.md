# Wire Spring Beans
After registering beans for service classes, we can "wire" them in our
controllers with ZK's variable resolver. To wire a Spring bean in a
composer, we need to apply a
`org.zkoss.zkplus.spring.DelegatingVariableResolver`.
Then, we can apply annotation `@WireVariable` on a variable which we
want to wire a Spring bean with. ZK will then wire the corresponding
Spring bean with **the variable whose name is the same as the bean's
name**. Alternatively, you can specify the bean's name with
`@WireVariable("beanName")`.

You might think why don't we just register our controllers(or
ViewModels) as Spring beans, so that we can use Spring's `@Autowire`. We
don't recommend to do so. The main reason is that none of Spring bean's
scope matches ZK's composer's life cycle, for details please refer to [
Developer's
Reference](ZK_Developer%27s_Reference/Integration/Middleware_Layer/Spring "wikilink").

**Wire beans in a composer**

```java
@VariableResolver(org.zkoss.zkplus.spring.DelegatingVariableResolver.class)
public class SidebarChapter4Controller extends SelectorComposer<Component>{

    private static final long serialVersionUID = 1L;

    //wire components
    @Wire
    Grid fnList;

    //wire service
    @WireVariable("sidebarPageConfigPagebase")
    SidebarPageConfig pageConfig;

    ...
}
```

-   Line 11: Specify bean's name `sidebarPageConfigPagebase`

**Wire beans in a ViewModel**

```java
@VariableResolver(org.zkoss.zkplus.spring.DelegatingVariableResolver.class)
public class ProfileViewModel implements Serializable{
    private static final long serialVersionUID = 1L;

    //wire services
    @WireVariable
    AuthenticationService authService;
    @WireVariable
    UserInfoService userInfoService;
    ...
}
```

-   Line 6: Wire a Spring bean whose bean name is `authService`.


## Wire Manually
When using `@WireVariable` out of a composer (or a ViewModel), ZK will
not wire Spring beans for you automatically. If you need to get a Spring
bean, you can wire them manually. The example below wires a Spring bean
in a page initiator:

```java
@VariableResolver(org.zkoss.zkplus.spring.DelegatingVariableResolver.class)
public class AuthenticationInit implements Initiator {

    @WireVariable
    AuthenticationService authService;

    public void doInit(Page page, Map<String, Object> args) throws Exception {
        //wire service manually by calling Selectors API
        Selectors.wireVariables(page, this, Selectors.newVariableResolvers(getClass(), null));

        UserCredential cre = authService.getUserCredential();
        if(cre==null || cre.isAnonymous()){
            Executions.sendRedirect("/chapter8/login.zul");
            return;
        }
    }
}
```

-   Line 9: After applying `@VariableResolver` and `@WireVariable`, use
    <javadoc>org.zkoss.zk.ui.select.Selectors</javadoc> to wire Spring
    beans manually.

After completing above steps, integration of Spring is done. The
application's appearance doesn't change, but its infrastructure is now
managed by Spring. You can visit http://localhost:8080/zkessentials to
see the result.
