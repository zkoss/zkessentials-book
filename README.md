# Navigation and Template

Overview
========

In traditional navigation, a user usually switches to different
functions by visiting different pages of an application, a.k.a
page-based navigation. In ZK, we can have another choice to design the
navigation in AJAX-based where users don't need to visit different
pages. In page-based navigation, users need to switch pages frequently,
we should maintain a consistent page design throughout whole application
to help users keep track of where they are. Luckily ZK provides
**Templating** to keep multiple pages in the same style easily.

In this chapter, the example application we are going to build looks as
follows:

![ center | 600px](Tutorial-ch7-ajax-based.png  " center | 600px")

The sidebar is used for navigation control. The lower 4 menu items lead
you to different functions and they only change the central area's
content. All other areas are unchanged to maintain a consistent layout
style among functions.

Templating
==========

In our example application, we want to keep the header, the sidebar, and
the footer unchanged regardless of which function a user chooses. Only
the central area changes its content according to the function chosen by
users.

In page-based navigation, each function is put into a separated page and
we need to have a consistent style. One way is to copy the duplicated
part from one zul to another, but it is hard to maintain. Fortunately,
ZK provides a [
Templating](ZK Developer%27s Reference/UI_Patterns/Templating/Composition "wikilink")
technique that lets you define a template zul and apply it to multiple
zul pages afterwards. All zul pages that apply the same template zul
have the same layout, so changing the template zul can change the layout
of all pages once.

The steps to use templating are:

1.  Create a template zul and define anchors
2.  Apply the template in the target zul and define zul fragments for
    anchors

Then when you visit the target zul page, ZK will insert those fragments
to corresponding anchors upon anchor names and combine them as one page.

Create a Template ZUL File
--------------------------

Creating a template is nothing different from creating a normal zul, but
you should define one or more anchors by specifying annotation
`@insert()` at `self`. You can give any name to identify an anchor that
will be used to insert a zul fragment with the same anchor name later.

**chapter7/pagebased/layout/template.zul**

``` {.xml}
<zk>
    <borderlayout hflex="1" vflex="1">
        <north height="100px" border="none" >
            <include src="/chapter3/banner.zul"/>
        </north>
        <west width="260px" border="none" collapsible="true"
                splittable="true" minsize="300">
            <include src="/chapter7/pagebased/layout/sidebar.zul"/>
        </west>
        <center id="mainContent" autoscroll="true" border="none"
                self="@insert(content)">
        </center>
        <south height="50px" border="none">
            <include src="/chapter3/footer.zul"/>
        </south>
    </borderlayout>
</zk>
```

-   Line 10,11: Define an anchor with name `content`

Apply the Template
------------------

After creating a template zul, we can apply it in another zul with
directive

`&lt;?init class="org.zkoss.zk.ui.util.Composition" arg0="template_path"?&gt;`

This directive tells ZK that this page uses the specified template. Then
we also have to define zul fragments that are inserted to the anchor in
the template zul with annotation `@define(anchorName)` at `self`
attribute. The `anchorName` you specify should correspond to one of
anchors defined in the template zul.

**chapter7/pagebased/index.zul**

``` {.xml}

<?link rel="stylesheet" type="text/css" href="/style.css"?>
<?init class="org.zkoss.zk.ui.util.Composition"
        arg0="/chapter7/pagebased/layout/template.zul"?>
<zk>
    <include self="@define(content)" src="/chapter7/pagebased/home.zul"/>
</zk>
```

-   Line 2,3: Tell ZK that we want to use a template zul for current
    page and give the path of template zul.
-   Line 5: The anchor name `content` correspond to the anchor name
    defined in the template zul in previous section.

After above steps, when you visit
<http://localhost:8080/essentials/chapter7/pagebased/index.zul>, ZK will
render the page based on `template.zul` and attach the *Include*
component to `&lt;center&gt;`.

Page-based Navigation
=====================

Traditional web applications are usually designed with page-based
navigation. Each function corresponds to an independent page with
independent URL. The navigation is very clear for users as they know
where they are from the URL and they can press "go back" button on their
browser to go back to previous pages in history. But the drawback is
users have to wait whole page reloading every time they switch to a
function. Additionally, developers also have to maintain multiple pages
that have similar contents but applying a template zul can reduce this
problem.

![ center |
600px](Tutorial-ch7-page-based-navigation.png  " center | 600px")

<div style="text-align:center">
**Page-based Navigation**

</div>
To build a page-based navigation, first you should prepare pages for
those items in the sidebar.

![ center](Tutorial-ch7-pagebased.png  " center")

From above image, you can see there are four zul pages which correspond
to items in the sidebar under "chpater7\\pagebased"
(index-profile-mvc.zul, index-profile-mvvm.zul, index-todolist-mvc.zul,
index-todolist-mvvm.zul). Then we can link four items of the sidebar to
these zul pages by redirecting a browser.

