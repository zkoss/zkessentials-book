# Login

It is a common way to request an account and a password for
authentication. We create a login page to collect user's account and
password, and the login page also uses a template zul to keep a
consistent style with the index page. However, there should be no sidebar because users without logging in should not be able to access main functions. So we need to change our template to create the sidebar according to user credential in the session with a shadow component, `<if>`.

**chapter7/layout/template.zul**

```xml
<zk>
	<borderlayout hflex="1" vflex="1" >
		<north height="100px" border="none" >
			<apply templateURI="/chapter7/layout/banner.zul"/>
		</north>
	<!-- create only when the currentUser is not an anonymous  -->
		<if test="${not sessionScope.userCredential.anonymous}">
		<west width="260px" border="none" collapsible="true" splittable="true" minsize="300">
			<apply templateURI="/chapter6/ajaxbased/sidebar.zul"/>
		</west>
		</if>
		<center id="mainContent" autoscroll="true" border="none">
			<!-- the main content will be insert to here -->
			<apply template="center"/>
		</center>
		<south height="50px" border="none">
			<apply templateURI="/chapter1/footer.zul"/>
		</south>
	</borderlayout>
</zk>
```

-   Line 7: Determine if there is an anonymous user credential in the session by EL.


The login form is built with *Grid*. This page should be accessible for
all users, so we don't have to apply `AuthenticationInit`.

**chapter7/login.zul**

```xml
<?link rel="stylesheet" type="text/css" href="/style.css"?>
<zk>
<div height="100%">
	<template name="layout" src="/chapter7/layout/template.zul"/>
	<apply template="layout"/>
	<template name="center">
		<hbox vflex="1" hflex="1" align="center"
			pack="center" spacing="20px">
			<vlayout>
				<window id="loginWin"
					apply="org.zkoss.essentials.chapter7.LoginController"
					title="Login with you name" border="normal" hflex="min">
					<vbox hflex="min" align="center">
						<grid hflex="min">
							<columns>
								<column hflex="min" align="right" />
								<column />
							</columns>
							<rows>
								<row>
									Account :
									<textbox id="account" width="200px" />
								</row>
								<row>
									Password :
									<textbox id="password" type="password"
										width="200px" />
								</row>
							</rows>
						</grid>
						<label id="message" sclass="warn" value="&#160;" />
						<button id="login" label="Login" />

					</vbox>
				</window>
				(use account='zkoss' and password='1234' to login)
			</vlayout>
		</hbox>
	</template>
</div>
</zk>
```

-   Line 6: Define a template named `cetner` which is used in `template.zul`.
-   Line 26: Specify "password" at `type`, then user input will be
    masked.

This login controller collects the account and password and validates
them with an authentication service class. If the password is correct,
the authentication service class saves user's credentials into the
session.

**Controller used in chapter7/login.zul**

```java

public class LoginController extends SelectorComposer<Component> {

    //wire components
    @Wire
    Textbox account;
    @Wire
    Textbox password;
    @Wire
    Label message;

    //services
    AuthenticationService authService = new AuthenticationServiceChapter7Impl();


    @Listen("onClick=#login; onOK=#loginWin")
    public void doLogin(){
        String nm = account.getValue();
        String pd = password.getValue();

        if(!authService.login(nm,pd)){
            message.setValue("account or password are not correct.");
            return;
        }
        UserCredential cre= authService.getUserCredential();
        message.setValue("Welcome, "+cre.getName());
        message.setSclass("");

        Executions.sendRedirect("/chapter7/");

    }
}
```

-   Line 20: Authenticate a user with account and password and save
    user's credential into the session if it passes.
-   Line 28: Redirect to index page after successfully authenticated.

After login, we want to display a user's account in the banner. We can
use EL to get a user's account from `UserCredential` in the session.

**chapter7/layout/banner.zul**

```xml
<div hflex="1" vflex="1" sclass="banner">
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

-   Line 7: The [sessionScope](http://books.zkoss.org/wiki/ZUML%20Reference/EL%20Expressions/Implicit%20Objects%20(Predefined%20Variables)/sessionScope)
    is an implicit object that you can use within EL to access session's
    attribute. It works as the same as `getAttribute()`. You can use it
    to get session's attribute with dot notation, e.g.
    `${sessionScope.userCredential}` equals to calling
    `getAttribute("userCredential")` of a `Session` object.
