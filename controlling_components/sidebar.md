# Sidebar
Let's start from making the sidebar work.

![](/images/sidebar.png)

I build it with `<grid>` and 2 columns, a table-looking component.
```xml
<grid hflex="1" vflex="1" sclass="sidebar">
    <columns>
        <column width="36px"/>
        <column/>
    </columns>
    <rows>
    </rows>
</grid>
```
In real application, the items in the sidebar are usually built dynamically from a database or upon a user's permission. Hence, I also don't put static `<row>` in the zul, and I will create them dynamically based on data at the server.

When users click a row, they will be brought to the corresponding site.


## Sidebar Page Config
We implement our sidebar configuration including its name, image icon path, and corresponding URL, etc. We will create each `<row>` based on `SidePage`.

```Java
public class SidebarPageConfigChapter2Impl implements SidebarPageConfig{

	HashMap<String,SidebarPage> pageMap = new LinkedHashMap<String,SidebarPage>();
	public SidebarPageConfigChapter2Impl(){		
		pageMap.put("fn1",new SidebarPage("zk","ZK","/imgs/site.png","http://www.zkoss.org/"));
		pageMap.put("fn2",new SidebarPage("demo","ZK Demo","/imgs/demo.png","http://www.zkoss.org/zkdemo"));
		pageMap.put("fn3",new SidebarPage("devref","ZK Developer Reference","/imgs/doc.png","http://books.zkoss.org/wiki/ZK_Developer's_Reference"));
	}

	...
}
```
