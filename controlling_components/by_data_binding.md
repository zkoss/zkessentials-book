# MVVM Pattern
In addition to the MVC approach, ZK also allows you to design your
application in another architecture: [ *MVVM
(Model-View-ViewModel)*](http://books.zkoss.org/zk-mvvm-book/8.0/index.html).
This architecture also divides an application into 3 roles: **View**, **Model**,
and **ViewModel**.
* **Model** is responsible for exposing data while performing business
logic which is usually implemented by users.
* **View** is responsible for displaying data which is what ZUL does.
(The View and Model plays the same roles as they do in MVC)
* **ViewModel** acts like a special Controller which is responsible for exposing data from the Model to the View and providing required actions and logic for user requests from the View.

The ViewModel is a View abstraction, which contains a View's state and
behavior. The biggest difference from the Controller in the MVC is that
**ViewModel does not contain any reference to UI components** and knows nothing about the View's visual elements. There should be a role that synchronizes data and handle events and response between View and ViewModel. This role eliminates the dependency between View and ViewModel. Hence, this clear separation between View and ViewModel decouples ViewModel from View and makes ViewModel more reusable and more abstract.


# ZK MVVM Approach
By following this pattern, ZK supports MVVM approach which controls components by the data binding. Under ZK context, the relationship of 3 roles looks like:

* ViewModel, it's just a POJO (plain ordinary Java object) that doesn't need to extend any parent class, neither implements any interface.
* ZK UI components plays View.
* MyServiceClass is not a real class name. It represents any class which is usually implemented by you performs business logic like searching or authentication.

![](/images/mvvm.png)

Since the ViewModel contains no reference to UI components, you cannot
control components directly by their setter and getter. Therefore, ZK supports a data binding mechanism, ZK Bind, to synchronize data and handle events and respond between the View and ViewModel. Additionally, this mechanism also has to bridge events from the View to the action provided by the ViewModel. In this binding system,
the [**binder**](http://books.zkoss.org/wiki/ZK Developer's Reference/MVVM/DataBinding/Binder) plays the key role to operate the whole mechanism. The binder is like a broker and responsible for communication between View and ViewModel.

In brief, MVVM approach allows you to **control components with data binding** (not by API).


{% include "sidebar.md" %}


# Create a ViewModel
VieModel contains the state of the page, so we have to analysis what data to show in the page. Since I need to build sidebar rows, the only data we need is `SidebarPageConfig`. Consequently, we instantiate a `SidebarPageConfigChapter2Impl` and expose its `PageConfig` list by getter, `getSidebarPages()`;
```java
public class SidebarViewModel {

    private SidebarPageConfig pageConfig = new SidebarPageConfigChapter2Impl();

    public List<SidebarPage> getSidebarPages() {
        return pageConfig.getPages();
    }

}
```


# Apply a ViewModel on a Component
To apply a ViewModel, we need to bind a ZK component to the ViewModel by setting its `viewModel` attribute [^1]
with the ViewModel's id in `@id`  and the ViewModel's full-qualified
class name in ` @init` like:

    <div viewModel="@id('vm')@init('foo.MyViewModel')">

The id is like a variable and we access the ViewModel's
properties by the ID, e.g. `vm.name`. Whilst the full-qualified class name is used to instantiate the ViewModel object itself. So the component that a ViewModel is bound to becomes the **Root View Component** for the ViewModel. All child components of this Root View Component are bound to the same ViewModel and its properties, so we usually bind a page's root component to a ViewModel.

Apply the ViewModel like:

**chapter2/sidebar-mvvm.zul**
```xml
<grid viewModel="@id('vm') @init('org.zkoss.essentials.chapter2.SidebarViewModel')">
```

[^1] Since ZK 8, you don't need to specify
`apply="org.zkoss.bind.BindComposer"` explicitly. Because ZK implicitly applies `BindComposer` for you if you specify a ViewModel.


# Binding Data with ViewModel's Properties
Now that ViewModel is prepared and bound to a component, we can bind a
component's attributes to the ViewModel's property. Once the binding is established, ZK will synchronize (load or save) data between components and the ViewModel for us automatically.


{% include "../handling_user_input/component_model_template.md" %}

## Render a list of sidebar pages
`<grid>` is a component that can accept a `ListModel` object (or Java Collection) and renders its `<row>` based on it. Since `SidebarViewModel` has exposed `List<SidebarPage>` via getter method, it is the data model and we can bind it to `model` attribute like:

**chapter2/sidebar-mvvm.zul**
```xml
<grid viewModel="@id('vm') @init('org.zkoss.essentials.chapter2.SidebarViewModel')"
	...
	model="@load(vm.sidebarPages)" >
```


### Define Listbox Template
The last part is to define a template, so that `<grid>` can know how to render its `List<SidebarPage>` with `<row>`. If you don't define it, `<grid>` renders the model with a default built-in template.

```xml
<grid viewModel="@id('vm') @init('org.zkoss.essentials.chapter2.SidebarViewModel')"
	...
	model="@load(vm.sidebarPages)" >
	...
	<rows>
		<template name="model">
			<row sclass="sidebar-fn" >
				<image src="@load(each.iconUri)"/>
				<label value="@load(each.label)"/>
			</row>
		</template>
	</rows>
</grid>
```
- Line 6: The `name` attribute has to be **model** which means it's  [a template for `<listbox>` model](https://www.zkoss.org/wiki/ZK%20Developer's%20Reference/MVC/View/Template/Listbox%20Template).

#### implicit `each`
The variable `each` is an implicit variable that you can use without declaration inside `<template>`, and it represents one object (`SidebarPage`) of the data model for each iteration when rendering. We use `each` with dot notation to reference a data object's property. In each `<row>`, we show an image icon by loading a URL and a label.


# Define Commands
ViewModel also contains View's application logic which are implemented by methods. We call such a method "Command" of the ViewModel. These methods usually manipulate data in the ViewModel, for example adding or deleting an item. The View's behaviors are usually triggered by events from the View. The data binding mechanism also supports binding an event to a ViewModel's command. Firing the component's event will trigger the execution of bound command that means invoking the corresponding command method.

In ZK, to declare a command method in a ViewModel, you should apply
annotation `@Command` to a method. You could specify a command
name which is the method's name by default if no specified.

Our sidebar just has one behavior: redirect a browser to the corresponding URL. In order to get the URL, we need to accept `SidebarPage` as a parameter with [`@BindingParam`](http://books.zkoss.org/zk-mvvm-book/8.0/advanced/parameters.html):

```Java
public class SidebarViewModel {
    ...
    @Command
    public void navigate(@BindingParam("page") SidebarPage page) {
        Executions.getCurrent().sendRedirect(page.getUri());
    }
}
```
Line 5: [`Executions.getCurrent()`](https://www.zkoss.org/javadoc/latest/zk/org/zkoss/zk/ui/Executions.html#getCurrent()) returns the current [`Execution`](https://www.zkoss.org/javadoc/latest/zk/org/zkoss/zk/ui/Execution.html) which is a wrapper of `HttpServletRequest`. [`sendRedirect()`](https://www.zkoss.org/javadoc/latest/zk/org/zkoss/zk/ui/Execution.html#sendRedirect(java.lang.String)) will redirect a browser to the specified URL.

I apply `@BindingParam("page")` in front of the Parameter which means I will pass `SidebarPage` object with a command binding in a zul with the key `page`.


# Handle User Interactions by Command Binding
After we finish binding attributes to the ViewModel's data, we still
need to handle user actions, e.g. clicking. Under the MVVM approach,
we handle events by binding an event attribute (e.g. `onClick`) to a
**Command** of a ViewModel like:

`onClick="@command('mycmd')"`

After we bind an event to a Command, each
time the event is fired, ZK will invoke the corresponding command method.


**chapter2/sidebar-mvvm.zul**
```xml
<rows>
    <template name="model">
        <row sclass="sidebar-fn" onClick="@command('navigate', page=each)">
            ...
        </row>
    </template>
</rows>
```
Line 3: By default the command name is the method name `navigate`. We need to pass the `SidebarPage` with key `page` to the command method with the implicit variable `each`.

After above steps, the sidebar works as expected.
