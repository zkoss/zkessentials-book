# AJAX-based Navigation - MVVM approach

Surely we can also implement AJAX-based navigation in MVVM approach. Every page are quite similar with previous section. Just switch different pages with data binding.

In this example, we modularize each page with an independent ViewModel. In order to communicate between sidebar and content area that are bound with different ViewModels, we need to use [global command binding](http://books.zkoss.org/zk-mvvm-book/8.0/data_binding/global_command_binding.html). You can treat it as a command binding mentioned in previous chapter, but it can invoke a command method declared in other ViewModels.

The basic idea is: sidebar sends a global command when a user clicks an item then content area change *Include* component's `src` attribute to navigate pages.


**chapter7/ajaxbased_mvvm/sidebar.zul**
```xml
    <rows>
		<template name="model">
			<row sclass="sidebar-fn" onClick="@global-command('onNavigate', page=each)">
				<image src="@load(each.iconUri)"/>
				<label value="@load(each.label)"/>
			</row>
		</template>
	</rows>
```
- Line 3: Specify a global command binding and pass `SidebarPage` as a parameter


**chapter7/ajaxbased_mvvm/mainContent.zul**
```xml
<zk>
	<include id="mainInclude"
		viewModel="@id('vm') @init('org.zkoss.essentials.chapter7.ajaxbased.mvvm.NavigationViewModel')"
		src="@load(vm.includeSrc)" />
</zk>
```

The code below demonstrate how to declare a global command method and receive a parameters.
```java
package org.zkoss.essentials.chapter7.ajaxbased.mvvm;

import org.zkoss.bind.annotation.BindingParam;
import org.zkoss.bind.annotation.GlobalCommand;
import org.zkoss.bind.annotation.NotifyChange;
import org.zkoss.essentials.services.SidebarPage;
import org.zkoss.zk.ui.Executions;

public class NavigationViewModel {

	private String includeSrc = "/chapter7/ajaxbased/home.zul";

	@GlobalCommand("onNavigate")
	@NotifyChange("includeSrc")
	public void onNavigate(@BindingParam("page") SidebarPage page) {
		String locationUri = page.getUri();
		String name = page.getName();

		//redirect current url to new location
		if(locationUri.startsWith("http")){
			//open a new browser tab
			Executions.getCurrent().sendRedirect(locationUri);
		} else {
			includeSrc = locationUri;

			//advance bookmark control,
			//bookmark with a prefix
			if(name!=null){
				Executions.getCurrent().getDesktop().setBookmark("p_"+name);
			}
		}
	}

	public String getIncludeSrc() {
		return includeSrc;
	}

}
```


# Source Code
-   [ZUL
    pages](https://github.com/zkoss/zkessentials/tree/master/src/main/webapp/chapter7)
-   [Java](https://github.com/zkoss/zkessentials/tree/master/src/main/java/org/zkoss/essentials/chapter7)
