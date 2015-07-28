# Iterate a Collection
Starting from the basic, we can just render the list of menu node with `<forEach>` (notice the captalized 'E') and `<navitem>`. The `<forEach>` can iterate over a collection of objects. Specifying the collection by the `items` attribute and access the current item through a variable named `each`. Or you can define the current item variable by the `var` attribute e.g. if you write `<forEach items="@load(menuItems)" var="menu">` then you should use `menu` in a data binding expression like `<navitem label="@load(menu.label)">`.


**chapter5/index.zul**
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
