# Flow Control
Next, we will render a menu node upon its sub-menu existence. If a menu node has a sub-menu, we render it by `<nav>`, otherwise render it by `<navitem>`. To achieve this, we have to use shadow components for flow control, `<choose>`, `<when>`, and `<otherwise>`. (There is an `<if>` which we don't use in the example.)
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

But this way we only create a `<nav>` without its `<navitem>`. We need to traverse each node in its sub-menu and create corresponding components (`<nav>` or `<navitem>`) which is just like the way to do now for each menu-node. It means we need to reuse this `<forEach>` and its child elements.
