# Apply CSS

In addition to setting a component's attribute, we can also change a
component's style by *CSS (Cascading Style Sheet)*. There are two
attributes to apply CSS:

1. `style` attribute. Like style attribute on HTML element, you can
directly write CSS syntax as the attribute's value.
```xml
<label value="Chapter 3" style="font-weight: bold;"/>
```
2. `sclass` attribute. You should specify a CSS class selector name as
the attribute value.
```xml
<div sclass="banner">
```

To use a CSS class selector, you should define it first in a ZUL. There
are 2 ways to define a CSS class selector.
1. `<style>` tag.
```xml
<zk>
    <style>
        .banner {
            background-color:#102d35;
            color: white;
            padding: 5px 10px;
        }
    </style>
    ...
</zk>
```
2. `<?link ?>` directive. It can link to a external style sheet which
can apply to many pages. We use this way in the example to define CSS.

```xml

<?link rel="stylesheet" type="text/css" href="/style.css"?>
<zk>
    ...
</zk>
```
