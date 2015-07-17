# Secure Your Pages

Even if you have implemented an authentication mechanism, a user still can
access a page if he knows the page's URL. Therefore, we should protect a
page from illegal access by checking user's credentials in his session
when a page is requested by a user.


##Authentication Service

We can implement a service class that performs the authentication
operations.

**Get user credentials**

```java
public class AuthenticationServiceChapter3Impl implements AuthenticationService,Serializable{
    public UserCredential getUserCredential(){
        Session sess = Sessions.getCurrent();
        UserCredential cre = (UserCredential)sess.getAttribute("userCredential");
        if(cre==null){
            cre = new UserCredential();//new a anonymous user and set to session
            sess.setAttribute("userCredential",cre);
        }
        return cre;
    }
...
}
```

-   Line 3, 4: As we mentioned, we can get current session and check
    user's credentials to verify its authentication status.

**Log in/ log out**

```java
public class AuthenticationServiceChapter7Impl extends AuthenticationServiceChapter5Impl{


    UserInfoService userInfoService = new UserInfoServiceChapter3Impl();

    @Override
    public boolean login(String nm, String pd) {
        User user = userInfoService.findUser(nm);
        //a simple plan text password verification
        if(user==null || !user.getPassword().equals(pd)){
            return false;
        }

        Session sess = Sessions.getCurrent();
        UserCredential cre = new UserCredential(user.getAccount(),user.getFullName());
        ...
        sess.setAttribute("userCredential",cre);


        return true;
    }

    @Override
    public void logout() {
        Session sess = Sessions.getCurrent();
        sess.removeAttribute("userCredential");
    }
    ...
}
```

-   Line 14: Get the current session.
-   Line 17: Store user credentials into the session with a key
    `userCredential` which is used to retrieve credential back in the
    future.
-   Line 26: Remove the stored user credentials in the session.


## Page Initialization

ZK allows you to [ run some code before ZK Loader instantiates any component](http://books.zkoss.org/wiki/ZK%20Developer's%20Reference/UI%20Patterns/Page%20Initialization)
by implementing an `org.zkoss.zk.ui.util.Initiator`.
When we apply an initiator to a zul, ZK will invoke our initiator before creating components.

We can create an initiator to check existence of a user's credentials in
the session. If a user's credentials is absent, we determine it's an
illegal request and redirect it back to the login page.

**Page initiator to check a user's credentials**

```java
public class AuthenticationInit implements Initiator {

    //services
    AuthenticationService authService = new AuthenticationServiceChapter7Impl();

    public void doInit(Page page, Map<String, Object> args) throws Exception {

        UserCredential cre = authService.getUserCredential();
        if(cre==null || cre.isAnonymous()){
            Executions.sendRedirect("/chapter7/login.zul");
            return;
        }
    }
}
```

-   Line 1, 6: A page initiator class should implement
    `org.zkoss.zk.ui.util.Initiator` and override
    `doInit()`.
-   Line 8: Get a user's credentials from current session.
-   Line 10: Redirect users back to login page.

Then we can apply this page initiator to those pages we want to protect
from unauthenticated access.

**chapter7/index.zul**

```xml
<?link rel="stylesheet" type="text/css" href="/style.css"?>
<!-- protect page by the authentication init  -->
<?init class="org.zkoss.essentials.chapter7.AuthenticationInit"?>

```
-   Line 3: Because index.zul is the main page, we apply this page
    initiator on it.

After above steps are complete, if you directly visit
http://localhost:8080/essentials/chapter7/index.zul without authentication, you will be redirected to the login page.
