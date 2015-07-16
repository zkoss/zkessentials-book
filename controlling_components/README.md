# Controlling Components

We can contruct the user interfacw with ZK components but also control them. In this chapter, we continue to use the previous chapter's example, but we will remove the 3 items with hyper links in the sidebar and replace them with a redirecting action. To achieve this, we will write code in Java for each item to respond to a user's clicking and redirect the user to an external site.


# In Zscript

The simplest way to respond to a user's clicking is to write an event
listener method and invoke it in the `onClick` attribute. We can define
an event listener in Java inside a `<zscript>` element and those codes
will be interpreted when the ZUL is visited. This element also allows
other scripting languages such as JavaScript, Ruby, or Groovy.
`<zscript>` is *very suitable for fast prototyping* and it's interpreted
when a zul page is evaluated. You can see the changed result without
re-deployment. But it has issues in performance and clustering
environment, *we don't recommend to use it in production environment*.

**Event listener redirect()**

```xml
<grid hflex="1" vflex="1" sclass="sidebar">
    <zscript><![CDATA[
        //zscript code, it runs on server site, use it for fast prototyping
        java.util.Map sites = new java.util.HashMap();

        sites.put("zk","http://www.zkoss.org/");
        sites.put("demo","http://www.zkoss.org/zkdemo");
        sites.put("devref","http://books.zkoss.org/wiki/ZK_Developer's_Reference");


        void redirect(String name){
            String loc = sites.get(name);
            if(loc!=null){
                execution.sendRedirect(loc);
            }
        }
    ]]></zscript>
...
```

-   Line 2: It's better to enclose your code with `<![CDATA[ ]]>`.
-   Line 11: Define an event listener method like a normal Java method
    and it redirects a browser according to the passed variable.
-   Line 14: The [execution](http://books.zkoss.org/wiki/ZUML%20Reference/EL%20Expressions/Implicit%20Objects%20(Predefined%20Variables)
    is a implicit variable which you can use it directly without
    declaration. It represents an execution of a client request that
    holds relevant information.

After defining the event listener, we should specify it in a *Row*'s
event attribute `onClick` because we want to invoke the event listener
when clicking a *Row*.

**Invoke event listeners at "onClick"**

```xml
<grid>
    ...
    <rows>
        <row sclass="sidebar-fn" onClick='redirect("zk")'>
            <image src="/imgs/site.png"/> ZK
        </row>
        <row sclass="sidebar-fn" onClick='redirect("demo")'>
            <image src="/imgs/demo.png"/> ZK Demo
        </row>
        <row sclass="sidebar-fn" onClick='redirect("devref")'>
            <image src="/imgs/doc.png"/> ZK Developer Reference
        </row>
    </rows>
</grid>
```

Now if you click a *Row* of the *Grid* in the sidebar, your browser will
be redirected to a corresponding site.

This approach is very simple and fast, so it is especially suitable for
building a prototype. However, if you need a better architecture for
your application, you had better not use ZScript.


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
    id="fnList"
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
    Grid fnList;

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
    Grid fnList;

    //services
    SidebarPageConfig pageConfig = new SidebarPageConfigChapter2Impl();

    @Override
    public void doAfterCompose(Component comp) throws Exception{
        super.doAfterCompose(comp);

        //initialize view after view construction.
        Rows rows = fnList.getRows();

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
    Grid fnList;

    //services
    SidebarPageConfig pageConfig = new SidebarPageConfigChapter2Impl();

    @Override
    public void doAfterCompose(Component comp) throws Exception{
        super.doAfterCompose(comp);

        //initialize view after view construction.
        Rows rows = fnList.getRows();

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
    <javadoc>org.zkoss.zk.ui.event.SerializableEventListener</javadoc>.
-   Line 44: Implement the business logic in`onEvent()` method, and this
    method will be called if the listened event is sent to the server.
    Here we get current execution by
    <javadoc>org.zkoss.zk.ui.Executions</javadoc> and redirect a client
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
*<http://localhost:8080/essentials/chapter2>*.

