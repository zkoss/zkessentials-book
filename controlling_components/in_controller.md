#In Controller
In this section, we will demonstrate how to redirect users to an
external site with **event listeners** in a **Controller** when they
click an item in the sidebar.

The most commonly used architecture in web applications is *MVC
(Model-View-Controller)* which separates an application into 3 parts.
The *Model* is responsible for exposing data while performing business
logic which is usually implemented by users, and the *View* is responsible
for displaying data which is what ZUL does. The *Controller* can change
the View's presentation and handle events from the View. The benefit of
designing an application in MVC architecture is that your application is
more modularized.

In ZK world, a `org.zkoss.zk.ui.util.Composer` plays
the same role as the Controller and you can assign it to a target
component. Through the composer, you can listen events of the target
component and manipulate target component's child components to change
View's presentation according to your requirement. To create a
Controller in ZK you simply create a class that inherits
`org.zkoss.zk.ui.select.SelectorComposer`.

```java
public class SidebarChapter2Controller extends SelectorComposer<Component>{
    //other codes...
}
```

Then "connect" the controller with a component in the zul by specifying
fully qualified class name in `apply` attribute. After that the
component and all its child components are under the control of the
controller.

**chapter2/sidebar.zul**

```xml
<grid hflex="1" vflex="1" sclass="sidebar"
    id="sidebar"
    apply="org.zkoss.essentials.chapter2.SidebarChapter2Controller">
    <columns>
        <column width="36px"/>
        <column/>
    </columns>
    <rows/>
</grid>
```

-   Line 2: A component id can be used to retrieve the component in a
    composer, please see the next section.
-   Line 3: Apply a controller to a component.
-   Line 8: Here we don't create 3 *Row*s in the zul because we need to
    add an event listener programmatically on each *Row* in the
    composer.


## Wire Components
To control a component, we must retrieve it first. In
`org.zkoss.zk.ui.select.SelectorComposer`, when you
specify a `@Wire` annotation on a field or setter method, the
SelectorComposer will automatically find the component and assign it to
the field or pass it into the setter method. By default
`SelectorComposer` will find the component whose id and type both equal
to the variable name and type respectively.

```java
public class SidebarChapter2Controller extends SelectorComposer<Component>{

    //wire components
    @Wire
    Grid sidebar;

    ...
}
```

-   Line 4,5 : SelectorComposer looks for a *Grid* whose id is "fnList"
    and assign it to the variable `fnList`.


## Initialize the View
It is very common that we need to initialize components when a zul file
is loaded. In our example, we need to create *Row*s of the *Grid* for
the sidebar, therefore we should override a [ composer life-cycle
method](http://books.zkoss.org/wiki/ZK%20Developer's%20Reference/MVC/Controller/Composer#Lifecycle)
`doAfterCompose(Component)`. The passed argument, `comp`, is the
component that the composer applies to, which in our example is the
*Grid*. This method will be called after all the child components under
the component which has the composer applied to it are created, so we
can change components' attributes or even create other components in it.

``` java
public class SidebarChapter2Controller extends SelectorComposer<Component>{

    //wire components
    @Wire
    Grid sidebar;

    //services
    SidebarPageConfig pageConfig = new SidebarPageConfigChapter2Impl();

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
    *Row*s with event listeners and put them into *Grid*. We will
    discuss them in next section.

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
and all changes you made will reflect to clients.

Now we are going to explain how to create components and add an event
listener to respond to users' clicking. Basically, there are 3 steps to
create a component:

1.  Create a component object.
2.  Setup the component's attributes.
3.  Append to the target parent component.

In `constructSidebarRow()` method, we create *Row*s and add an event
listener to each of them.

```java
public class SidebarChapter2Controller extends SelectorComposer<Component>{

    //...

    //wire components
    @Wire
    Grid sidebar;

    //services
    SidebarPageConfig pageConfig = new SidebarPageConfigChapter2Impl();

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

-   Line 21: Append a newly-created *Row* will make it become a child
    component of *Rows*.
-   Line 28: The first step to create a component is instantiating its
    class.
-   Line 32: Append a component to establish the parent-child
    relationship.
-   Line 36: You can change a component's attributes by various setter
    methods and their method names correspond to tag's attribute name.
-   Line 39: We create an `EventListener` anonymous class for
    convenience. Under a clustering environment, your event listener
    class should implement
    `org.zkoss.zk.ui.event.SerializableEventListener`.
-   Line 44: Implement the business logic in`onEvent()` method, and this
    method will be called if the listened event is sent to the server.
    Here we get current execution by
    `org.zkoss.zk.ui.Executions` and redirect a client
    to a new URL.
-   Line 48: Apply the event listener to a *Row* for listening
    `Events.ON_CLICK` event which is triggered by a mouse clicking
    action.

In Line 28 \~ 36, those codes work equally to writing in a zul as
follows:

```xml
<row sclass="sidebar-fn">
    <image/><label/>
</row>
```

After completing above steps, when a user clicks a *Row* on the sidebar,
ZK will call a corresponding `actionListener ` then the browser will be
redirected to a specified URL. You can see the result via
http://localhost:8080/zkessentials/chapter2.
