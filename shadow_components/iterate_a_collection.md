# Iterate a Collection
To start with the basics, we can just render the list of menu nodes with `<forEach>` (notice the captalized 'E') and `<navitem>`. The `<forEach>` can iterate over a collection of objects. Specify the collection by the `items` attribute and access the current item through a variable named `each`. Or you can define the current item variable by the `var` attribute e.g. if you write `<forEach items="@load(menuItems)" var="menu">` then you should use `menu` in a data binding expression like `<navitem label="@load(menu.label)">`.


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

However, doing it this way will only render each menu node as a `<navitem>`. Thus, we need to further render these menu nodes with a sub-menu in order to create a `<nav>`.
