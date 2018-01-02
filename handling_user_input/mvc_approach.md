# MVC Approach
Under this approach, we implement all event handling and presentation
logic in a controller with no code present in the ZUL file. This
approach makes the responsibility of each role (Model, View, and
Controller) more cohesive and allows you to control components directly.
It is very intuitive and very flexible.


## Construct a Form Style Page
With the concept and technique we talked about in last chapter, it
should be easy to construct a form style user interface as follows. It
uses a two-column *Grid* to build the form style layout and different
input components to receive user's profile like name and birthday. The
zul file below is included in the *Center* of the *Border Layout*.

**chapter3/profile-mvc.zul**

```xml
<?link rel="stylesheet" type="text/css" href="/style.css"?>
<window apply="org.zkoss.essentials.chapter3.mvc.ProfileViewController"
    border="normal" hflex="1" vflex="1" contentStyle="overflow:auto">
    <caption src="/imgs/profile.png" sclass="fn-caption"
        label="Profile (MVC)"/>
    <vlayout>
        <grid width="500px">
            <columns>
                <column align="right" hflex="min"/>
                <column/>
            </columns>
            <rows>
                <row>
                    <cell sclass="row-title">Account :</cell>
                    <cell><label id="account"/></cell>
                </row>
                <row>
                    <cell sclass="row-title">Full Name :</cell>
                    <cell>
                    <textbox id="fullName"
                    constraint="no empty: Please enter your full name"
                    width="200px"/>
                    </cell>
                </row>
                <row>
                    <cell sclass="row-title">Email :</cell>
                    <cell>
                    <textbox id="email"
                    constraint="/.+@.+\.[a-z]+/: Please enter an e-mail address"
                    width="200px"/>
                    </cell>
                </row>
                <row>
                    <cell sclass="row-title">Birthday :</cell>
                    <cell>
                        <datebox id="birthday"
                        constraint="no future" width="200px"/>
                    </cell>
                </row>
                <row>
                    <cell sclass="row-title">Country :</cell>
                    <cell>
                        <listbox id="country" mold="select" width="200px">
                            <template name="model">
                                <listitem label="${each}" />
                            </template>
                        </listbox>
                    </cell>
                </row>
                <row>
                    <cell sclass="row-title">Bio :</cell>
                    <cell><textbox id="bio" multiline="true"
                        hflex="1" height="200px" />
                    </cell>
                </row>
            </rows>
        </grid>
        <div>You are editing <label id="nameLabel"/>'s profile.</div>
        <hlayout>
            <button id="saveProfile" label="Save"/>
            <button id="reloadProfile" label="Reload"/>
        </hlayout>
    </vlayout>
</window>
```

