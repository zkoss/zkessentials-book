# Shadow Components

ZK 8 introduces a new set of components: **shadow components**. Shadow components, which are similar to [shadow DOM](http://w3c.github.io/webcomponents/spec/shadow/), neither create a corresponding component on the server side nor a widget on the client side. They simply inject their child components into the current page. If we use them with data binding, a page can dynamically change upon a ViewModel's data. Since shadow components have flow control and iteration function, if you identify a repeating view pattern appearing in your application, you can implement that view pattern with shadow components to modulize your view. Hence, their most powerful feature is **making the view reusable**.

Below is the list of shadow components:

* `<apply>`: allows you to choose which template to be applied. It will look up the template inside-out recursively.
* `<forEach>`: allows you to iterate over a collection of objects. Specify the collection by using the `items` attribute, and you can access the current item through a variable specified at the `var` attribute.
* `<if>`: allows the conditional execution of its body according to the value of the test attribute.
* `<choose>`/`<when>`/`<otherwise>`: they are used for logic and flow control like Java's `switch`/`case`/`default` statement.


We can use shadow components anywhere on a zul. For example, we can create a component based on a condition:
```xml
<if test="@load(vm.readonly)">
    <button label="Edit"/>
</if>
```

Or create a collection of components:
```xml
<forEach begin="0" end="3">
	<button label="${'Button'+=each}"/>
</forEach>
```

## Navigation Menu Example
In this chapter, we will demonstrate the power of shadow components with a navigation menu example shown below:
![](../images/ze-ch7-menu.png)

The menu is made by templates containing shadow components, and it can render different menu hierarchies without any change. You can switch 2 sets of menu hierarchy via the radio group on the right hand side. This example demonstrates how to reuse a common view pattern.

### Menu Data Structure
We store a menu hierarchical structure in a linked list, and each node in the list may have sub-menus.

```java
package org.zkoss.essentials.chapter5.template;

import java.util.*;
import org.zkoss.bind.annotation.*;
import org.zkoss.zk.ui.event.SelectEvent;
import org.zkoss.zkmax.zul.Navitem;

public class MenuViewModel {

    private List<MenuNode> menuHierarchy = null;
    ...
}
```


```java
package org.zkoss.essentials.chapter5.template;

import java.util.List;

public class MenuNode {

    private String label;
    private String iconSclass;
    private List<MenuNode> subMenus;
    ...
}
```



