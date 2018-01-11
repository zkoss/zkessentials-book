# In Controller
In this section, we will demonstrate how to redirect users to an
external site with **event listeners** in a **Controller** when they
click an item in the sidebar.

# MVC Pattern
A well-known design pattern is **MVC
(Model-View-Controller)** which separates an application into 3 roles:

* **Model** is responsible for exposing data while performing business
logic which is usually implemented by users.
* **View** is responsible for displaying data which is what ZUL does.
* **Controller** can change the View's presentation and handle events sent from the View.

Such architecture follows [SRP (Single Responsibility Principle)](https://en.wikipedia.org/wiki/Single_responsibility_principle), and its benefit is that your application is more modularized. Therefore, modifying one part doesn't have to modify another part.

## ZK MVC Approach
By following this pattern, ZK traditionally supports MVC approach which controls components by calling their API. Under ZK context, the relationship of 3 roles looks like:

![](/images/zk-mvc.png)

* ZK `SelectorComposer`, that implements `Composer`, plays Controller.
* ZK UI components plays View.
* MyServiceClass is not a real class name. It represents any class which is usually implemented by you performs business logic like searching or authentication.


{% include "sidebar.md" %}


# Create a Controller
In ZK world, a `org.zkoss.zk.ui.util.Composer` plays
the same role as the **Controller**, and you can apply it to a target
component. Through the composer, you can handle events of the target
component (and its children) to change View's presentation according to your requirement. To create a Controller in ZK you simply create a class that inherits `org.zkoss.zk.ui.select.SelectorComposer`.

```java
public class SidebarChapter2Controller extends SelectorComposer<Component>{
    //other codes...
}
```

Then "associate" the controller with a component in the zul by specifying fully qualified class name in `apply` attribute. After that the component and all its child components are under the
controller's control.

**chapter2/sidebar.zul**
```xml
<grid apply="org.zkoss.essentials.chapter2.SidebarChapter2Controller"
    hflex="1" vflex="1" sclass="sidebar">
    ...
</grid>
```


## Wire Components
To control a component, we must get its object reference. In
`org.zkoss.zk.ui.select.SelectorComposer`, when you
specify a [`@Wire`](https://www.zkoss.org/wiki/ZK_Developer%27s_Reference/MVC/Controller/Wire_Components) annotation on a field or setter method, the
`SelectorComposer` will automatically find the component and assign it to the field or pass it into the setter method. By default
`SelectorComposer` locates the component whose ID and type both match the variable name and type respectively in the zul, and `@Wire` also supports [selector syntax](https://www.zkoss.org/wiki/ZK_Developer%27s_Reference/MVC/Controller/Wire_Components) to wire.

I can specify a component's ID to get wired by default selector in a controller.
**chapter2/sidebar.zul**
```xml
<grid id="sidebar"
    apply="org.zkoss.essentials.chapter2.SidebarChapter2Controller">
```


```java
public class SidebarChapter2Controller extends SelectorComposer<Component>{

    //wire components
    @Wire
    private Grid sidebar;

    ...
}
```

`SelectorComposer` looks for a `Grid` whose ID is "sidebar" and assign it to the variable `sidebar`.


## Initialize the View
It is very common that we need to initialize components when a zul file
is loaded. In our example, we need to create `<row>` in `<grid>` for
the sidebar, therefore we should override a [ composer life-cycle
method](http://books.zkoss.org/wiki/ZK%20Developer's%20Reference/MVC/Controller/Composer#Lifecycle)
`doAfterCompose(Component)`. The passed argument, `comp`, is the
component that the controller applies to, which in our example is the
`Grid`. ZK will call this method after the applied component, `<grid>` , and its all child components are created, so we can change components' attributes or even create other components in it.

``` java
public class SidebarChapter2Controller extends SelectorComposer<Component>{

    //wire components
    @Wire
    private Grid sidebar;

    //services
    private SidebarPageConfig pageConfig = new SidebarPageConfigChapter2Impl();

    @Override
    public void doAfterCompose(Component comp) throws Exception{
        super.doAfterCompose(comp);

        //initialize view after view construction.
        Rows rows = sidebar.getRows();

        for(SidebarPage page:pageConfig.getPages()){
            Row row = constructSidebarRow(page.getLabel(),page.getIconUri(),page.getUri());
            rows.appendChild(row);
        }
    }
}
```

-   Line 8: Here we demonstrate a configurable architecture, the
    `SidebarPageConfig` stores hyperlink's configuration such as URL,
    and label and we use this configuration to create and setup
    components in the sidebar.
-   Line 12: You have to call super class `doAfterCompose()` method ,
    because it performs initialization like wiring components for you.
-   Line 15 - 20: These codes involve the concept that we have not
    talked about yet. All you have to know for now is these codes create
    `<row>`s with event listeners and put them into *Grid*. We will
    show you the source in the next section.

# Events & Event Listeners

A ZK event (`org.zkoss.zk.event.Event`) is an
abstraction of an activity made by user, a notification made by an
application, and an invocation of server push. For example, a user
clicks a button on the browser, this will trigger `onClick` sent to the
server. If there is an event listener registered for the button's
`onClick` event, ZK will pass the event to the listener to handle it.
The event-listener mechanism allows us to handle all user interaction at
server side.


## Create Components & Event Listeners Programmatically

Manipulating components is the most powerful feature of ZK. You can
change the user interface by creating, removing, or changing components
and all changes you made will reflect to browsers.

Now we are going to explain how to create components and add an event
listener to respond to users' clicking. Basically, there are 3 steps to
create a component:

1.  Create a component object.
2.  Setup the component's attributes.
3.  Append to the target parent component.

In `constructSidebarRow()` method, we create `Row`s and add an event
listener to each of them.

```java
public class SidebarChapter2Controller extends SelectorComposer<Component>{

    //...

    private Row constructSidebarRow(String name,String label, String imageSrc, final String locationUri) {

        //construct component and hierarchy
        Row row = new Row();
        Image image = new Image(imageSrc);
        Label lab = new Label(label);

        row.appendChild(image);
        row.appendChild(lab);

        //set style attribute
        row.setSclass("sidebar-fn");

        //create and register event listener
        EventListener<Event> actionListener = new SerializableEventListener<Event>() {
            private static final long serialVersionUID = 1L;

            public void onEvent(Event event) throws Exception {
                //redirect current url to new location
                Executions.getCurrent().sendRedirect(locationUri);
            }
        };

        row.addEventListener(Events.ON_CLICK, actionListener);

        return row;
    }

}
```

-   Line 8: The first step to create a component is instantiating its
    class.
-   Line 12: Append a component to establish the parent-child
    relationship.
-   Line 16: You can change a component's attributes by various setter
    methods and their method names correspond to tag's attribute name.
-   Line 19: We create an `EventListener` anonymous class for
    convenience. Under a clustering environment, your event listener
    class should implement `org.zkoss.zk.ui.event.SerializableEventListener`.
-   Line 24: Implement the business logic in`onEvent()` method, and ZK will call this method when the listened event is sent to the server. Here we get current execution by
    `org.zkoss.zk.ui.Executions` and redirect a client to a new URL.
-   Line 28: Add the event listener to a `<row>` for listening
    `Events.ON_CLICK` event triggered by a mouse clicking
    action.

In Line 8 \~ 16, those codes work equally to writing a zul as
follows:

```xml
<row sclass="sidebar-fn">
    <image/><label/>
</row>
```

After completing above steps, when a user clicks a `<row>` on the sidebar,
ZK will call a corresponding `actionListener` then the browser will be
redirected to a specified URL. You can see the result via /chapter2/index.zul.