-   Line 4, 5:
    [*Caption*](http://books.zkoss.org/wiki/ZK%20Component%20Reference/Containers/Caption "wikilink")
    can be used to build compound header with an image for a [
    *Window*](http://books.zkoss.org/wiki/ZK%20Component%20Reference/Containers/Window ).
-   Line 6: [
    *Vlayout*](http://books.zkoss.org/wiki/ZK%20Component%20Reference/Layouts/Vlayout "wikilink") is
    a light-weight layout component which arranges child components
    vertically without splitter, align, and pack support.
-   Line 14: [
    *Cell*](http://books.zkoss.org/wiki/ZK_Component_Reference/Supplementary/Cell "wikilink") is
    used inside *Row*, *Hbox*, or *Vbox* for fully controlling style and
    layout.
-   Line 21, 22, 29, 30, 37: Specify `constraint` attribute of an input
    component can activate input validation feature and we will discuss
    it in later section.
-   Line 43: Some components have multiple molds, and each mold has its
    own style.
-   Line 44: *Template* component can create its child components
    repeatedly upon the data model of parent component, *Listbox*, and
    doesn't has a corresponding visual widget itself. The `name`
    attribute is required and has to be "model" in default case.
-   Line 45: The `${each}` is an implicit variable that you can use
    without declaration inside *Template*, and it represents an object
    of the data model list for each iteration when rendering. We use
    this variable at component attributes to reference the object's
    property with dot notation. In our example, we just set it at a
    *Listitem*'s label.
-   Line 59: [
    *Hlayout*](http://books.zkoss.org/wiki/ZK%20Component%20Reference/Layouts/Hlayout "wikilink"),
    like *Vlayout*, but arranges child components horizontally.


## User Input Validation

We can specify the `constraint` attribute of an input component with [
constraint
rule](http://books.zkoss.org/wiki/ZK Component Reference/Base Components/InputElement#Validation "wikilink")
to activate its input validation feature and the feature can work
without writing any code in a controller. For example:

```xml
    <textbox id="fullName" constraint="no empty: Plean enter your full name"
             width="200px"/>
```

-   The constraint rule means "no empty value allowed" for the
    *Textbox*. If the user input violates this rule, ZK will show the
    message after a colon.

```xml
    <textbox id="email"
             constraint="/.+@.+\.[a-z]+/: Please enter an e-mail address"
             width="200px"/>
```

-   We can also define a constraint rule using a regular expression that
    describes the email format to limit the value in correct format.

```xml
<datebox id="birthday" constraint="no future" width="200px"/>
```

-   The constraint rule means "date in the future is not allowed" and it
    also restricts the available date to choose.

Then, the input component will show the specified error message when an
input value violates a specified constraint rule.

![ ](../images/ze-ch5-email-constraint.png  " center | 500px")


# Initialize Profile Form
We want to create a drop-down list that contains a list of countries for
selection. When a user visit the page, the data in drop-down list should
be ready. To achieve this, we have to initialize a drop-down list in the
controller.

![ ](../images/ze-ch5-collection.png  " center | 300px")

<div style="text-align:center">
<strong>Country List</strong>
</div>

This is made using a *Listbox* in "select" mold. The *Listbox*'s data is
a list of country name strings provided by the controller. In ZK, all
data components are designed to accept a separate data model that
contains data to be rendered. You only have to provide such a data model
and a data component will render the information as specified in the
*Template*. This increases the data model's re-usability and decouples
the data from a component's implementation.

For a *Listbox*, we can provide a
`org.zkoss.zul.ListModelList` object.

'''Initialize data model for a *Listbox* '''

``` java
public class ProfileViewController extends SelectorComposer<Component>{
    ...
    @Wire
    Listbox country;

    @Override
    public void doAfterCompose(Component comp) throws Exception{
        super.doAfterCompose(comp);

        ListModelList<String> countryModel = new ListModelList<String>(CommonInfoService.getCountryList());
        country.setModel(countryModel);

        ...
    }

    ...

}
```

-   Line 10: Create a `ListModelList` object with a list of `String`
-   Line 11: Provide prepared data model object to the component by
    `setModel()`.

When a user visits this page, we want profile data to already appear in
the form and ready to be modified. Hence, we should initialize those
input components in a controller by loading saved data to input
components.

``` java

public class ProfileViewController extends SelectorComposer<Component>{

    //wire components
    @Wire
    Label account;
    @Wire
    Textbox fullName;
    @Wire
    Textbox email;
    @Wire
    Datebox birthday;
    @Wire
    Listbox country;
    @Wire
    Textbox bio;

    //services
    AuthenticationService authService = new AuthenticationServiceChapter3Impl();
    UserInfoService userInfoService = new UserInfoServiceChapter3Impl();

    @Override
    public void doAfterCompose(Component comp) throws Exception{
        super.doAfterCompose(comp);

        ListModelList<String> countryModel = new ListModelList<String>(CommonInfoService.getCountryList());
        country.setModel(countryModel);

        refreshProfileView();
    }

    ...

    private void refreshProfileView() {
        UserCredential cre = authService.getUserCredential();
        User user = userInfoService.findUser(cre.getAccount());
        if(user==null){
            //TODO handle un-authenticated access
            return;
        }

        //apply bean value to UI components
        account.setValue(user.getAccount());
        fullName.setValue(user.getFullName());
        email.setValue(user.getEmail());
        birthday.setValue(user.getBirthday());
        bio.setValue(user.getBio());

        ((ListModelList)country.getModel()).addToSelection(user.getCountry());
        ...
    }
}
```

-   Line 4: Wire ZK components as we talked in chapter 4.
-   Line 17: Service classes that are used to perform business
    operations or get necessary data.
-   Line 28: Load saved data to input components to initialize the View,
    so we should call it after initializing country list.
-   Line 33: This method reloads the saved data from service classes to
    input components.
-   Line 42\~46: Push saved user data to components by `setValue()`.
-   Line 48: Use `ListModelList.addToSelection()` to control the
    *Listbox*'s selection,

## Save & Reload Data


The example application has 2 functions, save and reload, which are both
triggered by clicking a button. If you click the "Save" button, the
application will save your input and show a notification box.

![](../images/ze-ch5-save.png  " center | 500px")

<div style="text-align:center">
**Click "Save" button**

</div>
In this section, we will demonstrate a more flexible way to define an
event listener in a controller with `@Listen` annotation instead of
calling `addEventListener()` method (mentioned in chapter 4).

An event listener method should be public, have a void return type, and
have either no parameter or one parameter of the specific event type
(corresponding to the event listened) with `@Listen` in a controller.
You should specify event listening rule in the annotation's element
value. Then ZK will "wire" the method to the specified components for
specified events. ZK provides various wiring selectors to specify in the
annotation, please refer to [ZK Developer%27s
Reference/MVC/Controller/Wire Event
Listeners](http://books.zkoss.org/wiki/ZK Developer%27s Reference/MVC/Controller/Wire Event Listeners "wikilink").

**Listen "Save" button's clicking**

``` java
public class ProfileViewController extends SelectorComposer<Component>{

    @Listen("onClick=#saveProfile")
    public void doSaveProfile(){
        ...
    }
    ...
}
```

-   Line 3: The `@Listen` will make `doSaveProfile()` be invoked when a
    user clicks a component (`onClick`) whose id is "saveProfile"
    (`#saveProfile`).

We can manipulate components to change the presentation in the event
listener. In `doSaveProfile()`, we get user's input from input
components and save the data to a `User` object. Then show the
notification to the client.

**Handle "Save" button's clicking**

``` java
public class ProfileViewController extends SelectorComposer<Component>{


    @Listen("onClick=#saveProfile")
    public void doSaveProfile(){
        UserCredential cre = authService.getUserCredential();
        User user = userInfoService.findUser(cre.getAccount());
        if(user==null){
            //TODO handle un-authenticated access
            return;
        }

        //apply component value to bean
        user.setFullName(fullName.getValue());
        user.setEmail(email.getValue());
        user.setBirthday(birthday.getValue());
        user.setBio(bio.getValue());

        Set<String> selection = ((ListModelList)country.getModel()).getSelection();
        if(!selection.isEmpty()){
            user.setCountry(selection.iterator().next());
        }else{
            user.setCountry(null);
        }

        userInfoService.updateUser(user);

        Clients.showNotification("Your profile is updated");
    }
    ...
}
```

-   Line 7: In this chapter's example, `UserCredential` is pre-defined
    to "Anonymous". We will write a real case in chapter 8.
-   Line 14: Get users input by calling `getValue()`.
-   Line 19: Get a user's selection for a *Listbox* from its model
    object.
-   Line 28: Show a notification box which is the most easy way to show
    a message to users.

To wire the event listener for "Reload" button's is similar as previous
one, and it pushes saved user data to components using `setValue()`.

``` java
public class ProfileViewController extends SelectorComposer<Component>{


    //wire components
    @Wire
    Label account;
    @Wire
    Textbox fullName;
    @Wire
    Textbox email;
    @Wire
    Datebox birthday;
    @Wire
    Listbox country;
    @Wire
    Textbox bio;

    ...
    @Listen("onClick=#reloadProfile")
    public void doReloadProfile(){
        refreshProfileView();
    }

    ...
}
```

-   Line 21: This method is listed in previous section.

After the above steps, we have finished all functions of the target
application. Quite simple, right? You can see the result at
http://localhost:8080/zkessentials/chapter3/index.zul
