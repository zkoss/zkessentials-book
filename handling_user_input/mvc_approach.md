# MVC Approach
Under this approach, we implement the event handling and presentation
logic in a controller with no code present in the ZUL file. This
approach makes the responsibility of each role (Model, View, and
Controller) more cohesive and allows you to control components directly.
It is very intuitive and very flexible.


# Construct a Form Style Page
With the concept and technique we talked about in last chapter, it
should be easy to construct a form style user interface as follows. We
uses a two-column `<grid>` to build the form style layout and different
input components to receive user's profile like name and birthday. The
zul file below is included in the `<center>` of `<borderlayout>`.

We build the frame of this form first:

**chapter3/profile-mvc.zul**
```xml
<?link rel="stylesheet" type="text/css" href="/style.css"?>
<window apply="org.zkoss.essentials.chapter3.mvc.ProfileViewController"
    border="normal" hflex="1" vflex="1" contentStyle="overflow:auto">
    <caption src="/imgs/profile.png" sclass="fn-caption"
        label="Profile (MVC)"/>
    <vlayout>
        <grid width="500px">
            ...
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
    [`<caption>`](http://books.zkoss.org/wiki/ZK%20Component%20Reference/Containers/Caption)
    can be used to build compound header with an image for a [
    `<window>`](http://books.zkoss.org/wiki/ZK%20Component%20Reference/Containers/Window ).
-   Line 6: [
    `<vlayout>`](http://books.zkoss.org/wiki/ZK%20Component%20Reference/Layouts/Vlayout) is
    a light-weight layout component which arranges its child components
    vertically without splitter, align, and pack support.
-   Line 11:
[`<hlayout>`](http://books.zkoss.org/wiki/ZK%20Component%20Reference/Layouts/Hlayout), like `<vlayout>`, but arranges its child components horizontally.

Then let's put components in a Grid to arrange them as a form style. A `<grid>` is basically composed by `<columns>` and `<rows>`:

```xml
<grid>
    <columns>

    </columns>
    <rows>

    </rows>
</grid>    
```
* `<columns`> can have `<column>` (no 's'), and `<rows>` can have `<row>` (no 's').


**chapter3/profile-mvc.zul**
```xml
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
                    <textbox id="fullName" width="200px"/>
                    </cell>
                </row>
                <row>
                    <cell sclass="row-title">Email :</cell>
                    <cell>
                    <textbox id="email" width="200px"/>
                    </cell>
                </row>
                <row>
                    <cell sclass="row-title">Birthday :</cell>
                    <cell>
                        <datebox id="birthday" width="200px"/>
                    </cell>
                </row>
                <row>
                    <cell sclass="row-title">Country :</cell>
                    <cell>
                        ...
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
```
- Line 3: [`hflex="min"`](https://www.zkoss.org/wiki/ZK_Developer%27s_Reference/UI_Patterns/Hflex_and_Vflex#Minimum_Flexibility) can limit the column's width just wider enough to hold each row's content without a line break.
-   Line 8:
[`<cell>`](http://books.zkoss.org/wiki/ZK_Component_Reference/Supplementary/Cell) is
    used inside `<row>`, `<hbox>`, or `<vbox>` to fully control a column's align, row/column span, and width in an individual row.


{% include "input_validation.md" %}





# Save & Reload Data

The example application has 2 functions, save and reload, which are both
triggered by clicking a button. If you click the "Save" button, the
application will save your input and show a notification box.

<div style="text-align:center">
<img src="../images/ze-ch5-save.png" >
</div>
<div style="text-align:center">
<strong>Click "Save" button</strong>
</div>

In this section, we will demonstrate a more flexible way to define an
event listener in a controller with [`@Listen`](https://www.zkoss.org/wiki/ZK_Developer%27s_Reference/MVC/Controller/Wire_Event_Listeners) other than
calling `addEventListener()`.

An event listener method should be public, have a void return type, and
have either no parameter or one parameter of the specific event type
(corresponding to the event listened) with `@Listen` in a controller.
You should specify event listening rule in the annotation's element
value. Then ZK will "wire" the method to the specified components for
specified events. ZK provides [various wiring selectors](http://books.zkoss.org/wiki/ZK Developer%27s Reference/MVC/Controller/Wire Event Listeners) to specify in the
annotation.

## Listen "Save" button's clicking

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


We can manipulate components to change the UI in the event
listener. In `doSaveProfile()`, we get a user's input from input
components and save the data to a `User` object. Then show a
notification to the client.

## Handle "Save" button's clicking

``` java
public class ProfileViewController extends SelectorComposer<Component>{
....

    @Listen("onClick=#saveProfile")
    public void doSaveProfile(){
        UserCredential userCredential = authService.getUserCredential();
        User user = userInfoService.findUser(userCredential.getAccount());
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

-   Line 7: In this chapter's example, `UserCredential` is initialized
    with "Anonymous". We will put a user data in chapter 8.
-   Line 14: Get users input by calling `getValue()`.
-   Line 19: Get a user's selection for a `Listbox` from its model
    object.
-   Line 28: Show a notification box which is the most easy way to show
    a message to users.


## Handle "Reload" Button's Clicking
To wire the event listener for "Reload" button's is similar as the previous
one, but we use a different selector this time. And the method pushes saved user data to components using `setValue()`, so we can just call previously-implemented `refreshProfileView()`.

``` java
public class ProfileViewController extends SelectorComposer<Component>{

    ...
    @Listen("onClick = button[label = 'Reload']")
    public void doReloadProfile(){
        refreshProfileView();
    }

    ...
}
```

-   Line 6: This method is listed in the previous section.

After the above steps, we have finished all functions of the target
application. Quite simple, right? Please run /chapter3/index.zul to see the result.
