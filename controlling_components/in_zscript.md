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
