# Create a ViewModel
ViewModel is an abstraction of View which contains the View's data,
state and behavior. It extracts the necessary data to be displayed on
the View from one or more Model classes. Those data are exposed through
getter and setter method like JavaBean's property. So that you can think ViewModel is  a
"Model of the View". It contains the View's state (e.g. user's
selection, whether a component is enabled or disabled) that might change
during user interaction.

In ZK, a ViewModel can simply be a POJO which contains data to display
on the ZUL and doesn't have any components. The example application
displays 2 kinds of data:
* the user's profile
* country list

The ViewModel should look like the following:
``` java
public class ProfileViewModel implements Serializable{

    //services
    private AuthenticationService authService = new AuthenticationServiceChapter3Impl();
    private UserInfoService userInfoService = new UserInfoServiceChapter3Impl();

    //data for the view
    private User currentUser;

    public User getCurrentUser(){
        return currentUser;
    }

    public List<String> getCountryList(){
        return CommonInfoService.getCountryList();
    }
}
```

-   Line 4,5: The ViewModel usually contains service classes that are
    used to get data from them or perform business logic.
-   Line 8, 10: We should declare current user profile and its
    getter method to be displayed in the zul.
-   Line 14: ViewModel exposes its data by getter methods, it doesn't
    have to define a corresponding member variable. Hence we can expose
    country list by getting from the service class.


## Initialize a ViewModel

Since the ViewModel is just a POJO, we can initialize its member fields in a constructor.
``` java
public class ProfileViewModel implements Serializable{
...
    public ProfileViewModel(){
		UserCredential userCredential = authService.getUserCredential();
		currentUser = userInfoService.findUser(userCredential.getAccount());
		if(currentUser==null){
			//TODO handle un-authenticated access
			return;
		}
	}
}    
```

If you need ZK context object like `Desktop` at initialization, please refer to [`@Init`](http://books.zkoss.org/zk-mvvm-book/8.0/advanced/parameters.html).

# Apply a ViewModel on a Component

To apply a ViewModel, we need to bind a ZK component to the ViewModel by setting its `viewModel` attribute [^1]
with the ViewModel's id in `@id`  and the ViewModel's full-qualified
class name in ` @init` like:

    <div viewModel="@id('vm')@init('foo.MyViewModel')">

The id is like a variable and we access the ViewModel's
properties by the ID, e.g. `vm.name`. Whilst the full-qualified class name is used to instantiate the ViewModel object itself. So the component that a ViewModel is bound to becomes the **Root View Component** for the ViewModel. All child components of this Root View Component are bound to the same ViewModel and its properties, so we usually bind a page's root component to a ViewModel.


``` xml
<window viewModel="@id('vm') @init('org.zkoss.essentials.chapter3.mvvm.ProfileViewModel')"
    border="normal" hflex="1" vflex="1" contentStyle="overflow:auto">
...
</window>
```
-   Line 1: Specify ViewModel's id with ` @id ` and the its
    full-qualified class name in ` @init `.


[^1] Since ZK 8, you don't need to specify
`apply="org.zkoss.bind.BindComposer"` explicitly. Because ZK implicitly applies `BindComposer` for you if you specify a ViewModel.


# Binding Data with ViewModel's Properties

Now that ViewModel is prepared and bound to a component, we can bind a
component's attributes to the ViewModel's property. The binding between
an attribute and a ViewModel's property is called **property binding**.
Once the binding is established, ZK will synchronize (load or save)
data between components and the ViewModel for us automatically.

![ ](/images/ze-Mvvm-databinding-role.png)


## Load a User
For the first row of this form, we want to show the user name, then we can load `User`'s `account` property to a `<label>` `value` attribute with data binding syntax `@Load`:
``` xml
...
    <rows>
        <row>
            <cell sclass="row-title">Account :</cell>
            <cell>
                <label value="@load(vm.currentUser.account)"/>
            </cell>
        </row>
        ...
    </rows>    
```
* Line 5: `vm` is the ViewModel's id. We use "dot notation" to access an object's properties. Then ZK actually calls getter for you, hence, `vm.currentUser.account` will invoke `getCurrentUser().getAccount()`.


{% include "../component_model_template.md" %}

## Load a Collection Object, Country List
This form needs a drop-down list that contains a list of countries. When a user visits the page, the data in drop-down list should be ready. To achieve this, we have to load the country list from the ViewModel.
![ ](/images/ze-ch5-collection.png)

In order to provide a dropdown list, we put a `<listbox>` in a `select` mold.
 ``` xml
     <cell>
         <listbox mold="select" width="200px">
         </listbox>
     </cell>
 ```

### Load a Data Model
Our ViewModel already returns a countryList. You might find `getCountryList()` return a `List` instead of a `ListModelList`, but don't worry. ZK will convert it automatically. To make the countryList as a data model of `<listbox>`, we have to bind it at `model` attribute:

``` xml
    <cell>
        <listbox model="@load(vm.countryList)" mold="select" width="200px">
        </listbox>
    </cell>
```

### Define Listbox Template
The last part is to define a template, so that `<listbox>` can know how to render its data model with `<listitem>`. If you don't define it, `<listbox>` renders the model with a default built-in template.

``` xml
    <cell>
        <listbox model="@load(vm.countryList)" mold="select" width="200px">
            <template name="model">
                <listitem label="@load(each)" />
            </template>
        </listbox>
    </cell>
...
```
- Line 3: The `name` attribute has to be **model** which means it's  [a template for `<listbox>` model](https://www.zkoss.org/wiki/ZK%20Developer's%20Reference/MVC/View/Template/Listbox%20Template).
-   Line 4: The `each` is an implicit variable that you can use
    without declaration inside `<template>`, and it represents one object of the data model for each iteration when rendering. We use
    this variable with dot notation at component attributes to reference a data object's property . In our example, we just set it at `<listitem>`'s label.

After above steps, you can see a list of country in the form.
