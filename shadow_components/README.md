# Shadow Components

ZK 8 introduces a new set of components: **shadow component**. The shadow component which is like [shadow DOM](http://w3c.github.io/webcomponents/spec/shadow/) neither creates a corresponding component on the server side nor a widget on the client side. It just injects its child components inside into current page. If we use them with data binding, a page can dynamically changes upon a ViewModel's data. Since shadow components have flow control and iteration function, if you identify a repeating view pattern appearing in your application. You can implement that view pattern with shadow components to modulize your view. Hence, its most powerful usage is **making the view reusable**.

These shadow components are:

* `<apply>`: allows you to choose which template to be applied. It will lookup the template inside-out recursively.
* `<forEach>`: allows you to iterate over a collection of objects. Specifying the collection by using the `items` attribute, and you can access the current item through a variable named specified at the `var` attribute.
* `<if>`: allows the conditional execution of its body according to the value of the test attribute.
* `<choose>`/`<when>`/`<otherwise>`: they are used for logic and flow control like Java's `switch`/`case`/`default` statement.


We can use shashow components on anywhere of a zul. For example, create a component upon a condition:
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


## Navigation Menu
We will demonstrate shadow components' power with a navigation menu example below:
![](../images/ze-ch7-menu.png)

The menu is made by templates containing shadow components, and it can render different menu hierarchies without any change. You can switch 2 sets of menu hierarchy via the radio group on the right hand side. This example demonstrates how to reuse a common view pattern.

### Menu Hierarchy
We store a menu hierarchical structure in a linked list, and each node in the list might have sub-menu.

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

### Render a Collection
Starting from the basic, we can just render the list of menu node with `<forEach>` (notice the captalized 'E') and `<navitem>`. The `<forEach>` can iterate over a collection of objects. Specifying the collection by the `items` attribute and access the current item through a variable named `each`. Or you can define the current item variable by the `var` attribute e.g. if you write `<forEach items="@load(menuItems)" var="menu">` then you should use `menu` in a data binding expression like `<navitem label="@load(menu.label)">`.


**chapter5/template/menu.zul**
```xml
<navbar id="navbar" orient="horizontal" collapsed="false">
    <forEach items="@load(menuItems)">
        <navitem label="@load(each.label)"
            iconSclass="@load(each.iconSclass)">
        </navitem>
    </forEach>
</navbar>
```

But this way only renders each menu nodes as a `<navitem>`. However, we need to render those menu nodes with a sub-menu as a `<nav>`.


### Create Components upon a Condition
Next, we will render a menu node upon its sub-menu existence. If a menu node has a sub-menu, we render it by `<nav>`, otherwise render it by `<navitem>`. To achieve this, we have to use shadow components for control structure, `<choose>`, `<when>`, and `<otherwise>`. (There is an `<if>` which we don't use in the example.)
* Ther `<choose>` is just like Java `switch` statement.
* The `<when>` is like `case` statement, and you should specify a data binding expression on `test` attribute to be evaluated.
* The `<otherwise>` works like `default` case in a `switch` statement to specify a default action.

Now, we can test whether a menu node has a subMenus or not by `test="@load(empty each.subMenus)"`. If it does, create a `<nav>`, otherwise create a `<navitem>`.

```xml
<navbar id="navbar" orient="horizontal" collapsed="false">
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
</navbar>
```

But this way we only create a `<nav>` without its `<navitem>`. We need to traverse each node in its sub-menu and create correponding components (`<nav>` or `<navitem>`) which is just like the way to do now for each menu-node. It means we need to reuse this `<forEach>` and its child elements.


## Resue Components with Template
In ZK 8, `<template>` is our recommended form to reuse a view pattern composed by a group of components. Puting components into `<template>` can make them reusable easily by `<apply>`. It usually involves 2 steps:
1. Define a template
2. Apply a template

### Define a template
Since ZK 8, you can put a `<template>` inside any component. Define a template will not create any component until you apply it. You can define a template like:

```xml
<div>
    <template name="layout">
    <!-- UI components or shadow components -->
    </template>
</div>
```

or a path of a zul
```xml
<div>
    <template name="layout" src="/mytemplate.zul"/>
</div>
```
The usage is the same as we mentioned in previous chapters. But We can also use the following tags to describe a component creation logic based on certain conditions.


### Apply a template
When we apply a template, ZK will create the components inside the template upon its logic and insert those components into the position of `<apply>` tag. Therefore, we also call it *Tempalte Injection*.

We usually apply a template with its name like:

```xml
<apply template="layout"/>
```
Or apply with a path of a zul:
```xml
<apply templateURI="/chapter1/banner.zul"/>
```

## Turn into Templates
To reuse the `<forEach>`, we turn it into a template named `iterate` first.

```xml
<navbar id="navbar" orient="horizontal" collapsed="false" onSelect="@command('navigate')" >
    <apply template="iterate" menuItems="@ref(vm.menuHierarchy)"/>
</navbar>
<template name="iterate">
    <forEach items="@load(vm.menuHierarchy)">
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
- Line 2: We pass a parameter by `menuItems="@ref(vm.menuHierarchy)"`. Therefore, we can access the menu list in `<forEach>` by `items="@load(menuItems)"`.


In this simple case (just 2 choices), we can re-write it in a simpler way by creating 2 templates for `nav` and `navitem` repectively.

```xml
<template name="menu">
    <nav label="@load(menuItem.label)" iconSclass="@load(menuItem.iconSclass)"/>
</template>
```
```xml
<template name="menuitem" >
    <navitem label="@load(menuItem.label)" />
</template>
```

Then replace `<choose>`/`<when>`/`<otherwise>` with ternary operator `?` like:

```xml
<template name="iterate">
    <forEach items="@load(menuItems)">
        <apply template="@load(empty each.subMenus?'menuitem':'menu')" menuItem="@ref(each)"/>
    </forEach>
</template>
```


## Apply a Template inside a Template
Everything is fine so far except those sub-menu are not rendered. That's because in template `menu`, we just render a menu node itself as a `<nav>` and doesn't render its sub-menu. A node in a sub-menu is also a menu node, and it could also have sub-menu. We still need to render a sub-menu node like we do for a menu node, using a control structure. The best thing is: we don't need to repeat ourselves in template `menu` again. We can apply the template `iterate` to iterate a collection of menu nodes recursively.

**3 templates in this example**
```xml
<template name="menu">
    <nav label="@load(menuItem.label)" iconSclass="@load(menuItem.iconSclass)">
        <apply template="iterate" menuItems="@ref(menuItem.subMenus)"/>
    </nav>
</template>
<template name="iterate">
	<forEach items="@load(menuItems)">
		<apply template="@load(empty each.subMenus?'menuitem':'menu')" menuItem="@ref(each)"/>
	</forEach>
</template>
<template name="menuitem" >
	<navitem label="@load(menuItem.label)" />
</template>
```
- Line 3: Apply the previous tempalte `iterate` here to traverse each menu node and render them.





