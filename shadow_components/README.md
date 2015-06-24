# Navigation Menu

In addition to `<apply>`, there are other shadow components like `<if>`, `<choose>`, `<when>`, and `<forEach>`. Applying a layout template doesn't demonstrate actual power of shadow components. The most powerful part of shadow components is **making view snippet reusable**.

We will demonstrate its power with a navigation menu example below:
![](images/ze-ch7-menu.png)

The menu is made by templates contains shadow components, and it can render differents menu hierarchies without any change. You can switch 2 sets of menu hierarchy via radio group on the right hand side. This example demonstrates how to reuse a common view pattern.

## Menu Hierarchy
We store a menu hierarchical structure in a linked list, and each node in the list might have sub-menu or not.

```java
package org.zkoss.essentials.chapter7.template;

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
package org.zkoss.essentials.chapter7.template;

import java.util.List;

public class MenuNode {

	private String label;
	private String iconSclass;
	private List<MenuNode> subMenus;
	...
}
```

## Render a Collection
Starting from the basic, we can just render the list of menu node with `<forEach>` (notice the captalized 'E') and `<navitem>`. The `<forEach>` can iterate over a collection of objects. Specifying the collection by the `items` attribute and access the current item through a variable named `each`. Or you can define the current item variable by the `var` attribute e.g. if you write `<forEach items="@load(menuItems)" var="menu">` then you should use `menu` in a data binding expression like `<navitem label="@load(menu.label)">`.


**chapter7/template/menu.zul**
```xml
<navbar id="navbar" orient="horizontal" collapsed="false">
	<apply template="iterate" menuItems="@ref(vm.menuHierarchy)"/>
	<template name="iterate">
		<forEach items="@load(menuItems)">
			<navitem label="@load(each.label)"
				iconSclass="@load(each.iconSclass)">
			</navitem>
		</forEach>
	</template>
</navbar>
```
- Line 2: We pass a parameter by `menuItems="@ref(vm.menuHierarchy)"`. Therefore, we can access the menu list in `<forEach>` by `items="@load(menuItems)"` (line 4).



## Dynamic Template
Next, we render a menu node upon its sub-menu existence. If a menu node has a sub-menu, we render it by `<nav>`, otherwise render it by `<navitem>`. To achieve this, we have to use shadow components for control structure, `<choose>`, `<when>`, and `<otherwise>`. (There is an `<if>` which we don't use in the example.)
* Ther `<choose>` is just like Java `switch` statement.
* The `<when>` is like `case` statement, and you should specify a data binding expression on `test` attribute to be evaluated.
* The `<otherwise>` works like `default` case in a `switch` statement to specify a default action.

Now, we can test whether a menu node has a subMenus or not by `test="@load(empty each.subMenus)"`. If it does, apply the template `nav`, otherwise apply `navitem`.

```xml
<template name="iterate">
	<forEach items="@load(menuItems)">
		<choose>
			<when test="@load(empty each.subMenus)">
				<navitem label="@load(each.label)" />
			</when>
			<otherwise>
				<nav label="@load(each.label)" iconSclass="@load(each.iconSclass)"/>
			</otherwise>
		</choose>
	</forEach>
</template>
```

In this simple case (just 2 choices), we can write it in a simpler way by creating 2 templates for `nav` and `navitem` repectively first.

```xml
<template name="nav">
	<nav label="@load(menuItem.label)" iconSclass="@load(menuItem.iconSclass)"/>
</template>
```
```xml
<template name="navitem" >
	<navitem label="@load(menuItem.label)" />
</template>
```

Then replace `<choose>`/`<when>`/`<otherwise>` with ternary operator `?` like:

```xml
<template name="iterate">
    <apply template="@load(empty each.subMenus?'navitem':'nav')" menuItem="@ref(each)"/>
</template>
```


## Apply a Template inside a Template
Everything is fine so far except those sub-menu are not rendered. That's because in template `nav`, we just render a menu node itself as a `<nav>` and doesn't render its sub-menu. A node in a sub-menu is also a menu node, and it could also have sub-menu. We still need to render a sub-menu node like we do for a menu node, using a control structure. The best thing is: we don't need to repeat ourselves in template `nav` again. We can apply the template `iterate`.

```xml
<template name="nav">
	<nav label="@load(menuItem.label)" iconSclass="@load(menuItem.iconSclass)">
		<apply template="iterate" menuItems="@ref(menuItem.subMenus)"/>
	</nav>
</template>
```
- Line 3: Apply the previous tempalte `iterate` here to traverse each menu node and render