Next, we apply the template zul created before on those 4 pages.

**/chapter7/pagebased/index-profile-mvc.zul**

``` {.xml}
<?link rel="stylesheet" type="text/css" href="/style.css"?>
<?init class="org.zkoss.zk.ui.util.Composition"
        arg0="/chapter7/pagebased/layout/template.zul"?>
<zk>
    <include self="@define(content)" src="/chapter5/profile-mvc.zul"/>
</zk>
```

-   Line 2,3: Apply the template in
    `/chapter7/pagebased/layout/template.zul`.
-   Line 5: Define a zul fragment for the anchor `content`.

Our example application creates those menu items dynamically upon a
configuration, so we should initialize configuration.

**Page-based navigation's sidebar configuration**

``` {.java}
public class SidebarPageConfigPagebasedImpl implements SidebarPageConfig{

    HashMap<String,SidebarPage> pageMap = new LinkedHashMap<String,SidebarPage>();
    public SidebarPageConfigPagebasedImpl(){
        pageMap.put("zk",new SidebarPage("zk","www.zkoss.org","/imgs/site.png","http://www.zkoss.org/"));
        pageMap.put("demo",new SidebarPage("demo","ZK Demo","/imgs/demo.png","http://www.zkoss.org/zkdemo"));
        pageMap.put("devref",new SidebarPage("devref","ZK Developer Reference","/imgs/doc.png"
                ,"http://books.zkoss.org/wiki/ZK_Developer's_Reference"));

        pageMap.put("fn1",new SidebarPage("fn1","Profile (MVC)","/imgs/fn.png"
                ,"/chapter7/pagebased/index-profile-mvc.zul"));
        pageMap.put("fn2",new SidebarPage("fn2","Profile (MVVM)","/imgs/fn.png"
                ,"/chapter7/pagebased/index-profile-mvvm.zul"));
        pageMap.put("fn3",new SidebarPage("fn3","Todo List (MVC)","/imgs/fn.png"
                ,"/chapter7/pagebased/index-todolist-mvc.zul"));
        pageMap.put("fn4",new SidebarPage("fn4","Todo List (MVVM)","/imgs/fn.png"
                ,"/chapter7/pagebased/index-todolist-mvvm.zul"));
    }

    ...

}
```

-   Line 10\~17: Specify URL and related data for each menu item's
    configuration.

The following codes show how to redirect a user to an independent page
when they click a menu item in the sidebar.

**Controller for page-based navigation**

``` {.java}
public class SidebarPagebasedController extends SelectorComposer<Component>{

    ...

    //wire service
    SidebarPageConfig pageConfig = new SidebarPageConfigPagebasedImpl();

    @Override
    public void doAfterCompose(Component comp) throws Exception{
        super.doAfterCompose(comp);

        //to initial view after view constructed.
        Rows rows = fnList.getRows();

        for(SidebarPage page:pageConfig.getPages()){
            Row row = constructSidebarRow(page.getName(),page.getLabel(),page.getIconUri(),page.getUri());
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

-   Line 15: Create menu items in the sidebar upon configurations with
    *Row*s.
-   Line 39: Add a event listener to redirect a browser to the URL
    specified in the menu item a user clicks.

Visit <http://localhost:8080/essentials/chapter7/pagebased/index.zul>.
You will see the URL changes and whole page reloads each time you click
a different menu item.

AJAX-based Navigation
=====================

When switching between different functions in paged-based navigation,
you find that only central area's content is different among those pages
and other three areas (header, sidebar, and footer) contain identical
content. But in page-based navigation, a browser has to reload all
contents no matter they are identical to previous page when switching to
another function. With AJAX's help, ZK allows you to implement another
navigation way that only updates necessary part of a page instead of
reloading the whole page.

![ center |
600px](Tutorial-ch7-ajax-based-navigation.png  " center | 600px")

<div style="text-align:center">
**AJAX-based Navigation**

</div>
The easiest way to implement AJAX-based navigation is changing `src`
attribute of *Include* component. It can change only partial content of
an page instead of redirecting to another page to achieve the navigation
purpose. This navigation way switches functions by only replacing a
group of components instead of whole page and therefore has faster
response than page-based one. But it doesn't change a browser's URL when
each time switching to a different function. However, if you want users
can keep track of different functions with bookmark, please refer to [
Browser History
Management](ZK_Developer's_Reference/UI_Patterns/Browser_History_Management "wikilink").

We will use the same layout example to demonstrate AJAX-based
navigation.

Below is the index page, its content is nearly the same as the index
page of page based example except it includes a different zul.

**chapter7/ajaxbased/index.zul**

``` {.xml}
<?link rel="stylesheet" type="text/css" href="/style.css"?>
<?init class="org.zkoss.zk.ui.util.Composition"
        arg0="/chapter7/ajaxbased/layout/template.zul"?>
