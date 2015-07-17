# Include a Separate Page

To complete the page, we need to put those individual pages into
corresponding area of the *Border Layout*.

For all areas, we use *Include* component to combine separated pages.
This component can combine a separated zul for you when the parent zul
is visited. This usage is presented below:

**chapter1/index.zul**

```xml

<?link rel="stylesheet" type="text/css" href="/style.css"?>
<zk>
    <borderlayout hflex="1" vflex="1">
        <north height="100px" border="none" >
            <include src="/chapter1/banner.zul"/>
        </north>
        <west width="260px" border="none" collapsible="true"
                splittable="true" minsize="300">
            <include src="/chapter1/sidebar.zul"/>
        </west>
        <center id="mainContent" autoscroll="true">
            <include src="/chapter1/main.zul"/>
        </center>
        <south height="50px" border="none">
            <include src="/chapter1/footer.zul"/>
        </south>
    </borderlayout>
</zk>
```

-   Line 1: This directive links a external style sheet under root
    folder.
-   Line 5, 8,11, 14: Specify a separated zul path at `src` attribute to
    include a page into current page.

After including 4 separated zul pages, we complete the example of this
chapter. You can visit *<http://localhost:8080/essentials/chapter1>* to
see the result. Since we set welcome file to "index.zul" in web.xml,
*<http://localhost:8080/essentials/chapter1/index.zul>* will be visited
by default.
