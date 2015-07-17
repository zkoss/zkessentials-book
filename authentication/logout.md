# Logout

When a user logs out, we usually clear his credentials from the session
and redirect him to the login page. In our example, clicking "Logout"
label in the banner can log you out.

**chpater8/layout/banner.zul**

```xml
<div hflex="1" vflex="1" sclass="banner" >
    <hbox hflex="1" vflex="1" align="center">
        <!-- other components -->

        <hbox apply="org.zkoss.essentials.chapter7.LogoutController"
            hflex="1" vflex="1" pack="end" align="end" >
            <label value="${sessionScope.userCredential.name}"
                if="${not sessionScope.userCredential.anonymous}"/>
            <label id="logout" value="Logout"
            if="${not sessionScope.userCredential.anonymous}" sclass="logout"/>
        </hbox>
    </hbox>
</div>
```

-   Line 9, 10: We listen `onClick` event on logout label to perform
    logout action.

**LogoutController.java**

```java

public class LogoutController extends SelectorComposer<Component> {

    //services
    AuthenticationService authService = new AuthenticationServiceChapter7Impl();

    @Listen("onClick=#logout")
    public void doLogout(){
        authService.logout();
        Executions.sendRedirect("/chapter7/");
    }
}
```

-   Line 8: Call service class to perform logout.
-   Line 9: Redirect users to login page.

After completing the above steps, you can visit
`http://localhost:8080/essentials/chapter7` to see the result.