<zk>
    <include id="mainInclude" self="@define(content)"
        src="/chapter7/ajaxbased/home.zul"/>
</zk>
```

-   Line 5,6: We give the component id for we can find it later with ZK
    selector.

This navigation is mainly implemented by changing the `src` attribute of
the *Include* component to switch between different zul pages so that it
only reloads included components without affecting other areas . We
still need to initialize sidebar configuration:

**AJAX-based navigation's sidebar configuration**

``` {.java}
public class SidebarPageConfigAjaxbasedImpl implements SidebarPageConfig{

    HashMap<String,SidebarPage> pageMap = new LinkedHashMap<String,SidebarPage>();
    public SidebarPageConfigAjaxbasedImpl(){
        pageMap.put("zk",new SidebarPage("zk","www.zkoss.org","/imgs/site.png","http://www.zkoss.org/"));
        pageMap.put("demo",new SidebarPage("demo","ZK Demo","/imgs/demo.png","http://www.zkoss.org/zkdemo"));
        pageMap.put("devref",new SidebarPage("devref","ZK Developer Reference","/imgs/doc.png","http://books.zkoss.org/wiki/ZK_Developer's_Reference"));

        pageMap.put("fn1",new SidebarPage("fn1","Profile (MVC)","/imgs/fn.png","/chapter5/profile-mvc.zul"));
        pageMap.put("fn2",new SidebarPage("fn2","Profile (MVVM)","/imgs/fn.png","/chapter5/profile-mvvm.zul"));
        pageMap.put("fn3",new SidebarPage("fn3","Todo List (MVC)","/imgs/fn.png","/chapter6/todolist-mvc.zul"));
        pageMap.put("fn4",new SidebarPage("fn4","Todo List (MVVM)","/imgs/fn.png","/chapter6/todolist-mvvm.zul"));
    }
    ...
}
```

-   Line 9 \~ 12: Because we only need those pages that doesn't have
    header, sidebar, and footer, we can re-use those pages written in
    previous chapters.

In the sidebar controller, we get the *Include* and change its `src`
according to the menu item's URL.

**Controller for AJAX-based navigation**

``` {.java}
public class SidebarAjaxbasedController extends SelectorComposer<Component>{

    @Wire
    Grid fnList;

    //wire service
    SidebarPageConfig pageConfig = new SidebarPageConfigAjaxbasedImpl();

    @Override
    public void doAfterCompose(Component comp) throws Exception{
        super.doAfterCompose(comp);

        //to initial view after view constructed.
        Rows rows = fnList.getRows();

        for(SidebarPage page:pageConfig.getPages()){
            Row row = constructSidebarRow(page.getName(),page.getLabel(),page.getIconUri(),page.getUri());
            rows.appendChild(row);
        }
    }

    private Row constructSidebarRow(final String name,String label, String imageSrc, final String locationUri) {

        //construct component and hierarchy
        Row row = new Row();
        Image image = new Image(imageSrc);
        Label lab = new Label(label);

        row.appendChild(image);
        row.appendChild(lab);

        //set style attribute
        row.setSclass("sidebar-fn");

        //new and register listener for events
        EventListener<Event> onActionListener = new SerializableEventListener<Event>(){
            private static final long serialVersionUID = 1L;

            public void onEvent(Event event) throws Exception {
                //redirect current url to new location
                if(locationUri.startsWith("http")){
                    //open a new browser tab
                    Executions.getCurrent().sendRedirect(locationUri);
                }else{
                    //use iterable to find the first include only
                    Include include = (Include)Selectors.iterable(fnList.getPage(), "#mainInclude")
                            .iterator().next();
                    include.setSrc(locationUri);

                    ...
                }
            }
        };
        row.addEventListener(Events.ON_CLICK, onActionListener);

        return row;
    }

}
```

-   Line 46,47: Since *Include* is not a child component of the
    component that `SidebarAjaxbasedController` applies to, we cannot
    use `@Wire` to retrieve it. Therefore, we use `Selectors.iterable()`
    to get components with the id selector from the page. Because ZK
    combines included components and its parent into one ZK page.
-   Line 48: Change the `src` to the corresponding URL that belongs to
    the clicked menu item.

Visit the
<http://localhost:8080/essentials/chapter7/ajaxbased/index.zul> to see
the result.

Source Code
===========

-   [ZUL
    pages](https://github.com/zkoss/zkessentials/tree/master/src/main/webapp/chapter7)
-   [Java](https://github.com/zkoss/zkessentials/tree/master/src/main/java/org/zkoss/essentials/chapter7)



