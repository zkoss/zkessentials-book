# Apply CSS

In addition to setting a component's attribute, we can also change a
component's style by *CSS (Cascading Style Sheet)*. There are two
component attributes to apply CSS:

1. `style` attribute. Like the "style" attribute on an HTML element, you can directly write CSS rules as the attribute's value.
```xml
<label value="Chapter 1" style="font-weight: bold;"/>
```
2. `sclass` attribute. You should specify a CSS class name as
the attribute value.
```xml
<div sclass="banner">
```

To apply a CSS class, you should define it first in a ZUL. There
are 2 ways to declare a CSS class.
1. Inside `<style>` tag.
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
2. In an external CSS file and included by `<?link ?>` directive.

It can link to a external style sheet which can be applied to many pages. We use this way in the example to declare CSS.

```xml

<?link rel="stylesheet" type="text/css" href="/style.css"?>
<zk>
    ...
</zk>
```
