# Create a ViewModel
ViewModel is an abstraction of View which contains the View's data,
state and behavior. It extracts the necessary data to be displayed on
the View from one or more Model classes. Those data are exposed through
getter and setter method like JavaBean's property. ViewModel is also a
"Model of the View". It contains the View's state (e.g. user's
selection, whether a component is enabled or disabled) that might change
during user interaction.

In ZK, the ViewModel can simply be a POJO which contains data to display
on the ZUL and doesn't have any components. The example application
displays 2 kinds of data: the user's profile and country list in the
`Listbox`. The ViewModel should look like the following:

**Define properties in a ViewModel**
``` java

public class ProfileViewModel implements Serializable{

    //services
    AuthenticationService authService = new AuthenticationServiceChapter3Impl();
    UserInfoService userInfoService = new UserInfoServiceChapter3Impl();

    //data for the view
    User currentUser;

    public User getCurrentUser(){
        return currentUser;
    }

    public List<String> getCountryList(){
        return CommonInfoService.getCountryList();
    }

    @Init // @Init annotates a initial method
    public void init(){
        UserCredential cre = authService.getUserCredential();
        currentUser = userInfoService.findUser(cre.getAccount());
        if(currentUser==null){
            //TODO handle un-authenticated access
            return;
        }
    }
    ...
}
```

-   Line 4,5: The ViewModel usually contains service classes that are
    used to get data from them or perform business logic.
-   Line 8, 10: We should define current user profile data and its
    getter method to be displayed in the zul.
-   Line 14: ViewModel exposes its data by getter methods, it doesn't
    have to define a corresponding member variable. Hence we can expose
    country list by getting from the service class.
-   Line 18: There is a marker annotation `@Init` for a method which
    should be at most one in each ViewModel and ZK will invoke this
    method after instantiating a ViewModel class. We should perform
    initialization in it, e.g. get user credential to initialize
    `currentUser`.



# Apply a ViewModel on a Component

Before data binding can work, we must apply a composer called
**org.zkoss.bind.BindComposer**. It will create a **binder** for the
ViewModel and instantiate the ViewModel's class. Then we should bind a
ZK component to our ViewModel by setting its `'''viewModel'''` attribute
with the ViewModel's id in `@id`  and the ViewModel's full-qualified
class name in ` @init`. The id is used to reference the ViewModel's
properties, e.g. `vm.name`, whilst the full-qualified class name is used
to instantiate the ViewModel object itself. So that component becomes
the **Root View Component** for the ViewModel. All child components of
this Root View Component can be bound to the same ViewModel and its
properties, so we usually bind the root component of a page to a
ViewModel.

``` xml

<window apply="org.zkoss.bind.BindComposer"
    viewModel="@id('vm') @init('org.zkoss.essentials.chapter3.mvvm.ProfileViewModel')"
    border="normal" hflex="1" vflex="1" contentStyle="overflow:auto">
...
</window>
```

-   Line 1: Under MVVM approach, the composer we apply is fixed
    **org.zkoss.bind.BindComposer**.
-   Line 2: Specify ViewModel's id with ` @id ` and the its
    full-qualified class name in ` @init ` for the binder.

## Data Binding to ViewModel's Properties

Now that ViewModel is prepared and bound to a component, we can bind a
component's attributes to the ViewModel's property. The binding between
an attribute and a ViewModel's property is called "property binding".
Once the binding is established, ZK will synchronize (load and save)
data between components and the ViewModel for us automatically.

![ ](/images/ze-Mvvm-databinding-role.png)

Let's demonstrate how to make *Listbox* load a list of country name from
the ViewModel. We have talked about the data model
concept in previous MVC approach section, and we also need to prepare a model object that
are defined in one of our ViewModel's properties, `countryList`. You
might find `getCountryList()` return a `List` instead of a
`ListModelList`, but don't worry. ZK will convert it automatically. We
use `@load` to load a ViewModel's property to a component's attribute
and `@save` to save an attribute value into a ViewModel's property
(usually for an input component). If both loading and saving are
required, we could use `@bind`.

``` xml
...
    <cell>
        <listbox model="@load(vm.countryList)" mold="select" width="200px">
            <template name="model">
                <listitem label="@load(each)" />
            </template>
        </listbox>
    </cell>
...
```

-   Line 3: We setup a load binding with `@load`. The `vm` is the
    ViewModel's id we specified at `@id` in previous section and the
    target property (`countryList`) can be referenced in dot notation.
-   Line 4: *Template* component, we have explained in MVC approach
    section, can create its child components repeatedly upon the data
    model of parent component.
-   Line 5: The implicit variable `each` which you can use without
    declaration inside *Template* represents each object in the data
    model for each iterative rendering (It represents String object of a
    country name in this example). We use this variable to access
    objects of data model. In our example, we just make it as a
    *Listitem*'s label.

In MVC approach, we have to call an input component's getter method
(e.g. `getValue()` ) to collect user input. But in MVVM approach, ZK
will save user input back to a ViewModel automatically. For example in
the below zul, user input is saved automatically when you move the focus
out of the *Textbox*.

``` xml
        <textbox value="@bind(vm.currentUser.fullName)"
            constraint="no empty: Plean enter your full name" width="200px"/>
```

For the property `currentUser`, we want to both save user input back to
the ViewModel and load value from the ViewModel, so we should use the
`@bind` at `value` attribute. Notice that you can bind `selectedItem` to
a property, then the user's selection can be saved automatically to the
ViewModel.

``` xml
...
    <rows>
        <row>
            <cell sclass="row-title">Account :</cell>
            <cell>
                <label value="@load(vm.currentUser.account)"/>
            </cell>
        </row>
        <row>
            <cell sclass="row-title">Full Name :</cell>
            <cell>
                <textbox value="@bind(vm.currentUser.fullName)"
                    constraint="no empty: Plean enter your full name"
                        width="200px"/>
            </cell>
        </row>
        <row>
            <cell sclass="row-title">Email :</cell>
            <cell>
                <textbox value="@bind(vm.currentUser.email)"
                constraint="/.+@.+\.[a-z]+/: Please enter an e-mail address"
                    width="200px"/>
            </cell>
        </row>
        <row>
            <cell sclass="row-title">Birthday :</cell>
            <cell>
                <datebox value="@bind(vm.currentUser.birthday)"
                    constraint="no future" width="200px"/>
            </cell>
        </row>
        <row>
            <cell sclass="row-title">Country :</cell>
            <cell>
                <listbox model="@load(vm.countryList)"
                    selectedItem="@bind(vm.currentUser.country)"
                        mold="select" width="200px">
                    <template name="model">
                        <listitem label="@load(each)" />
                    </template>
                </listbox>
            </cell>
        </row>
        <row>
            <cell sclass="row-title">Bio :</cell>
            <cell>
                <textbox value="@bind(vm.currentUser.bio)"
                    multiline="true" hflex="1" height="200px" />
            </cell>
        </row>
    </rows>

...
```

-   Line 12, 20, 28, 47: Use `@bind` to save user input back to the
    ViewModel and load value from the ViewModel.
-   Line 36: Bind `selectedItem` to `vm.currentUser.country` and the
    selected country will be saved to `currentUser`.
